---
title: "Exit Bash"
date: 2018-05-15T17:58:22+03:00
draft: false
---

Exit, Bash!
###########

I wish I would have been told at the beginning of my journey: 
"Yes, Bash has some quirky behavior and yes, you can work around them. 
**But unless what you are doing is either trivial or insignificant, you should really look for a better tool.**"
And by trivial, as we will see, I mean not using conditionals, subshells, pipes or functions. 

I'm not considering Bash appropriate for writing any important application (you'd have to be insane to want to do that). But I am evaluating it as something it is commonly used for: Writing scripts to **surround and support** important applications, such as building release artifacts, running test automation and performing deployments.  Therefore, in this blog post I will focus on a single essential feature: **Exit when something goes wrong**. I don't care what Bash decides to do: sigkill itself, kernel panic, or set the computer on fire. Anything is better than continuing despite errors. 

I feel like I've been in an abusive relationship during the last 10 years. Most of the time, everything was fine, but if I'd fail to follow some subtle, seemingly arbitrary rule, I'd get slapped in the face. But I kept going, learning more and more obscure rules because I thought it was worth it. I thought that once I learned a little bit more, everything would be great.  I thought the problem was with me, that I just needed to get better. But no more. 
If you are in a bad relationship with someone, it's not your responsibility to tip-toe around them to avoid getting beaten. It is your responsibility to get out. 


This is my story of learning Bash over the last 10 something years. I've always seen proficiency in shell scripting as a virtue and requirement for working in UNIX environments. This is partly true: Interactive shell scripting is one of the best computer interfaces available, and I don't see this changing anytime soon. 
But the fact that it is well known, ubiquitous and convenient will tempt you into using it for automation of critical tasks. 




This is not intended as a rant about programming languages, even though it might seem like one. Nor is it a guide on how to use Bash more safely. 
Instead this is a reminder for my future self that Bash is probably not the right tool for the job. Actually, I have told myself this many times already, but keep falling into the same traps because it is too convenient. Hopefully putting this in writing will have stronger impact. 







Bash "strict mode" :code:`set -euo pipefail`
============================================


Let's say we have been using Bash for a couple of years and have started getting comfortable with its peculiarities. In particular, with regards to exiting on error, we have learned three important lessons: 

`Always use 'set -e' <http://mywiki.wooledge.org/BashPitfalls#cd_.2Ffoo.3B_bar>`_

.. code-block:: bash

	cd folder; rm -r * # This will make you sad

`Always use 'set -u' <https://github.com/valvesoftware/steam-for-linux/issues/3671>`_

.. code-block:: bash

	rm -rf "$STEAMROOT/"* # This will make you _really_ sad

`Always use 'set -o pipefail' <http://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html#The-Set-Builtin>`_

.. code-block:: bash

	# This is not going to give you a very secure password if
	# the file doesn't exist
	password=$(cat ./file_with_random_content | sha512sum) 




**From here on, assume that every single code sample is preceeded by** :code:`set -euo pipefail`.  Let's write some production code. 


Tests in Bash
=============


**First quiz: What does this code do?**

.. code-block:: bash

	if [ "$passed" -lt 100]; then
	  echo "The release is broken, abort."
	else
	  echo "Pushing to production, because ${passed}% of the tests passed"
	fi

