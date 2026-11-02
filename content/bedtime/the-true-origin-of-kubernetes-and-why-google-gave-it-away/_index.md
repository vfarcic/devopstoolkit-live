
+++
title = "The True Origin of Kubernetes (And Why Google Gave It Away)"
date = 2026-11-02T15:00:00+00:00
draft = false
+++


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the story of Kubernetes.


You already know Kubernetes. That's exactly why you never asked where it came from. You wrote YAML this week. You know roughly what a pod is. You have been on a call where somebody said "we'll just put it in the cluster," and everybody nodded, and nobody said out loud what that was going to cost.

But that is the last ten minutes of a much longer story. Before Kubernetes, machines had names. There was a system inside Google that nobody outside Google had ever laid eyes on. There was a genuinely better answer out of Berkeley, and it lost. And there was a meeting where the man in charge of Google's infrastructure listened to this whole plan and could not believe what he was hearing.

So tonight I'll tell you where Kubernetes actually came from, why Google gave away the thing it called a competitive advantage, and which companies ended up collecting.

<!--more-->

{{< youtube FJk5v8k79EY >}}


## Pets vs Cattle: Servers Before Kubernetes — The Machines Had Names


Once upon a time, computers had names.


There was a machine called `zeus`, and a machine called `hera`, and one called `db-old-DO-NOT-TOUCH`. They were real objects, in a real building, with stickers on the front. You could walk up and put your hand on the one running payroll. They were pets. You named them, you nursed them through every illness, and when one died, somebody took it personally.


Deploying meant connecting to a machine you could name and copying files onto it. Maybe a shell script. Maybe Capistrano, or rsync, or a wiki page with fourteen steps where step nine was "ask Dave." If a disk died, somebody drove to the building.


The knowledge of which box did what lived in two places: that fourteen-step wiki page, which was out of date, and Dave. Dave could not go on holiday. Dave could not get sick. Everybody knew this, and nobody wrote a damn thing down.


And because a machine took months to buy, you bought for the worst day of the year. So every machine spent most of its life doing almost nothing, waiting for the Christmas sales. Whole rooms of expensive metal, idling politely.


And yet, somehow, nobody ever had enough. Every team was always one ticket away from more hardware, because the spare capacity was never where you needed it. The payroll machines dozed while the checkout machines were on fire, and you could not move one ounce of the first to the second. Different rack. Different owner. Different purchase order.


The entire story I'm about to tell you starts with one company looking at all that idle metal, at a scale nobody else on earth had, and deciding it was unacceptable.


## Google's Borg — The Mother Nobody Was Allowed to See


In the early 2000s, Google had the same problem as everybody else, except a thousand times larger and about ten years earlier.


So they built something to fix it, and they called it Borg. After the Star Trek Borg. The collective. The thing that assimilates everything it touches and makes it part of one system.


Borg changed the question. You no longer said "run this program on that machine." You said what you wanted — this much CPU, this much memory, this many copies, keep it alive — and Borg decided where it went, and moved it when it needed to, and restarted it when it died, and never told you which machine, because that was no longer any of your business. To you, ten thousand machines were one computer. They weren't pets any more. They were cattle: numbered, identical, and replaced without a funeral. And because none of them belonged to a team, the spare capacity went wherever the fire was.


By the time anyone outside could read about it, Borg was running hundreds of thousands of jobs, from thousands of different applications, across clusters of tens of thousands of machines. It packed batch work into the gaps left by serving work. It over-committed deliberately, because it knew nobody uses what they ask for. Those rooms of idling metal, at Google, were not idling.


Then Google tried to build Borg again, better, from scratch. They called it Omega, and it let many schedulers work on the same cluster at once. Most of its ideas were folded back into Borg. The rest — one shared record of the cluster that everything watches and nobody owns — would later walk out of the building.


And outside Google, nobody had seen Borg. There was no product. There was no paper about Borg. There were job adverts with strange words in them, and engineers who left Google and went quiet whenever the conversation got specific, and a rumour that somewhere in California there was a way of running computers that made everything the rest of us were doing look like carrying water in our hands.


Omega got a public paper in 2013. Borg, the original, did not get one until April 2015, ten months after Kubernetes was announced.


The outside world read about the rewrite before anyone was allowed to see the original. We met the child first, and spent a year guessing what the mother looked like.

