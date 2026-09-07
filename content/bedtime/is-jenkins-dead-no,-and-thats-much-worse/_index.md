
+++
title = "Is Jenkins Dead? No, And That's Much Worse"
date = 2026-09-07T15:00:00+00:00
draft = false
+++


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the tale of Jenkins. The butler who never once said no.


Now, this is not one of those stories where the hero loses. Jenkins won. Jenkins won everything. For the better part of a decade, if you wrote software for a living, Jenkins stood between your keyboard and your customers, and it built your code, and it tested it, and it shipped it, every single night, without ever once being thanked.


And here's the thing. 

Almost nobody chooses Jenkins any more. Ask around your office and you will find people who would rip it out tomorrow morning. They can't. Twenty-odd years after a young engineer at Sun Microsystems wrote the first version of it, it is still there, still running the thing that actually ships your product, and everybody has quietly agreed not to fucking touch it.

So tuck in. Because tonight's nightmare isn't that our hero dies at the end. It's that he doesn't. He is still running tonight. And you cannot switch him off.

<!--more-->

{{< youtube LbxQqAuQ4Xk >}}


## Kohsuke Kawaguchi & Hudson — The Boy Who Kept Breaking Things


Once upon a time, in the summer of 2004, there was a young engineer at Sun Microsystems, and his name was Kohsuke Kawaguchi.


And Kohsuke kept breaking the build. His own machine was always fine, of course. But somebody else would update their workspace, and suddenly nothing compiled and the whole thing went to shit, and the phone would ring. And on the other end would be somebody who had just lost their entire morning, being very, very polite about it. "I think you touched this the last time. Can you look into it?" And it usually was him.


Now, 

Kohsuke was a lazy man — he will happily tell you so himself. And a lazy man in that position does not resolve to be more careful. A lazy man thinks: what if something else checked? Something tireless. Something that takes the work the moment you hand it over, builds the whole thing from scratch, and tells you the truth about it before anybody else finds out. What he wanted, when you get down to it, was a butler.


So he built one. He named it Hudson, after a butler from a television series, because that is exactly what it was. A servant. You ring, and it comes, and it does the thing, and it does not complain about it.


And this is the detail that matters. Everything that happens over the next twenty-odd years happens because of this one thing. Kohsuke did not build something that runs builds. He built something that runs any command you give it, whenever you tell it to. Builds first. But any command.


In most places, that job had belonged to a shell script on a machine under somebody's desk. It went off at two in the morning. Exactly one person understood it. And when that person went on holiday, the whole team held its breath.


Hudson was released in February two thousand and five, and Kohsuke's colleagues loved it immediately. Partly because it was good. Mostly because they were all exactly as lazy as he was.


What Kohsuke built there has a name: continuous integration. And he did not invent it — the idea was already a decade old, and CruiseControl had been doing it in the open since 2001. What Hudson did was put it within reach of everybody. And it managed that by saying yes to everybody. Yes to you: CruiseControl wanted XML configuration files, and Hudson gave you a web page and a button. And yes to everyone else, because Kohsuke built it so that anybody could bolt a new ability onto it without asking his permission. That's the plugin system, and it grew into one of the largest plugin ecosystems in open source — eighteen hundred of them, written by thousands of people.

## Hudson Takes Over — The Butler Who Never Slept


Hudson did not stay at Sun Microsystems for very long.


It was free, it was open source, and it ran on more or less anything, so it got out of Sun the way these things always do. One engineer at a time. Somebody used it at work, liked it, took it with them to the next job, and set it up there in an afternoon.


And that really was all it took. An afternoon. You gave it a machine, you told it where your code lived, you clicked a few things, and by the end of the day something was building your project every time anybody touched it. If you have ever done that for the first time, you remember how good it felt.


Meanwhile, Kohsuke kept shipping. Not every year. Not every quarter. A new release every week — a habit the project still has to this day.


Now, here is the thing about Hudson. On its own, it did not do very much at all. It scheduled work, it ran things, and it showed you the results on a web page. Everything it could actually do — talk to your version control, run your build tool, publish your test results, send the email when it all fell over — every single one of those was a plugin.


So the plugins came. Dozens at first, and then hundreds, written by people Kohsuke had never met, for tools he had never used. Whatever strange internal thing your company depended on, sooner or later somebody wrote a plugin for it, and then Hudson could talk to that too.


Which means the thing the entire industry was coming to rely on was mostly not written by the people who made it. It was a large pile of other people's work, held together by a small program whose real talent was holding things.


In May of two thousand and eight, at JavaOne, Sun gave Hudson a Duke's Choice Award. Sun handing out a prize to a program that one of its own engineers had written because he kept breaking the build.


And by the end of that decade, it had simply won. CruiseControl faded. The others faded. If you were doing continuous integration at all, you were almost certainly doing it with Hudson.


