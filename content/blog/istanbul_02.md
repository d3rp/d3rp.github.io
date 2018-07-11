---
title: "Project in Istanbul"
date: 2017-12-14T22:54:43+02:00
draft: true
---

An Android Navigating in Istanbul
==================================

Making of a specification for an android app and to implement a given specification on the same platform.

Started as a group of 13 in total of which 2 were not present after the first presentation.
A multinational team with people from India, Bangladesh, Nepali, Spain and Finland. Six of the remaining ten were finnish. Meetings were held in English. 
Decided on two team managers of whom only the other one attended the meetings. I managed to persuade people into naming an additional technical manager, as the second manager of the first two appointed started also missing meetings. 
After a few meetings it started to seem obvious, that not many of us had any experience on the platform we were developing for. It remains a mystery to me at this point - which is two weeks prior to the first demo to be shown in person at Istanbul - that is any of the group members actually learning the android API? I know the few that actually have been working on the issues at hand have been learning more on top of what they knew already. Ironically these are the ones that already had prior experience developing for android.
At first I was exalted with this opportunity to broaden my knowledge in programming for the mobile platforms, team work and getting something to show for the future job interviewers as proof that I have hands on experience. But alas, I have come to the conclusion, that the most I can get from this apathetic and poorly motivated team effort is just something very self centred for example learning the openstreetmap format and how to use it on android. 
As I conformed with this new path of mine in the long winding road of winding up with programming skills, I found a comrade in arms that was also interested in fiddling around with the openstreetmap libraries.
We set sails and embarked on a short mission to get something substantial done. There is two weeks to go, and it is the first time in this project I have actual hope, that I'll be getting something out of this - even if it is the faith in my abilities of wrestling an open source library or two in the name of navigation.

What I've learned so far
------------------------

In meetings - keep your arguments simple and short.
A team consisting of foreigners that don't speak natively English, do not necessarily understand those, who do - or do at least speak fluently. Also - they might obscure this fact, which results in many misunderstandings. 
By confidently expressing my thoughts with the fullest vocabulary I could provide, with elaborating my point of views accurately enough to sooth my inner perfectionist, I instead of making myself crystal clear smudged most of my arguments beyond my team mates' reach. They didn't of course ask what I mean - they argued my point - and it took me few weeks to realize, why did they choose another path to the one I thought we had agreed on at the meetings? 
So .. if they are not fluent English speakers - or even if they aren't native English speakers - speak in short and simple terms.
I have to admit, I feel like a puts that tried flamboyantly and almost boisterously show off my fluent English skills. Those are skills, that I bet any good speaker of the language would find out to be flaunted with flaws.. 

A simple outline for the next steps is not rocket science
Olen myös huomannut miten ryhmätyössä menee ilmeisen usein turhaa aikaa asioiden vatvomiseen. Aika harvoin olen toistaiseksi havainnut, että lähdettäisiin alusta asti kollektiivisesti vauhtiin jonkin rautalankamaisen suunnitelman kanssa jaksottaen alkupään perustoiminnot tehokkaasti. 
Projektia hoidettaessa ryhmässä tuntuu, että perustoiminnot heti alkuun ovat:

1. tehtävän annon hahmotus - ovatko kaikki kartalla?
2. nopea tarkistus mitä kukin tekee - tahtotilat ja osaaminen..
3. brainstorming..
4. ideoiden raakkaaminen - käytännönläheinen tarkastelu brainstormaamisesta.. 

Tämä ei ole rakettitiedettä ja näihin neljään kohtaan menee enintään tunti, jos ne hoitaa ytimekkäästi. Jos näitä ei hoideta ytimekkäästi, vaan lähetään hapuilemaan tuntoja ja tiloja - haahuilemaan elämän syviä totuuksia - niin näissä neljässä kohdassa palaa aikaa  ja muuta sellaista haahuilevaa, pahimmillaan viikko.

Silence is Golden
-----------------

