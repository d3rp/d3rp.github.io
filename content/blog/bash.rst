---
title: "Top 10 Tips to Safely Trim Your Fingernails with a Chainsaw"
date: 2018-08-11T11:43:49+03:00
author: "Gustav Larsson"
---


Or, Why I think Bash should stay out of production environments
===============================================================


I've always considered proficiency in shell scripting a virtue and a requirement for working in UNIX environments. This is partly true: Interactive shell scripting is, in my opinion, one of the best computer interfaces available, and I don't see this changing anytime soon. But the fact that you can seemingly achieve what you need quickly and concisely will 
tempt you into using shell scripts for automation of critical tasks. 

This is not intended as a rant about programming languages, even though it might seem like one. Nor is it a guide on how to use Bash more safely. 
Instead, this is a reminder for my future self that Bash is probably not the right tool for the job. Actually, I have told myself this many times already, but I keep falling into the same traps because it is too convenient. 
The message I'm trying to get across is: 
"Yes, Bash has some quirky behavior and yes, it is possible work around it. 
**But unless what you are doing is either trivial or insignificant, you should really look for a better tool.**"


I'm not considering Bash appropriate for writing important applications. But I am evaluating it as something it is commonly used for: Writing scripts to **surround and support** important applications, such as building release artifacts, running test automation and performing deployments. 


Therefore, this blog post will focus on a single feature I consider essential: **Exit when something goes wrong**. I don't care what Bash decides to do: :code:`SIGKILL` itself, kernel panic, or set the computer on fire. When running scripts that surround and support important applications, anything is better than continuing despite errors.


Bash "strict mode" :code:`set -euo pipefail`
============================================


Let's say we have been using Bash for a couple of years and have started getting comfortable with its peculiarities. In particular, with regards to exiting on error, we have learned three important lessons: 

`Always use 'set -e' <http://mywiki.wooledge.org/BashPitfalls#cd_.2Ffoo.3B_bar>`_ (terminate the script if a command fails)

{{< highlight bash >}}
	cd folder; rm -r * # This will make you sad
{{< /highlight >}}

`Always use 'set -u' <https://github.com/valvesoftware/steam-for-linux/issues/3671>`_ (terminate the script if referencing an uninitialized variable)

{{< highlight bash >}}
	rm -rf "$STEAMROOT/"* # This will make you _really_ sad
{{< /highlight >}}

`Always use 'set -o pipefail' <http://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html#The-Set-Builtin>`_ (treat an error in the pipeline as an error of the whole command)

{{< highlight bash >}}
	# This is not going to give you a very secure password if
	# the file doesn't exist
	password=$(cat ./file_with_random_content | sha512sum) 
{{< /highlight >}}



These three commands are referred to as "Bash strict mode" by many resources online, and will supposedly protect against the dangerous pitfalls of Bash.
Whether this is a fair description remains to be seen. 

**From here on, assume that every single code sample is preceeded by** :code:`set -euo pipefail`.  Now let's write some production code. 


Tests in Bash
=============


**First quiz: What does this code do?**

{{< highlight bash >}}
	if [ "$passed" -lt 100]; then
	  echo "The release is broken, abort."
	else
	  echo "Pushing to production, because ${passed}% of the tests passed"
	fi
{{< /highlight >}}