And notice how it won, because this matters later. Almost nobody picked Hudson for its architecture. People picked it because it cost nothing, because you could have it running before lunch, and because somebody had already written a plugin for whatever strange thing you needed. That is how infrastructure actually gets adopted — not by being the best design, but by being the path of least resistance. Which is wonderful. Right up until the day you would like to change your mind.

## Oracle and the Jenkins Fork — The Night They Took His Name


In January of 2010, Oracle bought Sun Microsystems. And Hudson, along with everything else that Sun owned, went with it.


Kohsuke left a few months later, and went to work for a young company called CloudBees.


Now, the fight, when it came, did not start over money. It did not even start over the code. It started over where the project lived. Hudson's home was Sun's old infrastructure, and it was slow, and it was creaking, and the developers wanted to move everything to GitHub, where the rest of the world already was.


And then, in the autumn of two thousand and ten, a migration went wrong, and the Hudson developers found themselves locked out of their own source code.


So they said: "Right, screw this, we're moving." And Oracle said no. And when they pushed, Oracle reached for the one thing it could actually hold on to. Not the code — the code was open source, anybody could take a copy. The name. On the twenty-ninth of October, while the developers were busy moving the project to GitHub, Oracle filed a trademark application for the word Hudson.


Nobody ever went to court. There was no lawsuit, no judge. There were meetings, and in those meetings Oracle would not let go of the name, and the developers worked out that fighting Oracle over a single word was not something any of them could afford.


So they did the other thing. They held a vote. Keep the name, or walk away and call the project something else. Two hundred and fourteen people voted to walk. Fourteen voted to stay.


And then they had to choose a new name. They considered Alfred, after Batman's butler, and let it go because something else already had it. And in the end they settled on the name of a different butler altogether. Jenkins.


Because that was the joke. Oracle had taken the name, so they went and got another one exactly like it. Same code. Same people. Same plugins. Same project. All that changed was the word on the front. Everything else changed later, until it was a completely different project and still, somehow, the same one.


Oracle kept Hudson, and in May of two thousand and eleven it handed the whole thing to the Eclipse Foundation — the code, the trademark, the domain name, all of it. Hudson's website was switched off in January of two thousand and twenty, and the project was archived the year after that. By then, almost nobody had used it in years.


Now, it is tempting to make Oracle the villain of this chapter, and I don't think that's quite right. 

Oracle had just paid seven point four billion dollars for Sun, and securing what you've bought is exactly what you do. The mistake was thinking that Hudson was one of the things it had bought. Sun didn't own Hudson. Sun owned a word, attached to a project built by thousands of people who worked somewhere else entirely. Oracle secured the only part it could legally hold, and lost the only part that was worth anything. And it wasn't a one-off. In the eighteen months around that acquisition, MySQL became MariaDB, OpenSolaris became Illumos, OpenOffice became LibreOffice, and Hudson became Jenkins. Four for four.

## Jenkins Plugins — Two Thousand Hands


The plugins never stopped coming. A few hundred became a thousand. A thousand became eighteen hundred, which is roughly where it stands today.


And remember what Kohsuke actually built. Not something that runs builds. Something that runs any command you give it, whenever you tell it to. So people gave it everything. Not just building and testing. Deployments. Database migrations. Nightly backups. Certificate renewals. The Friday release. That report the finance team needs on the first of every month. The little script that used to live on somebody's laptop.


Which meant that, quietly, over about ten years, Jenkins became the most privileged machine your company owned. It could reach production. It held the cloud credentials, and the signing keys, and the keys to every server you had. If you wanted to own a company, you did not attack the company. You attacked its Jenkins.


And all of those plugins ran in a single process, sharing everything. Which meant that upgrading one of them could break another one, written by somebody else, years earlier, for an entirely different purpose.


So people stopped upgrading. And you have met this one. The Jenkins nobody touches. The one four years behind, where somebody has effectively taped a note next to the upgrade button that says please don't.


Some of those plugins had been abandoned altogether. The person who wrote the one your release depends on stopped maintaining it in twenty sixteen, moved on, and never came back. It still works. Nobody is watching it.


In January of twenty twenty-four, a vulnerability called CVE-2024-23897 let a stranger with no account at all start reading files off your Jenkins. Not your application. Your Jenkins. The machine holding all the keys.


And by then, most of them could not be rebuilt anyway. Twenty years of settings, entered by clicking, saved to a disk. If your Jenkins burned down tonight, you would not restore it from source control. You would try to remember.


Here is the part I want you to sit with. Every single one of those plugins was the right call on the day somebody installed it. Nobody was lazy, nobody was stupid. Somebody needed a thing to happen on a schedule, there was already a machine that did things on a schedule, and handing it over took about four minutes. That is what lock-in actually is. It is never a decision — nobody signs anything. It is an accumulation. Two thousand small, correct yeses, and then one morning you work out that leaving would cost more than anybody at your company is willing to spend. That is how you get fucked. Politely. Four minutes at a time.