## Apache Mesos — The Answer That Came With Homework


Around 2009, at Berkeley, a graduate student named Benjamin Hindman was building a way to run a whole cluster of machines as one, in the open.


He called it Mesos, and it was elegant in a way that still makes engineers a bit misty. Mesos did not try to know what your work needed. Instead it offered: here are some free machines, do you want them? And your own scheduler — your framework, in Mesos language — said yes or no. Two levels. Mesos handled the resources; you handled the decisions.


That design was more flexible than anything else in existence, and it worked. Hindman took it to Twitter, and the Fail Whale, the cartoon Twitter showed you every time it fell over, started showing up a lot less. It ran at Airbnb. It ran pieces of Apple. In 2013, Hindman and Tobias Knaup and Florian Leibert founded Mesosphere to sell it.


Mesos was first. Mesos was proven at a scale nobody could argue with. And Mesos did not hand you a solution. It handed you a toolkit and a reading list.


Because "bring your own scheduler" means you have to bring your own fucking scheduler. To actually run a web service you also needed Marathon. To run scheduled work you needed Chronos. Twitter wrote Aurora. Every one of those was another component to learn, choose, operate and explain to your boss.


I worked with Mesos. The company I was at back then was building a product meant to run on top of an orchestrator, and we chose Mesos, because everything else was too young. And it worked, and it was amazing, right up until it wasn't. If I'd had more money back then, I'd have spent it on a psychiatrist, to deal with the nightmares Mesos and friends gave me.

What Mesos asked for was an engineering team that wanted to assemble its own orchestration layer out of parts.

Twitter had that team. Apple had that team. Most companies didn't. They just wanted to put their applications somewhere and go home.

## Why the Kubernetes Logo Has Seven Spokes — The Name Hidden in Plain Sight


In the autumn of 2013, three people inside Google looked at Amazon and got frightened.


Their names were Joe Beda, Brendan Burns and Craig McLuckie. What they could see was that Amazon Web Services was turning into something nobody would ever catch — not by selling better virtual machines, because a virtual machine is a virtual machine and Amazon was already there.


But they also knew something the rest of us didn't. Google's real advantage was never the machines. It was Borg. It was the way Google ran the machines. And in 2013 a small company called Docker had just made containers something an ordinary developer could actually use — which meant, for the first time, that Google's way of running software could be handed to somebody who did not work at Google.


What made Google special, the way it ran the machines, was the one thing it never sold. Kubernetes was the first time it tried.


Beda, Burns and McLuckie started a project, and it needed a codename. Borg was already Borg.


They called it Project Seven. After Seven of Nine.


Now. Seven of Nine is a character from Star Trek: Voyager. And she is not just any Borg. She is a Borg drone who gets severed from the collective — cut out of the hive and made to learn how to be a single person in a world that is not the Borg.


The public name came to Craig McLuckie in his car, on the way to work. This thing, he thought, is like steering a container ship. What would you call the helmsman? Kubernetes: Greek for helmsman.


When Tim Hockin drew the logo, he drew a ship's wheel with seven spokes. A ship's wheel almost always has an even number of spokes. Six, or eight, or ten. Hockin drew seven.


Seven spokes. Go and look. It is on every slide, every T-shirt, every conference badge in this industry, and it has been for more than a decade.

They told us exactly what it was. They put the secret in the logo, in public, on day one, and we have been staring straight at it ever since without counting.

## Why Google Open-Sourced Kubernetes — The Argument About Giving Away the Weapon


Before Kubernetes could leave the building, somebody had to say yes. That somebody was Urs Hölzle: Google's eighth employee, and the man responsible for the technical infrastructure underneath everything Google did.


They pitched it to him, and Hölzle said:

"So let me get this straight. You want to build an external version of the Borg task scheduler. One of our most important competitive advantages. The one we don't even talk about externally. And, on top of that, you want to open source it?"


He was right. That was precisely what the mad bastards wanted to do.


The argument that eventually won went like this. Google could not beat Amazon selling virtual machines, apples to apples. But Amazon's real moat was not price — it was that once your application was built into Amazon's particular shapes, it could not leave. Everybody knew it. Everybody was quietly paying for it.