Did you spot the syntax error?
It shouldn't be dangerous though, since we used :code:`set -e`, right? 


	
{{< highlight txt >}}
	./script.sh: line 123: [: missing `]'
	Pushing to production, because 7% of the tests passed
{{< /highlight >}}

What happened here? We mistakenly thought that that :code:`set -e` would make sure that the script would exit on error. 
But if the error happens within an :code:`if`-case, this check is disabled, since :code:`if` needs to interpret different status codes to function.

This kind of bug is easy to introduce and can take a long time to discover. Since the tests pass most of the time, the script will seemingly behave correctly, unless someone digs through the logs. 

**Second quiz: What's the difference between these two code blocks? Is one safer than the other if we get an unknown value of "$passed"?**


{{< highlight bash >}}
	if [ "$passed" -ge 100 ]; then
	  echo "Pushing to production, because ${passed}% of the tests passed"
	else
	  echo "The release is broken, abort."
	fi
{{< /highlight >}}


{{< highlight bash >}}
	if [ "$passed" -lt 100 ]; then
	  echo "The release is broken, abort."
	else
	  echo "Pushing to production, because ${passed}% of the tests passed"
	fi
{{< /highlight >}}

Logically they seem to be identical. But what happens if :code:`$passed` is not a number? Or if it's an empty string?

The latter code block is problematic (let's pretend that the input comes from a well-written Node.js application): 

{{< highlight txt >}}
	./script.sh: line 123: [: undefined: integer expression expected
	Pushing to production, because undefined% of the tests passed
{{< /highlight >}}



The :code:`if`-case is designed to execute a command (in this case :code:`/usr/bin/[`) and if the status code is :code:`0`, it executes the :code:`if` block, otherwise it executes the :code:`else` block. This makes sense in some contexts. For example: 

{{< highlight bash >}}
    if [ -z "$string" ]; then
      # The string is empty
    else
      # It's not
    fi
{{< /highlight >}}

The problem occurs when more than two exit codes are possible and it is important to distinguish between different positive values. :code:`/usr/bin/[` **actually has three possible status codes: 0 for positive test, 1 for negative test and 2 for failure.** The issue is that :code:`if` treats :code:`1` and :code:`2` the same. 


What can we do to work around this problem?

{{< highlight bash >}}
	# Regex to the rescue! :(
	re='^[0-9]+$' 
	if ! [[ "$a" =~ $re ]] ; then
	   echo 'not a number'
	   exit 1
	fi
	if [ "$a" -lt 5 ]; then
		...
	fi
{{< /highlight >}}


{{< highlight bash >}}
    # If it's not equal to itself, it's not a number
    if ! [ "$a" -eq "$a" ]; then
      echo 'not a number'
      exit 1
    fi
    if [ "$a" -lt 5 ]; then
      ...
    fi
{{< /highlight >}}

My favorite solution is monkey patching :code:`[`. In addition to completely breaking syntax highlighting, it allows you to use your original code safely without modifications. Maybe. 

{{< highlight bash >}}
  function [ () {
    # Patched test operator. I have no idea if this is reliable, 
    # it will probably fail spectacularly on some corner-case.
    builtin [ "$@" # Do the actual test. Use 'builtin' to avoid a recursive loop.
    statuscode=$?
    case $statuscode in
      0)
        echo "Status code: $statuscode The test is positive"
        return $statuscode
      ;;
      1)
        echo "Status code: $statuscode The test is negative"
        return $statuscode
      ;;
      2)
        echo "Status code: $statuscode The test failed with an error, exiting"
        exit 1
      ;;
      *)
        echo 'This should never happen??'
        exit 1
      ;;
    esac
  }
{{< /highlight >}}

I think it's fair to say that none of the solutions above are acceptable. 

.. Is that acceptable?


Double brackets
---------------

But hang on! Bash also supports the non-POSIX double-brackets :code:`[[]]` for tests. Surely those have to be better?

{{< highlight bash >}}
	if [[ "$passed" -lt 100 ]]; then
	  echo "The release is broken, abort."
	else
	  echo "Pushing to production, because ${passed}% of the tests passed"
	fi
{{< /highlight >}}

Let's try some different inputs: 

- Regular numbers work as expected.

- Setting :code:`passed` to an empty string converts it to a zero. This can cause unexpected behaviour. 

- Setting :code:`passed` to non-alphanumeric characters throws an error that is ignored:

{{< highlight txt >}}
	./script.sh: line 123: [[: &/?: syntax error: operand expected (error token is "&/?")
	Pushing to production, because &/?% of the tests passed
{{< /highlight >}}

- Setting :code:`passed` to a number followed by non-digits also continues on error: 

{{< highlight txt >}}
	./script.sh: line 123: [[: 5p: value too great for base (error token is "5p")
	Pushing to production, because 5p% of the tests passed
{{< /highlight >}}


- Setting :code:`passed` to an alphabetic string: 

{{< highlight txt >}}
	./script.sh: line 123: foobar: unbound variable
{{< /highlight >}}

Finally, the script actually stopped when encountering an error! But look at **why** it stopped: The variable foobar is undefined. It turns out that when :code:`[[` does a numerical comparison, it evaluates strings as variable names. 
Somewhat weird, but I guess it allows you to write :code:`[[ "var" -lt 5 ]]` instead of :code:`[[ "$var" -lt 5 ]]`, **which is extremely useful if you hate dollar signs and love programming languages that try to guess what you mean.**


But having a language interpret the value of a variable as code should make every programmer feel uneasy. Let's see how far we can push this. 

{{< highlight bash >}}
	passed='oh'
	oh='god'
	god='please'
	please='no'
	no='why'
	why='is'
	is='this'
	this='a'
	a='feature'
	feature='seriously'
	[[ "$passed" -lt 100 ]]
{{< /highlight >}}

{{< highlight txt >}}
	./script.sh: line 123: seriously: unbound variable
{{< /highlight >}}


Let's create a loop: 

{{< highlight bash >}}
	passed='passed'
	[[ "$passed" -lt 100 ]]
{{< /highlight >}}

{{< highlight txt >}}
	./script.sh: line 123: [[: passed: expression recursion level exceeded (error token is "passed")
{{< /highlight >}}

How about a banking application, just for fun: 

{{< highlight bash >}}
	set -euo pipefail

	bank_account_balance=100
	withdraw="$1"
	echo "current balance: $bank_account_balance"

	if [[ "$withdraw" -lt "$bank_account_balance" ]] && [[ "$withdraw" -gt 0 ]]; then
	  echo allowed
	  ((bank_account_balance -= withdraw)) || true
	else
	  echo not allowed
	fi
	echo "new balance: $bank_account_balance"
{{< /highlight >}}

{{< highlight txt >}}
	./bank.sh '((bank_account_balance=99999))'
	current balance: 100
	not allowed
	new balance: 99999
{{< /highlight >}}

Oops...

Regardless of which test operator you use, **I guess the lesson is that you always have to validate your input... before you validate your input?** 


Command Substitution
====================

To use output from commands in other commands, we use command substitution :code:`$()`. 

**Third quiz:** Let's say we want to encrypt some secret using a random password. Which one of these is safer if :code:`generate_password` fails?

{{< highlight bash >}}
    pw="$(generate_password)"
    echo "secret" | encrypt --passphrase "$pw" \
      | mail -s 'Encrypted Secret' me@example.com
{{< /highlight >}}

{{< highlight bash >}}
    echo "secret" | encrypt --passphrase "$(generate_password)" \
      | mail -s 'Encrypted Secret' me@example.com
{{< /highlight >}}

In the latter example, regardless of the return code of the subshell, the parent shell will continue using whatever was printed to stdout (probably not a great password). The first block is safe, since an assignment (without a main command), will return with `"the exit status of the last command substitution performed" <http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html#tag_02_09_01>`_. This will be caught by :code:`set -e`, and the script will exit. 



Local variables
===============

It is arguably considered best practice to use functions and local variables to restrict scope. In our example, we wouldn't want :code:`$pw` to be available to the whole script, since it might accidentally be misused or overwritten. 
So we take the safe code from the previous example, put it in a function and make the variable local. 

.. However, it's not without its dangers. We just learned to 


{{< highlight bash >}}
    f () {
      local pw="$(generate_password)"
      echo "secret" | encrypt --passphrase "$pw" \
        | mail -s 'Encrypted Secret' me@example.com
    }
{{< /highlight >}}

What could possibly be wrong with this?

Reading :code:`man bash` reveals the answer: 

{{< highlight txt >}}
	local [option] [name[=value] ... | - ]
				  ...

				  The return status is 0 unless local is used outside a function, an invalid name 
				  is supplied, or name  is  a  readonly variable.
{{< /highlight >}}

Even if :code:`generate_password` fails, Bash will keep going with a bad password. 
So the only safe way to use local variables with command substitution is to define and assign variables on different lines: 

{{< highlight bash "hl_lines=3-4" >}}
    # This is actually safe
    f () {
      local pw
      pw="$(generate_password)"
      echo "secret" | encrypt --passphrase "$pw" \
        | mail -s 'Encrypted Secret' me@example.com
    }
{{< /highlight >}}


Pipes
=====

But hang on, passing an encryption key as a commandline argument is bad practice. Anyone on the same system could run :code:`ps` and read it. It would be better to pass it as :code:`STDIN`. 

{{< highlight bash >}}
    generate_password | encrypt /tmp/secret \
        | mail -s 'Encrypted Secret' me@example.com
{{< /highlight >}}
   
And since we are using :code:`set -euo pipefail`, the script should exit if :code:`generate_password` fails, right? 
When :code:`pipefail` is set, the return status of the pipeline will be set to the exit code of the last command with a non-zero status. This will be caught by :code:`set -e`, and the script will exit.  `But not until all commands in the pipeline have completed: <https://tiswww.case.edu/php/chet/bash/bashref.html#Compound-Commands>`_

{{< highlight txt >}}
    "The shell waits for all commands in the pipeline to terminate before returning a value."
{{< /highlight >}}

So the script will stop processing after the line, but will happily send the data encrypted with a bad password first. 
The solution, again, is to first create the message and assign it to a variable, which would allow the script to exit on error. 

{{< highlight bash >}}
    # This is actually safe
    msg="$(generate_password | encrypt /tmp/secret)"
    echo "$msg" | mail -s 'Encrypted Secret' me@example.com
{{< /highlight >}}
   

Unless it's ok to pass bad data through the entire pipe, you have to be very careful. 


The truth about set -e
======================

Before we can go any further, we have to really understand what :code:`set -e` does, and more importantly, doesn't do. 

From :code:`man set` and the `POSIX specification <http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_25>`_: 

{{< highlight txt >}}
    When this option is on, when any command fails (for any of the reasons
    listed  in Section 2.8.1, Consequences of Shell Errors or by returning
    an exit status greater than zero), the shell  immediately  shall  exit
    with the following exceptions:

      1. The  failure of any individual command in a multi-command pipeline
         shall not cause the shell to exit. Only the failure of  the  pipe-
         line itself shall be considered.

      2. The  −e  setting shall be ignored when executing the compound list
         following the while, until, if, or elif reserved word, a  pipeline
         beginning  with  the !  reserved word, or any command of an AND-OR
         list other than the last.

      3. If the exit status of a compound command  other  than  a  subshell
         command  was  the  result of a failure while −e was being ignored,
         then −e shall not apply to this command.

      This requirement applies to the shell environment  and  each  subshell
      environment separately.
{{< /highlight >}}

Ok, let's try to understand this bit by bit. 


{{< highlight txt >}}
    When this option is on, when any command fails (for any of the reasons
    listed  in Section 2.8.1, Consequences of Shell Errors or by returning
    an exit status greater than zero), the shell  immediately  shall  exit
    with the following exceptions:
{{< /highlight >}}

Translation: "Exit on error, except...". 

{{< highlight txt >}}
      1. The  failure of any individual command in a multi-command pipeline
         shall not cause the shell to exit. Only the failure of  the  pipe-
         line itself shall be considered.
{{< /highlight >}}

"By default, we only care about the exit code of the last command in the pipe", so :code:`false | false | false | true` would not be considered an error, since the last command succeeded. This behaviour is made more sane by :code:`set -o pipefail`. 

{{< highlight txt >}}
      2. The  −e  setting shall be ignored when executing the compound list
         following the while, until, if, or elif reserved word
{{< /highlight >}}
		 
Ok, makes sense: the :code:`if`-case expects either success or error, so :code:`set -e` has to be ignored for it to work. 

{{< highlight txt >}}
		 a pipeline beginning with the ! reserved word
{{< /highlight >}}
		 
Hmm, I guess that if we have the NOT operator before a failing command, the line should be considered successful. Then logic would dictate: 

{{< highlight bash >}}
		 true     # don't exit?
		 ! false  # don't exit?
		 false    # exit?
		 ! true   # exit?
{{< /highlight >}}


Nope. Read the text again: If there is a :code:`!` on the line, :code:`set -e` **is disabled**, which yields: 

{{< highlight bash >}}
		 true     # status code 0: don't exit
		 ! false  # status code 0: don't exit
		 false    # status code 1: exit
		 ! true   # status code 1: don't exit!
{{< /highlight >}}




And saving the best for last: 

{{< highlight txt >}}
		 or any command of an AND-OR list other than the last.
{{< /highlight >}}

This is where things start to get really weird. 

Let's say that we start with a piece of code that works: 


{{< highlight bash >}}
    scp remoteserver:/releases/latest .
    echo 'Deploying release'
{{< /highlight >}}

If the network goes down while transferring, we won't try to deploy half a release, because of :code:`set -e`. 
Sometime later, you realize that you need more detailed logging: 

{{< highlight bash >}}
    scp remoteserver:/releases/latest . && echo 'Successfully pulled release'
    echo 'Deploying release'
{{< /highlight >}}

But the seemingly harmless addition completely breaks the protection, because :code:`scp` suddenly became a "command of an AND-OR list other than the last". 

{{< highlight txt >}}
	Timeout, server 1.2.3.4 not responding.
	lost connection
	Deploying release
{{< /highlight >}}

Furthermore, if we learned anything from the NOT-operator it would be that we have to read the specification carefully: What is the meaning of "last" is this context? Could it mean the last command executed, as in "Run commands according to the rules of the conditionals and if the last executed command failed, terminate the process."?

Of course not, that would be way to easy. It *clearly* means the last command **as written on the line**. Which gives us another subtle behavior: 

{{< highlight bash >}}

	{ echo 'false 1'; false; } && { echo 'true 1'; true; } || { echo 'false 2'; false; } 
	echo "Survived"

{{< /highlight >}}
{{< highlight txt >}}

	false 1
	false 2

{{< /highlight >}}

{{< highlight bash >}}

	{ echo 'false 1'; false; } || { echo 'false 2'; false; } && { echo 'true 1'; true; }
	echo "Survived"

{{< /highlight >}}

{{< highlight txt >}}

	false 1
	false 2
	Survived

{{< /highlight >}}

The exact same commands are executed, but the behavior of :code:`set -e` is different.  

Ok, this is clearly complex enough that we can't allow just anyone to mess with the production code. Let's collect all critical code into a function, and forbid anyone with less than 30 years of experience with Bash to modify it. Then all you need to do is call the function and nothing can go wrong, right?


{{< highlight bash >}}

	supercritical() {
	  # DO NOT MODIFY THIS FUNCTION
	  set -euo pipefail
	  scp remoteserver:/releases/latest .
	  echo 'Deploying release'
	}

	supercritical && echo "The critical function executed without errors!"

{{< /highlight >}}

If you've read this far, you probably know what to expect: 

{{< highlight txt >}}
	Timeout, server 1.2.3.4 not responding.
	lost connection
	Deploying release
	The critical function executed without errors!
{{< /highlight >}}


That's right, **by using conditionals AROUND the function, you change the behavior WITHIN the function!**

I honestly don't know if this is according to rule 2 or 3 above, but I don't care anymore. I just know enough to walk away and never look back. 



Exiting Bash
============


**The point of this post is not to teach you how to use Bash more safely, but to tell you that you shouldn't have to.**

Let's suppose that you still would like to anyway. You are willing (like me, apparently) to spend unreasonable amounts of time studying the subtle behavior of Bash and accept the mental overhead needed to write code while going through all the rules in your head. 
Unless you live in a vacuum, this is not enough.
If you are part of a team, you cannot assume that everyone will be as dedicated to learn the intricacies of Bash as you are.  Someone will eventually add a seemingly innocent AND-statement which could make your production script unreliable. 

This is especially insidious because it relates to error handling. Most of the time, everything seems to be working fine. The problem with the last script might not have revealed itself, because :code:`scp` has never failed so far, but eventually there will be a network glitch. 
Bash allows you to quickly write scripts that *seem* to work, but don't handle corner cases in a sane way. 


I'm aware that these issues are not by design, but due to technical limitations and backwards compatibility. As an end-user of this tool however, it doesn't make any difference. I simply want to use tools I can trust. 

In an ideal world, I'll write a follow-up to describe the perfect Bash substitute. Let's see how that goes. 