## Pipeline, Blue Ocean and Jenkins X — The Vacuum in the Cupboard


Now, the people who run Jenkins knew all of this. Every single thing I have just described — they knew, and they tried to fix it.


Start with the falling over. Give Jenkins enough work and it runs out of memory, and it stops answering, and somebody has to go and restart it. And when it comes back, every build that was running is simply gone.


So they set out to fix that properly. A build ought to survive its own server restarting. Which means Jenkins has to be able to freeze a running script halfway through and pick it up again later. Which means it cannot simply run your script. It has to rewrite it into something that can be paused at every single step.


And that is why the language inside a Jenkinsfile looks like Groovy and is not Groovy. Write an ordinary loop the ordinary way, and it breaks. And the only way to find out is to commit it, push it, and wait four minutes for a stack trace to explain, in Java, why you are an idiot. That is not carelessness. That is a fix working exactly as designed, for a problem that should never have needed fixing.


And they kept going. In two thousand and sixteen, Jenkins 2.0 made pipelines a first-class thing and put a Jenkinsfile in your repository, where it belonged. That was right. That worked.


The same year, they built Blue Ocean — a modern, clean, genuinely lovely new interface. It went into maintenance in twenty twenty-three, never received another feature, and is now being retired altogether. So the screen you look at today is still, more or less, the screen from two thousand and nine.


In two thousand and eighteen came Configuration as Code, so you could finally describe a whole Jenkins in a file instead of by clicking. Ten years after everybody needed it. And it was, of course, a plugin.


And that same year, they sent an heir out into the new world. Jenkins X — Jenkins for Kubernetes, cloud-native, the future of everything. To survive out there, it had to take Jenkins out of Jenkins X. It runs on Tekton now. And today it has one maintainer, most of its releases are filed by a robot, and it has quietly dropped Jenkins from its name as well.


But back at Jenkins itself, none of those repairs could ever be a rebuild. That was the whole problem. Eighteen hundred plugins are written against the inside of Jenkins, so the inside of Jenkins cannot be changed. Everything had to go on top. Never underneath.


Which left the repair that mattered most permanently out of reach. There can only ever be one controller. It hands work out to as many machines as you like — that part works beautifully — but the controller itself cannot be copied. You cannot run two. So large companies ended up running dozens of separate Jenkins installations that have never heard of one another. And nobody could ever turn Jenkins into a service you simply sign up for.


And while all of that was going on, the world quietly stopped needing a Jenkins at all. In twenty nineteen, GitHub put continuous integration directly into the place your code already lived. And GitHub was not alone — Travis, CircleCI, Buildkite, GitLab, some of them at it for years already. No server to run. No plugins to install. Nobody to restart it at two in the morning.


And notice what did not happen in any of that. Pipelines, Blue Ocean, Configuration as Code, Jenkins X — Jenkins refused none of them. It accepted every single one, gratefully. The problem was never that it would not change. The problem is that it could only ever be added to, never replaced. And after twenty-odd years of only ever adding, what you have is not something you can fix. The only repair left is to leave. Which is the one thing almost nobody can afford to do.

## Jenkins Today — The Script Under the Desk


So here is where Jenkins is tonight. It did not die. It is not a museum piece. It is running right now, inside banks, inside telcos, behind VPNs, in regulated industries, on machines you cannot reach from the internet, doing exactly what it has always done.


Jenkins is not chosen any more. Jenkins is inherited.


And look at what losing actually looks like here. Around a third of companies run two different CI tools. Some run three. They did not replace Jenkins. They bought the new thing and kept Jenkins as well.


Because when a company moves to GitHub Actions, it moves the builds. The builds are the easy part. They are written down, somebody understands them, and they already live in the repository.


Everything else stays. The nightly backup stays. The certificate renewal stays. The database migration stays. The report the finance team needs on the first of every month stays. And the little script that used to live on somebody's laptop, and moved to Jenkins, and has run every night since two thousand and fourteen — that stays too.


So what is left, at a very large number of companies, is a machine that runs a pile of jobs nobody has ever made a list of. Fired every night at two in the morning. Understood by exactly one person. Touched by nobody.


Which is, of course, precisely the thing that Kohsuke built it to replace.


And it will not stop. It cannot. It has never once refused anything anybody has asked of it, and it is not going to start now. It will run tonight, and tomorrow night, and the night after that. One person where you work knows what all of it is for. And that person is going to leave — this year, or next year, or the year after that. And when they do, nobody is going to switch it off. Because switching it off would mean first finding out what it does.

## Goodnight


Goodnight, then. Sleep well.

And in about four hours, at two o'clock in the morning, your phone is going to ring. And it will be somebody who has already been awake for an hour, and who is not in the mood.

"Jenkins is down. Again. You brought this thing in here. Deal with it."