So: destroy the moat. Give the whole industry one common way to package and run software — the same way, everywhere, on anyone's hardware — and applications become portable. And when applications can leave, Amazon has to compete on the merits all over again, on ground Google believed it was better on. You do not win by building a better castle. You win by making castles worthless.


Give away the thing that is not the business, so the thing that *is* the business has to be fought for again. Economists call it commoditizing your complement. It is a genuinely brilliant move.


The yes came from a bus. Craig McLuckie found himself on a Google shuttle sitting next to Eric Brewer, one of Hölzle's key strategists, and by the end of the ride Brewer was convinced. Soon after, Hölzle said yes.


Joe Beda made the first commit on the sixth of June, 2014. Four days later, on the tenth of June, Eric Brewer stood up and announced Kubernetes to the world in a keynote — at DockerCon. Docker's own conference. Docker's own stage.


I was in that room. I'd love to tell you I remember the keynote. I don't. What I remember is the mood. We all used containers, and we all needed them in production, and this felt like something built from the ground up for them.

## Red Hat, the CNCF and the Container Wars — Everybody Picks a Side


Two weeks before Kubernetes was announced, a man at Red Hat called Matt Hicks sent one of his engineers a message: Google wants to know if we're in.


The engineer was Clayton Coleman. He got private access to the code that same day, started cleaning it up, and became the first contributor to Kubernetes who did not work for Google.


And Red Hat did something genuinely reckless. They had a product called OpenShift. It was a platform-as-a-service, it existed, it had customers — and they tore out its insides and rebuilt the whole thing on top of a project that had only just been announced, from a competitor, with no users. Within a year, Red Hat was the second largest contributor in the world, behind only Google.


On the twenty-first of July, 2015, Kubernetes reached version 1.0 — and Google let go of it. They donated it to a brand new foundation, created that day under the Linux Foundation, called the Cloud Native Computing Foundation.


That donation was not generosity. It was the whole plan. Nobody adopts a weapon that still belongs to a competitor — a standard owned by Google is just Google's product wearing a hat. Which also meant Google could never take it back, and they knew it when they signed.


And then the war, which was shorter than people remember. Docker had Swarm, which was simpler and built right in. Amazon had ECS, which was free if you were already on Amazon, and most people were. Mesos was still there, still first, still excellent. HashiCorp had Nomad.


By the end of 2017, it was over. In September, Mesosphere put Kubernetes on its own platform. And in October, Docker announced it would ship Kubernetes inside Docker, right next to Swarm.


Kubernetes did not win because it was the most elegant. It was the most complicated thing on that list by a distance. It won because it was the only one with an entire industry standing behind it, all of them having agreed, in a foundation, in public, that this was the one.


The losers settled their affairs, one by one. In 2019, Docker sold Swarm, along with its whole enterprise business, to a company called Mirantis, which kept it going for the customers who stayed. That same year, Mesosphere renamed itself D2iQ and went to work on Kubernetes, and in 2023 it sold what was left to Nutanix. In August 2025, the Apache Foundation retired Mesos. Almost nobody noticed.

## Amazon EKS and Microsoft AKS — The Ones Who Collected


Amazon had absolutely no reason to help Kubernetes. They had their own answer, ECS, and they had the customers, and the entire point of Kubernetes was to make their customers portable. Being asked to sell it was like being asked to sell the crowbar.


So they didn't. For years. And their customers ran Kubernetes on Amazon anyway — by hand, on raw virtual machines, wiring up their own control planes, paying Amazon for the machines while getting no help whatsoever with the hard part.


In November 2017, Amazon gave in and announced EKS. It went generally available the following June.


And Microsoft did something even simpler. In 2016, they hired Brendan Burns. One of the three people who created it. He said Microsoft was all-in on cloud, that it was his home-town team, and that there was something appealing about a company in transition. Azure built AKS into a flagship.


In the middle of 2026, eleven years after Google gave Kubernetes away, Amazon was still the biggest cloud on earth. Microsoft was second. Google was third.


Right. This next part is me, not the record, so take it as one man's reading.

The strategy worked. It worked exactly as designed. Lock-in genuinely weakened, portability genuinely happened, and the way you package and run software genuinely did become the same everywhere.
Google won the argument. Google set the standard the entire industry now runs on, and I'd argue GKE is still the best managed Kubernetes anybody offers.