It was okay though to check out the temperaments and motivations in our team in the early stage of our project by staying quiet in the background. That doesn't need to extend for too long. 
Also I would say, that if I had said what I had in mind from the first instance on, I would have come across probably as a bit of a prick. I assume I had seemed a bit pushy and too opinionated. Also the guys that were a bit introverted - not saying that I'm not one of them - would have probably laid back even more. Those guys I found to be one of the best to work with yet in comparison with my previous team experiences at my university.
The golden cut that I'm trying to sketch out with this is, that there's a place and a moment for everything and that I should be at the same time affirmative, but respective of others' opinions. A way to exert such a philosophy in practice is basically to talk only when you are sure that what you're saying is valid and factual. I found it to be a reasonable way with unknown team mates to not push my argument past one or two repetitions; If the counter part did not conform or understand at that point, they will later. Also they will understand, that I'm trying to share stuff I'm sure about, and not just throw a half assed opinion for the sake of just saying something and feel important. 

Find the Right Source
---------------------

Open source libraries can be a pain! I had done some past website projects with API's like the Google Calendar and Maps API, some webshop API etc.. They usually had very straight forward tutorials and neatly put documentations even though at the time I thought they were a reasonable amount of work to deal with. 
Those API's were nothing compared with what I now had laid out in front of me. The mapsforge API described erroneous ways. I did get something done with them, but had to look in the sample projects for more information. They had a javadoc but.. 
As I looked at the samples I also dabbled with the GraphHopper API for to estimate routes on the map. They had their separate project also, and I thought it was a good idea to test these two if they work at all. I got them running on my phone and thought I'm good to go for a clean project from the scratch.
Well.. the libraries were a bit off. It took a while to figure out that GraphHopper used a different version of mapsforge than what was found on the mapsforge project site. Also this new version wasn't compatible with mf documentation.. neither I could find a GraphHopper version to match the older version of mf.. 
To add confusion to the mix, GraphHopper used maven to build the project, which I hadn't used before. I tried finding some documentation about the maven to work with as I thought just plain cloning the git project sounds too naïve to work. I had to figure out this maven thing.. 
By the time I got the maven working, GraphHopper renewed most of their stuff. In two weeks the mapsforge version had upgraded a full version number upwards. Of course, this snapshot was not on their official site. In addition, now they also used Gradle for the project management, which I hadn't used before and thought it would be too naïve to.. 
Luckily by this time I got a team mate that just cloned the git project and got the thing working. In my defence it took him almost a week. But still..
Now we got the right libraries, but no documentation. We decided to move on to make the samples work and reverse engineer them. There was also javadocs and sources to add to the eclipse projects, but they weren't exactly working. Also, they had in part explanations like "LatLong marks the coordinates of the marker." Gee thanks! Anything about super classes? Generics? Something? Anything!? 
Also I have to tell about an embarrassing rookie mistake. I downloaded the library snapshots from the git by clicking on them and selecting "save link as..". Then I spent a good hour or two finding out why the libraries wouldn't work for eclipse. I have to say I found out many different ways for including libraries into a eclipse project. Probably all of them. It turned out, github did not give me the files, but actually the pages representing the code. I found this out by double clicking on the library and noticing the html tags there where should be binary code - or the ascii mismatch representing such.
If I had done my background research, checked all the branches available, read a bit of the forum for the devs as was recommended on the mapsforge front page and gone through the API pages and javadocs first - maybe even double googling them - I had probably had a good basic understanding of where to look for the things I need. In the end it was the dev mailing list that I went to ask, 3 hours after the last update on Graphhopper and an hour after that updater had left a message on the before mentioned mailing list, and got an answer which lead me to the right source. It was a branch not publicly shown on the project site. To be accurate, it was actually the Jenkins site, that had all the necessary data needed to kick start our own version.. which wasn't actually mentioned anywhere.

Resources Won't Help a Bad Team
--------------------------------

