
+++
title = "Heroku Is Dead (And It Taught Us Everything We Know)"
date = 2026-10-05T15:00:00+00:00
draft = false
+++


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the story of Heroku. The best place to run software that anybody ever built, and one of the least important things its owner has ever owned.


Your company has a platform team. Maybe you're on it. They're building an internal developer platform. It has a name with "launch" or "forge" or "runway" in it, a portal, a golden path, and a YAML file you fill in to get a database. It has been in progress for about two years.

Hold onto that. Because everything on that roadmap already shipped. Finished, priced and documented, in 2011. And the thing that shipped it is still running tonight — you can sign up in under a minute.

Your platform team is going to build it anyway.

Some of you deployed to it every day. Some of you have never heard the name.

Those two facts are the same story.

<!--more-->

{{< youtube 3CLrqv6ZtW8 >}}



## Heroku's Founding and git push heroku — The Command That Did Everything


Once upon a time, in June 2007, three people decided that writing software should be as easy as opening a web page.


They were called James Lindenbaum, Adam Wiggins, and Orion Henry. And to understand what happened to them, you have to remember what shipping code actually meant in 2007.



You rented a server. You installed an operating system on it. You installed a language runtime, and a web server, and a database, and then you spent an evening discovering that the version of the library your application needed was not the version your operating system had opinions about. You wrote a deployment script. The deployment script worked on your machine. You SSH'd into the server at eleven at night and ran it, and something in the middle failed, and you fixed that, and then something after it failed.


The application was ten files long. Getting it onto the internet took a weekend, and the weekend was not the interesting part.


So they named their company Heroku, which is "heroic" and "haiku" pushed together, because they wanted the difficult thing to come out short. They went through Y Combinator in the winter of 2008. And their first product was completely wrong.



It was a code editor that ran in the browser. You would write your Ruby application in a web page, and Heroku would run it. It was genuinely impressive, and people signed up, and then they did not come back, because almost nobody wants to write software in a text box.


But the founders noticed something in the logs. The people who did stay were not using it to write anything. They were pasting in code they had already written somewhere else, because Heroku was the fastest way they had ever found to get it running on the internet.


The editor was the product. The machinery that took your code and ran it on the internet was the plumbing underneath, built to make the editor possible, and nobody was ever supposed to look at it. Their customers had quietly informed them they had it the wrong way round.


So they threw away the product. The plumbing became the company.


Heroku launched properly in April 2009, and what it launched with was one command.


You wrote your application. You typed `git push heroku master`. And that was the entire deployment procedure. No server. No operating system. No runtime to install, no web server to configure, no script that worked on your machine and nowhere else. You pushed your code the same way you already saved your code, and a few seconds later there was a URL, and the URL worked.


I deployed to Heroku before I understood what I was doing. That's not modesty. I genuinely did not know what a web server was, and it didn't matter, because nobody asked me.

Before that, the gap between "I've written something" and "other people can use it" was measured in evenings. Heroku made it one line in a terminal I already had open. Not faster. A different category of thing. It deleted a whole class of work from my job, and it deleted it so completely that people who started after me never found out the work had existed.

And here's the part I keep having to explain in meetings. That one line is still the target. Not a nice bit of history — the target. Almost every internal platform I get shown has a slide near the front promising developers something very close to it. Seventeen years on, with far better technology underneath, and most teams are still building towards it.

That's the highest compliment you can pay a piece of infrastructure. It's also the murder weapon.

## Heroku Cedar, Buildpacks and Twelve-Factor — The Rules They Gave Away


Heroku named its platform generations after trees, in alphabetical order.


The first was Argent Aspen, in 2009. It ran Ruby, and only Ruby. The second was Badious Bamboo, in 2010. And in 2011, Heroku shipped Celadon Cedar, which is the one that mattered.