Syntax errors are easy mistakes to make, but it shouldn't be dangerous, since we used :code:`set -e`, right? Apparently not: 

	
.. code-block:: bash

	./bash.sh: line 8: [: missing `]'
	Pushing to production, because 7% of the tests passed

What happened here? We mistakenly thought that that :code:`set -e` would make sure that the script would exit on error. 
But if the error happens within an :code:`if`-case, this check is disabled, since :code:`if` needs to interpret different status codes to function.

"But that's user error" you say. True, but it's still extremely dangerous. No amount of experience will prevent you from some day making this mistake. 
As long as code is written by humans, programming languages should protect against simple mistakes like this. 


**Second quiz: What's the difference between these two code blocks? Is one safer than the other if we get an unknown value of "$passed"?**


.. code-block:: bash

	if [ "$passed" -ge 100 ]; then
		echo "Pushing to production, because ${passed}% of the tests passed"
	else
		echo "The release is broken, abort."
	fi


.. code-block:: bash

	if [ "$passed" -lt 100 ]; then
		echo "The release is broken, abort."
	else
		echo "Pushing to production, because ${passed}% of the tests passed"
	fi

Logically they seem to be identical. But what happens if :code:`$passed` is not a number? Or if it's an empty string?

The latter block is problematic (let's pretend that the input comes from a well-written Node.js application): 

.. code-block:: bash

	./bash.sh: line 8: [: undefined: integer expression expected
	Pushing to production, because undefined% of the tests passed



The way the :code:`if`-case is designed is to execute a command (in this case :code:`/usr/bin/[`) and if the status code is :code:`0` it executes the :code:`if` block, otherwise it executes the :code:`else` block. This makes sense in some contexts. For example: 

.. code-block:: bash
	 
    if [ -z "$string" ]; then
      # The string is empty
    else
      # It's not
    fi

The problem occurs when more than two exit codes are possible, and it is important to distinguish between different positive codes. :code:`/usr/bin/[` **actually has three possible status codes: 0 for positive test, 1 for negative test and 2 for failure.** The issue is that :code:`if` treats :code:`1` and :code:`2` the same. 


What can we do to avoid this problem?

.. code-block:: bash

	# Regex to the rescue!
	re='^[0-9]+$' 
	if ! [[ "$a" =~ $re ]] ; then
	   echo not number
	   exit 1
	fi
	if [ "$a" -lt 5 ]; then
		...
	fi


.. code-block:: bash

    # If it's not equal to itself, it's not a number
    if ! [ "$a" -eq "$a" ]; then
      echo 'not a number'
      exit 1
    fi
    if [ "$a" -lt 5 ]; then
      ...
    fi

My favorite solution is hot-patching :code:`[`. In addition to completely breaking syntax highlighting, it allows you to use your original code safely without modifications. Maybe. 

.. code-block:: bash

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

I think it's fair to say that none of the solutions above are acceptable. 

.. Is that acceptable?


Double brackets
---------------

But hang on! Bash also supports the non-POSIX double-brackets for tests. Surely those have to be better?

.. code-block:: bash

	if [[ "$passed" -lt 100 ]]; then
	  echo "The release is broken, abort."
	else
	  echo "Pushing to production, because ${passed}% of the tests passed"
	fi

- Regular numbers work as expected.

- Setting :code:`passed` to an empty string converts it to a zero. That's not good, but not horrendous either. 

- Setting :code:`passed` to non-alphanumeric characters causes an error that is ignored:

.. code-block:: bash

	./double_bracket_lt.sh: line 6: [[: &/?: syntax error: operand expected (error token is "&/?")
	Pushing to production, because &/?% of the tests passed

- Setting :code:`passed` to a number followed by non-digits also continues on error: 

.. code-block:: bash


	./doublestuff.sh: line 28: [[: 5p: value too great for base (error token is "5p")
	Pushing to production, because 5p% of the tests passed


- Setting :code:`passed` to an alphabetic string: 

.. code-block:: bash

	./double_bracket_lt.sh: line 6: foobar: unbound variable

Finally, the script actually stopped when encountering an error! But look at **why** it stopped: The variable foobar is undefined. It turns out that when :code:`[[` does a numerical comparison, it evaluates strings as variable names. 
Somewhat weird, but I guess it allows you to write :code:`[[ "var" -lt 5 ]]` instead of :code:`[[ "$var" -lt 5 ]]`, which is extremely useful if you hate dollarsigns and love programming languages that try to guess what you mean.


But having a language interpret the value of a variable as code should make every programmer feel uneasy. Let's see how far we can push this. 

.. code-block:: bash


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

.. code-block:: bash

	./doublevars.sh: line 26: seriously: unbound variable


Let's create a loop: 

.. code-block:: bash

	passed='passed'
	[[ "$passed" -lt 100 ]]

.. code-block:: bash

	./doubleloop.sh: line 8: [[: passed: expression recursion level exceeded (error token is "passed")

How about a banking application, just for fun: 

.. code-block:: bash

	set -euo pipefail

	bank_account_balance=100
	withraw="$1"
	echo "current balance: $bank_account_balance"

	if [[ "$withraw" -lt "$bank_account_balance" ]] && [[ "$withraw" -gt 0 ]]; then
		echo allowed
		((bank_account_balance -= withraw)) || true
	else
		echo not allowed
	fi
	echo "new balance: $bank_account_balance"

.. code-block:: bash

	./bank.sh '((bank_account_balance=99999))'
	current balance: 100
	not allowed
	new balance: 99999


Regardless of which test operator you use, **I guess the lesson is that you always have to validate your input... before you validate your input?** 


Command Substitution
--------------------

To use output from commands in other commands, we use command substitution :code:`$()`. 

**Third quiz:** Let's say we want to encrypt some secret using a random password. Which one of these is safer if :code:`generate_password` fails for any reason?

.. code-block:: bash

    pw="$(generate_password)"
    echo "secret" | encrypt --passphrase "$pw" \
      | mail -s 'Encrypted Secret' me@example.com

.. code-block:: bash

    echo "secret" | encrypt --passphrase "$(generate_password)" \
      | mail -s 'Encrypted Secret' me@example.com

In the latter example, regardless of the return code of the subshell, the parent shell will continue using whatever was printed to stdout (probably not a great password). The first block is safe, since an assignment (without a main command), will have `"with the exit status of the last command substitution performed" <http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html#tag_02_09_01>`_. This will be caught by :code:`set -e`, and the script will exit. 



Local variables
---------------

It is arguably considered best practice to use functions and local variables to restrict scopes. In our example, we wouldn't want :code:`$pw` to be available to the whole script, since it might accidentally be misused or overwritten. 
So we take the safe code from the previous example, put it in a function and make the variable local. 

.. However, it's not without its dangers. We just learned to 


.. code-block:: bash

    f () {
      local pw="$(generate_password)"
      echo "secret" | encrypt --passphrase "$pw" \
        | mail -s 'Encrypted Secret' me@example.com
    }

What could possibly be wrong with this?

Reading :code:`man bash` reveals the answer: 

	local [option] [name[=value] ... | - ]
				  ...

				  **The return status is 0 unless local is used outside a function, an invalid name is supplied, or name  is  a  readonly
				  variable.**

Even if :code:`generate_password` fails, Bash will keep going with a bad password. 
So the only safe way to use local variables with command substitution is to define and assign variables on different lines: 

.. code-block:: bash

    # This is actually safe
    f () {
      local pw
      pw="$(generate_password)"
      echo "secret" | encrypt --passphrase "$pw" \
        | mail -s 'Encrypted Secret' me@example.com
    }


Pipes
-----

But hang on, passing an encryption key as a commandline argument is bad practice. Anyone on the same system could run :code:`ps` and read it. So it would be better to pass it as :code:`STDIN`. 

.. code-block:: bash

    generate_password | encrypt /tmp/secret \
        | mail -s 'Encrypted Secret' me@example.com
   
And since we are using :code:`set -euo pipefail`, the script should exit if :code:`generate_password` fails, right? 
When :code:`pipefail` is set, the return status of the pipeline will be set to the exit code of the last command with a non-zero status. This will be caught by :code:`set -e`, and the script will exit.  `But not until all commands in the pipeline have completed: <https://tiswww.case.edu/php/chet/bash/bashref.html#Compound-Commands>`_

    "The shell waits for all commands in the pipeline to terminate before returning a value."

So the script will stop processing after the line, but will happily send the data encrypted with a bad password first. 
The solution, again, is to first create the message and assign it to a variable, which would allow the script to exit on error. 

.. code-block:: bash

    # This is actually safe
    msg="$(generate_password | encrypt /tmp/secret)"
    echo "$msg" | mail -s 'Encrypted Secret' me@example.com
   

Unless it's ok to pass bad data through the entire pipe, you have to be very careful. 


The truth about set -e
======================

Before we can go any further, we have to really understand what :code:`set -e` does, and more importantly, doesn't do. 

From :code:`man set` and also the `POSIX specification <http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_25>`_: 

.. code-block:: bash

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

Ok, let's try to understand this bit by bit. 


.. code-block:: bash

    When this option is on, when any command fails (for any of the reasons
    listed  in Section 2.8.1, Consequences of Shell Errors or by returning
    an exit status greater than zero), the shell  immediately  shall  exit
    with the following exceptions:

Translation: "Exit on error, except...". 

.. code-block:: bash

      1. The  failure of any individual command in a multi-command pipeline
         shall not cause the shell to exit. Only the failure of  the  pipe-
         line itself shall be considered.

"By default, we only care about the exit code of the last command in the pipe", so :code:`false | false | false | true` would not be considered an error, since the last command succeeded. This behaviour is made more sane by :code:`set -o pipefail`. 

.. code-block:: bash

      2. The  −e  setting shall be ignored when executing the compound list
         following the while, until, if, or elif reserved word
		 
Ok, makes sense: the if-case expects either success or error, so :code:`set -e` has to be ignored for it to work. 

.. code-block:: bash

		 a pipeline beginning with the ! reserved word
		 
Hmm, I guess that if we have the NOT operator before a failing command, the line should be considered successful. Then logic would dictate: 

.. code-block:: bash

		 true     # don't exit?
		 ! false  # don't exit?
		 false    # exit?
		 ! true   # exit?


Nope. Read the text again: If there is a :code:`!` on the line, :code:`set -e` **is disabled**, which yields: 

.. code-block:: bash

		 true     # status code 0: don't exit
		 ! false  # status code 0: don't exit
		 false    # status code 1: exit
		 ! true   # status code 1: don't exit!

So the exit code is negated, but :code:`set -e` is disabled, logic be damned. 



And saving the best for last: 

.. code-block:: bash

		 or any command of an AND-OR list other than the last.

This is where things start to get really weird. 

Let's say that we start with a piece of code that works: 

.. code-block:: bash

    scp remoteserver:/releases/latest .
    echo 'Deploying release'

If the network goes down while transferring, we won't try to deploy half a release, because of :code:`set -e`. 
Sometime later, you realize that you need more detailed logging: 

.. code-block:: bash

    scp remoteserver:/releases/latest . && echo 'Successfully pulled release'
    echo 'Deploying release'

But the seemingly harmless addition completely breaks the protection, because :code:`scp` became a "command of an AND-OR list other than the last". 

.. code-block:: bash

	Timeout, server 1.2.3.4 not responding.
	lost connection
	Deploying release

Furthermore, if we learned anything from the NOT-operator it would be that we have to read the text carefully: What is the meaning of "last" is this context? Could it mean the last command executed, as in "Run commands according to the rules of the conditionals and if the last executed command failed, terminate the process."?

Of course not, that would be way to easy. It *clearly* means the last command **as written on the line**. Which gives us another subtle behavior: 

.. code-block:: bash

	{ echo 'false 1'; false; } && { echo 'true 1'; true; } || { echo 'false 2'; false; } 
	echo "Survived"

.. code-block:: bash

	false 1
	false 2

.. code-block:: bash

	{ echo 'false 1'; false; } || { echo 'false 2'; false; } && { echo 'true 1'; true; }
	echo "Survived"

.. code-block:: bash

	false 1
	false 2
	Survived

The exact same commands are executed, but the behavior of :code:`set -e` is different. 


Ok, this is clearly complex enough that we can't allow just anyone to mess with the production code. Let's collect all critical code into a function, and forbid anyone with less than 30 years of experience with Bash to modify it. Then all you need to do is call the function and nothing can go wrong, right?


.. code-block:: bash

	supercritical() {
	  # DO NOT MODIFY THIS FUNCTION
	  set -euo pipefail
	  scp remoteserver:/releases/latest .
	  echo 'Deploying release'
	}

	supercritical && echo "The critical function executed without errors!"


If you've read this far, you probably know what to expect: 

.. code-block:: bash

	Timeout, server 1.2.3.4 not responding.
	lost connection
	Deploying release
	The critical function executed without errors!


That's right, **by using conditionals AROUND the function, you change the behavior WITHIN the function!**

I honestly don't know if this is according to rule 2 or 3 above, but I don't care anymore. I just know enough to walk away and never look back. 



Exiting Bash
============

**The point of this post is not to teach you how to use Bash more safely, it's that you shouldn't have to.**

But let's suppose that you still would like to. You are willing to spend the time necessary to learn all of the subtle behavior and accept the mental overhead to write code while memorizing all rules. 
Still, unless you are working in a vacuum, others will most likely not. 
If you work in a team, you cannot assume that everyone will be as dedicated as you, which means that eventually someone will naively add a conditional AND-statement which could make your production script unreliable. 

This is especially insidious because it relates to error handling. Most of the time, everything seems to be working fine. The problem with the last script might not have revealed itself, because :code:`scp` has never failed so far, but eventually there will be a network glitch. 
In other languages, I haven't tried throwing every kind of exception in every kind of context and I shouldn't have to. I should be able to operate using a simple mental model that uncaught exceptions will terminate the application. 




I should also mention that I'm aware that these issues are not by design, but due to technical limitations and backwards compatibility. As an end-user of this tool however, it doesn't make any difference. I simply want to use tools I can rely on. 


In the past, I would have said that
`pages <http://mywiki.wooledge.org/BashFAQ/105>`_
like 
`these <http://mywiki.wooledge.org/BashPitfalls>`_ are required reading for everyone working in a UNIX environment. 
But that would be like recommending reading "*Top 10 tips to safely trim your fingernails with a chainsaw*"