I happened to read a little more about UMLs from a book named UML asdflkjasd from Marvis Harvis "check reference". It happened to have a mention on the development process of an application and talked about iteration. I wonder how, even with having two project managers that are studying in the business program and one enthusiastic engineer student, that "has had past experience in real work situation" we didn't come up with _iteration_? Unfortunately the only iteration we've had so far is me and few guys with me trying to figure out a library in order to use it for the project. The iterative part in it has been us rewriting stuff, because it didn't work.
As the last week begins before we leave to Istanbul to present the project and work out its last quirks, the project seems to be still quite dormant. I decided at some point to focus on learning for myself and to get a better grasp of java and android instead of focusing too much on any project managerial aspects or such development organizing issues. Still I hoped the team mates would come to arms when the last days of the project were at hand.
To learn team management or coordination was not my goal in this project. I've had past experience with those things and have a grasp of how to go about those things. I'm not sure if they're my cup of tea or at least quite yet. This project was to be a learning experience about team work - especially one, that I wasn't going to coordinate. I wanted to see how would a team do its own thing? Would it find its dynamics and chemistry by itself? How would different temperaments come out and at what point  and with what issues?
Remedy's CEO came one day to talk to us about their experience in game industry. When he was showing us different factors in the equation of team work, he told that a bad or mediocre team would not make a brilliant AAA-level game even if it was provided with unlimited time and budget. I was baffled. The young inner kid in me wanted to believe, that anything was possible - especially if given enough time and effort. So how could he go in front of us the audience and state such a blatant assumption?
Judging by the lack of results, our bad or mediocre team was seemingly dealing with this dubious fact. I'm starting to believe, that it wasn't the motivation and the innocent dream of "everything's possible" that the CEO was bashing down on, but actually the real life experience he has about how the dynamics, group chemistry and skill sets resolve into a mess of things, when the team isn't good enough. The team's quality is judged by so many factors, that just having time and a good budget doesn't resolve everything. 

Why document this?
------------------

Supposedly I'm rambling about this experience for egoistic reasons satiating my narcissism by boasting "I knew that coming" and "I told them, but they wouldn't listen" type of aftermath weaslie wisdom. I do have a point here, even though it's not necessarily so much to share, but to learn. This is my attempt on trying to apply the "DRY" principle to the softer side of software development i.e. the people and project management skills.

I remember landing my few first drumming gigs a few years back. Some of the band mates thought I had rehearsed the songs very well. Probably they didn't even realise, that before that very first paid cover gig, I really had not played any cover songs at all.  Later I noticed why they thought that. Even seasoned players would sometimes come to a gig unrehearsed. It would always shine through. There's guys that can swim in unknown territory like that - jumping in for someone else with a moments notice and still play brilliantly - maybe even outplay the other members of the band. But those guys are an exception; The rule is, learn your lessons and know your stuff.

I don't want to cover mileage unnecessarily. I have a bit of age as it is. Drawing a direct line with my current proceeding rate in the business, I feel I'll be landing mediocre job opportunities at my late fifties if even then. To speed up that process, I want to milk the experiences as they go by to their fullest. I want to draw pictures and diagrams so they'd stick better. I want to put that extra effort to really learn my lessons. That's because I've learned a lesson in my
previous endeavours, which taught me that effort is everything. It's not that you have to drag along and work up the titles - if you know your stuff, you'll get the gig. Opportunities are a random thing, and when they land upon you, you wish you had done your homework.

Is this effective?

No. This is not. I bet there's many ways to learn these very same things in less time and more thoroughly. But if you read through all this and allow me to sound a bit presumptuous and with maybe a with a hint of a condescending tone - humour me - ask yourself why did you?

Everyone has their way of working on things. It's not to say, that the wheel hasn't been invented - in other words - that proper methods have not been devised for students and workers to use. But still you have to shuffle through all the methods, maybe invent few of your own, before you grasp what's best for you. It requires effort. This is my way of putting in some extra effort to learn things. This comes naturally to me. I like to write and document to some extent and I actually read my own writings from time to time. Then reading my writings takes me back and refreshes my memory. 