And Google is third. The company the weapon was aimed at is still the biggest cloud on earth, and the company in second place hired one of the people who wrote it.

You built the crowbar. You handed it out for free, on principle, to make sure nobody could accuse you of owning it. And then you watched two other companies use it to open doors you were standing outside of.

## Kubernetes vs Borg: What You Actually Got — Borg's API Without Borg's Datacenter


Kubernetes today is the part you already know. You got the interface. Declare what you want, and a loop somewhere works patiently and forever to make the world match the document. That idea is genuinely beautiful, it came straight out of Borg and Omega, and it is why Kubernetes' core API has barely changed since 2015.


What you did not get was everything underneath it. At Google, Borg was never alone. Borg sat on top of Google's storage, Google's network, Google's naming and locking service, Google's build system, machines Google designed for itself, and a decade of tooling built by people whose entire job was to keep that fleet healthy. None of that came in the box. None of that could come in the box. What was portable was the shape of the thing.


The team building Kubernetes knew how Borg had grown. It began as one big monolithic master, and over the years everything it didn't do — autoscaling, scheduled jobs, workflows — was added around it as separate services. So they built Kubernetes the other way round from the start. At the centre sat the API server: Omega's shared record, out of the building at last, keeping track of what you asked for and nothing else. Everything else is a separate piece that watches it and plugs into it. Anything missing could simply be added.


It worked. Anything missing was added. Then everything else was too.


Extensibility is why you have custom resources. And operators. And an ingress controller you argue about. And a service mesh that somebody installed in 2022, who has since left, and nobody remembers what the hell it does. And a landscape diagram with hundreds of logos on it that gets shown at conferences as evidence of a thriving ecosystem. It is. It is also a map of everything you now have to choose between.

And it is why you have a platform team.

At Google, the twenty-odd pieces underneath Borg were designed together and kept together by the same people. Out here, each of them exists five times over, from hundreds of projects that never met, each on its own release schedule, and somebody has to choose them, wire them together and keep them working through every upgrade. That somebody is your platform team.

## Kubernetes for AI Training: Slurm and Gang Scheduling — The Job That Waits


Recently, the work itself changed shape. Serving a finished model is mostly a web service: keep it alive, restart it when it dies, add copies when it gets busy. Kubernetes is superb at that, because Borg was superb at that, because that is what Google needed. Training a model is something else.


A training run is the older kind of work. Five hundred and twelve machines, together, for four days, and either they all start or there is no point starting.


Kubernetes schedules each piece of your job on its own, and gets everything there eventually. That is not a bug. It is the founding idea. But "eventually" is not a thing a training run can do. You submit a job that needs four machines, and three of them start, and the fourth sits there saying `Pending` because somebody else took the last GPU. And now about a million dollars of silicon is holding perfectly still, waiting for a friend.


What that job needs is a scheduler that will take all of it or none of it. That is called gang scheduling, and it is not new, and it is not clever, and it has been sitting in a batch scheduler called Slurm since 2002. So the newest, richest, most breathlessly discussed workload in the history of computing went and got itself one of the oldest schedulers in the building.


A common arrangement in 2026 is Slurm for the training and Kubernetes for the inference. Some put Slurm on top of Kubernetes, so Kubernetes runs the machines and Slurm decides who gets them.

Meanwhile the ecosystem is bolting the missing half on. Kueue. Volcano. Dynamic Resource Allocation. And KAI, a scheduler from NVIDIA. Gang scheduling and topology awareness, arriving as additions beside the scheduler rather than inside it.


And in December 2025, NVIDIA bought the company behind Slurm.


Train on Slurm, and your scheduler is now made by NVIDIA. Train on Kubernetes, and there's a good chance the piece deciding whether your GPUs sit idle came from NVIDIA too.

Google built the crowbar and watched other people collect. This time, the company collecting doesn't even have to choose a side. It sells the GPUs to both.

## Goodnight


Kubernetes did not fall. Nothing in this story is coming for it. It won so completely that it stopped being a decision — it is the floor now, and you are standing on it whether you chose it or not.


And as far as anyone outside is allowed to know, Borg is still running. Same job, same buildings, twenty-odd years in, quietly and competently, for the only company that ever had everything it needed. She never left home. She never had to be everything to everybody.


Goodnight, kids. Your platform team is still assembling the other twenty things.

Borg had them in the box.