Cedar was the moment Heroku stopped being a Ruby company. It ran Ruby, and Node, and Python, and Java, and Scala, and Clojure, and it did that through a thing called a buildpack: a small, replaceable piece of logic whose only job was to look at your code, work out what it was, and turn it into something runnable.


That word, buildpack, should sound familiar to some of you, and it should sound familiar for a reason. If you deploy to DigitalOcean's App Platform, or you use GitLab, or Google Cloud, or HashiCorp's Waypoint, or kpack — you are using buildpacks. Today. This week.

Heroku invented them here, in 2011, for itself. Hold onto that, because where they end up is the strangest thing in this entire story.


Underneath, your application ran in a dyno. A dyno was a container. A real one — isolated with Linux namespaces and cgroups, the same kernel primitives Docker would use later. It held no memory of yesterday. If it died, another one started. If you needed ten more, you moved a slider and there were ten more.


Heroku didn't invent containers — the Linux primitives had been sitting there for years. But nobody had built the tooling to make them usable, so Heroku wrote their own. They were running containers in production in 2009. Docker went public in 2013. Four years.

And Docker came out of dotCloud, which was a Heroku competitor. They built container tooling to run their platform, worked out the tooling was worth more than the platform, threw the platform away, and became Docker.

Heroku had the same thing and bet the other way. Keep the containers underneath, sell the experience on top. That bet is exactly why Heroku was so good. It also handed somebody else an entire industry.


Heroku's other invention of 2011 was not a piece of software at all.

Adam Wiggins wrote down the rules. Twelve of them. Keep your configuration in the environment, not in the code. Treat your backing services as attached resources. Build, release and run are separate stages. Keep your processes stateless. Treat logs as a stream of events, not as files. Make your application disposable, so it can start fast and stop cleanly.


He called it the Twelve-Factor App. And Heroku published it on the open internet, for free, with no licence and no catch, where anybody at all could read it.


I remember this coming out. I couldn't tell you the date, but I remember reading it and just going: "Fuck yeah. Obviously. That's how we should be doing this".

There was nothing to buy. Nothing to install. It was somebody writing down what good looks like, and it was so obviously right that arguing with it felt stupid.

Go and read it tonight. It's fifteen years old and most of it hasn't moved.


There was one more thing, which is the reason so many of you have a feeling about this company rather than an opinion about it. Heroku had a free tier, and it lasted for over a decade. Anyone with no money and no credit card could put a real application on the real internet and send somebody the link.


The free dyno went to sleep after thirty minutes of nobody visiting, and woke up slowly when somebody finally did, and everybody knew it, and nobody cared, because the alternative was nothing at all.


Every pet project I had ran on Heroku. Every single one. Free, for years.

And the company I worked for? Didn't run a thing there. Not one application. We paid other people for that.

Same engineer. Same week, sometimes. Two completely different answers, and the only variable was whose money it was.

That's anecdotal, and plenty of companies did pay them. But I don't think I was unusual. Heroku taught an enormous number of us what deployment even is — your first application on the internet, your first environment variable, your first database you didn't have to install — and it did all of that teaching on the house.

It won every heart in this industry, and almost none of the purchase orders.

## Salesforce Acquires Heroku — "Ruby Is the Language of Cloud 2"


On the eighth of December 2010, Salesforce announced it was buying Heroku for about two hundred and twelve million dollars in cash.


Marc Benioff explained why. The next era of computing, he said, was social, mobile and real-time. He called it Cloud 2. And then he said the sentence that this entire episode hangs from.


"Ruby is the language of Cloud 2."


Ruby? Seriously? To be honest, Ruby was right at the top of its arc that year... until we started asking "why the fuck is this written in Ruby?"

More importantly, lesson learned is that any platform that chains itself to one language is going to fail, and that was as obvious in 2010 as it is today. Containers are the proof. The entire point of them is that how you run an application has nothing to do with what it's written in.


The real reason for the purchase was better than the stated one, and it was not stupid at all.

Salesforce was extraordinary at selling software to the people who sign purchase orders. It was terrible at reaching the people who write code. It had built its own developer platform, Force.com, with its own proprietary language called Apex, and developers had looked at Apex and quietly declined.


Apex? Seriously?

A company that couldn't get anyone outside its own customer base to touch its programming language, announcing to the world which language the next era belongs to.

And that is exactly why they were shopping. They didn't buy Heroku because they understood developers. They bought Heroku because they had proof they couldn't reach them.


Heroku came with roughly a million Ruby developers already attached to it. Salesforce was not buying a product. It was buying a constituency it had never once managed to earn on its own.


Now, this is the bit where people start saying the founders sold out. That's bullshit.

Two hundred and twelve million in cash, in 2010, for a four-year-old company? That's not selling out, that's winning. Anyone who tells you they'd have turned it down is lying to you, or has never had to make payroll on a Friday.

## Heroku Enterprise, Docker and Kubernetes — Building for Procurement


Salesforce did not abandon Heroku. Quite a lot happened over the next few years, and it is worth listing what, because the list is the diagnosis.


In 2014, Heroku shipped Heroku Connect, which synchronised your application's data with Salesforce. In February 2015, Heroku Enterprise. In January 2016, Private Spaces — isolated networks, for companies that needed their applications inside a boundary. In June 2016, Teams. In June 2017, Heroku Shield, for organisations with HIPAA and PCI obligations.


Look at who every single one of those is for. Network isolation. Compliance boundaries. Data residency. Enterprise contracts. That is not a list of things a developer wants. That's a procurement checklist.

They paid two hundred and twelve million dollars for a million developers, and then spent six years building compliance features for insurance companies.

And here's the thing about that list. Compliance gets you through the paperwork. It doesn't get you adopted.

What gets a platform adopted inside a big company is being able to say yes. Yes to the strange networking. Yes to the stateful thing. Yes to the one process nobody is allowed to change. Heroku built everything you need to pass an audit, and nothing you need to say yes.

So they stopped serving the developers they'd bought, and built for an enterprise that still couldn't standardise on it. Six years, aimed between two customers, landing on neither.


Meanwhile, outside, the ground moved.


Docker arrived in 2013 and made containers something anybody could hold in their hand. Kubernetes arrived in 2014, and in 2015 Google gave it away to a foundation, so that it belonged to nobody.


And Kubernetes was, and still is, substantially worse than Heroku at the thing Heroku did. It's enormous. It's complicated. Nobody has ever typed one command into Kubernetes and had their application running thirty seconds later. Ask anyone who has spent an afternoon reading a YAML file out loud to a colleague, trying to work out which of the four indentation levels is the one that's wrong.


But Kubernetes could be extended. By anyone. In any direction.

Red Hat built OpenShift on top of it and sold it to banks. VMware built Tanzu. SUSE bought Rancher. Amazon, Google and Microsoft each sold a managed one. Datadog built a business watching it. An entire economy of a thousand companies grew on top of Kubernetes, and every one of them had a commercial reason to go out and evangelise the thing they were standing on.


Now, Heroku had a marketplace too. It launched with eighty-five add-ons in 2012. By 2015 there were about a hundred and fifty. Today there are a bit over two hundred.


Eighty-five, to two hundred, in fourteen years. That's the number I keep coming back to, and I think it's the actual cause of death.

Kubernetes did not win because it was good. Kubernetes is not good. Kubernetes won because it made a thousand other companies rich, and a thousand rich companies will carry you anywhere you want to go.

Heroku's abstraction was so complete, so beautifully sealed, that there was nowhere for anybody else to attach anything. You could sell Heroku a database to plug in. You could not build a product on top of Heroku and sell it, the way Red Hat built an entire enterprise business on top of somebody else's scheduler. There was no surface. And so nobody in the industry had a single financial reason to argue for Heroku in a meeting, ever, except Heroku.


On the twenty-fifth of August 2022, Heroku announced that the free tier was ending. It was gone by the twenty-eighth of November. The stated reason was fraud and abuse.


By 2022, the free tier wasn't teaching anybody anything any more. Nobody was learning to deploy on Heroku. They were learning on Docker, and on Kubernetes, and on whatever their company had already standardised on years earlier.

So killing the free tier wasn't the moment Heroku lost the developers. It was the moment they admitted it had already happened.

## Heroku Fir and Sustaining Engineering — The Right Answer, Eleven Years Late


In April 2025, after fourteen years, Heroku planted a new tree.


They called it Fir, and Heroku's own announcement was titled "Planting New Platform Roots in Cloud Native."


Fir was a genuine rebuild. Underneath, it ran on Kubernetes. It ran on ARM processors. It used OCI container images and Cloud Native Buildpacks, which meant that for the first time you could build your application on Heroku and then run that exact artifact somewhere else, on your own machine, in Docker, anywhere.


And it had native OpenTelemetry. Traces, metrics and logs, from your application and from the platform underneath it, in an open standard, exported anywhere you liked.


Heroku had gone and fixed it. The sealed box that nobody could see into, and nobody could build on, could now be seen into and built on.


And that is the right answer. Properly done, as well — not a patch on the old thing, a rebuild. Everything that had been wrong with Heroku for a decade, they went and fixed.

Eleven years after it would have mattered.


Ten months later, on the sixth of February 2026, Heroku's chief product officer, Nitin T Bhat, published a post called "An Update on Heroku."


It said that Heroku was moving to a sustaining engineering model, focused on "stability, security, reliability, and support." It said that Enterprise Account contracts "will no longer be offered to new customers," though existing ones would be honoured and could renew. It said there would be no change for customers using Heroku today.


It did not say the platform was closing, because it isn't. There is no shutdown date. You can still sign up tonight with a credit card, and it will still work, and it will still be lovely.


It said the investment is going to enterprise AI instead.


"Sustaining engineering." Nobody was confused by that.

The industry decoded it inside a day — the migration guides went up almost immediately. It means: start planning your exit. Stay as long as you need to, support will be there, your bill will keep arriving. And do not plan on still running on Heroku in five years.

It is the politest possible way of telling several thousand companies to leave.


So why would anyone shelve a product that works, and has customers, and makes money?

Arithmetic.

Salesforce does something like forty-six billion dollars a year. Nobody publishes Heroku's number, but nothing I can find puts it above a fraction of one percent of that. A rounding error that still needs security patches, compliance audits, incident response and a product team — while the board asks what you're doing about AI.

Killing it would be stupid. It makes money. Investing in it would also be stupid, because nothing it could possibly become would move that needle.

So you keep it alive and you stop feeding it. I'd probably sign the same memo.

And the best developer experience anyone has ever built ends up as a maintenance line item.

## Goodnight


Heroku is not dead. It is running tonight, quietly, competently, in a way that would embarrass most of what we have replaced it with.


And every application on it can leave whenever it likes — because Heroku is the company that taught this industry how to build applications that can leave.


Its ideas did not merely survive. They won completely.


The twelve factors are simply how software is built now. Configuration in the environment. Stateless processes. Logs as streams. Disposable, replaceable, identical instances. It is in your Dockerfile. It is in your Helm chart. It is in the pull request you approved this afternoon.


Most of the people following those rules have never heard the phrase "twelve-factor app", let alone the name of the company that wrote them down.


And the buildpacks Heroku invented became one of the Cloud Native Computing Foundation's top-tier projects in August 2026. Six months after Heroku was told to stop building things.


And your platform team is still going. Two years in. Six engineers. A portal, a golden path, a YAML file you fill in to get a database. Building something that looks a great deal like Heroku, for a company that could never have used Heroku.


Goodnight, kids.

Sleep well.

Your golden path is in review.
