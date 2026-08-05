
+++
title = "Docker's Rise and Fall: The Nightmare of Winning Too Well"
date = 2026-08-10T16:00:00+00:00
draft = false
+++


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the tale of Docker. The whale who carries the world.


And like all the best bedtime stories, it starts with a dream, it's full of wonder in the middle, and it ends with everyone screaming.


Because this isn't one of those stories where the hero loses. Oh no. This is worse. This is the story of a hero who *won*. Who won so completely that his name became a verb, that he conquered every data center on the planet, that he changed how the entire industry ships software forever.


So tuck in. Because tonight's nightmare is the scariest kind there is. The kind where you do everything right, and it still isn't enough.

<!--more-->

{{< youtube kmLztKueTcI >}}

- TODO: Every paragraph below is tagged with what's on screen while it's narrated.
- TODO: clip: <name> — a drop-in illustration clip. LOOP it for as long as that paragraph's narration runs; don't change its speed. A tag applies to the paragraph(s) beneath it until the next tag.
- EXCEPTION — "zoom" clips are rendered slow and must NOT be looped: speed up / retime them to fit the narration. These are the cold-open clips (`01-cover`, `02-title`), every chapter card, and any marker flagged "do NOT loop".
- TODO: clip: bedtime-docker-chapter-N — a chapter-title card: the storybook re-opens to a fresh page reading "CHAPTER N / <name>". The card STAYS on screen while the narrator reads the chapter's opening paragraph (the one directly beneath it) as voice-over; then cut to the first illustration. The on-screen text is the only title — nothing is read aloud for the card. Retime to VO, don't loop.
- TODO: talking head — cut to the on-camera narrator (no clip). Each chapter opens on its chapter card (VO) and closes on a talking head.

## Once Upon a Time


Once upon a time, there was a little company called dotCloud.


And dotCloud was, let's be honest, not doing great. They were trying to sell a platform for running other people's apps, and so was everybody else, and the money was running low. This is the part of the fairy tale where the family is poor and the winter is coming.


But dotCloud had this little internal tool. A thing they'd built just to make their own lives easier. A way to wrap up an application, with everything it needed to run, into a neat little box that would behave exactly the same no matter where you put it.


Now, the technology inside that box wasn't new. The Linux kernel had the pieces lying around for years. Namespaces, cgroups, all that plumbing. It was there. It was just a miserable, arcane pain in the ass to use. Only wizards touched it.


What the little company did was take that horrible plumbing and wrap it in something a normal human could actually use. Three commands. Build. Ship. Run. That's it.


And in 2013, instead of keeping it locked in a drawer, they did something a little bit crazy. They gave it away. They stood up on stage, showed the world their little tool, and open-sourced the whole thing for free.


They called it Docker. And its mascot was a friendly blue whale, carrying your containers on its back.


Remember that the tech was free. That the whole thing, the crown jewel, was given away for nothing. Because that, my friends, is the splinter under the fingernail of this entire story.

## The Whale Everyone Loved


And the world went absolutely insane for it.


You have to understand how big this was. Before the whale, getting software to run the same on your laptop and on the server was genuine hell. "Well, it works on my machine" was a real sentence people said, right before a fistfight. Deployments were rituals involving prayer and human sacrifice.


The whale made all of that just... go away. Wrap it in a container, and it runs the same everywhere. Every developer who tried it had the same reaction, which was roughly: "oh my god, where has this been all my life."


Adoption exploded. Not grew. *Exploded.* In a couple of years the whale went from a weird demo to the thing every company on earth was scrambling to adopt. "Docker" stopped being a product and became a verb. You didn't containerize your app, you "dockerized" it.


And the money men came running. Investors threw hundreds of millions of dollars at the whale. Valued the little company at over a billion. A unicorn. The darling of the entire industry. On paper, our whale was one of the most beloved, most valuable, most important creatures in all the land.


Everybody used Docker. Everybody loved Docker.

Almost nobody... paid Docker a single cent.

## The Crown Nobody Could Sell


Here's where the wonder starts to curdle. Here's where I have to tell you the uncomfortable part.


The whale had conquered the world by giving the best part away for free. And now the whale had to eat. Investors don't hand you hundreds of millions of dollars because they think your whale is adorable. They want it back. Ten times over.


So the little company had a problem, and it was a brutal one. How do you make money selling something everyone already has for free?

And the honest answer is... they never really figured it out.


They tried everything. They tried selling to enterprises. They tried a hosted registry. They tried premium features. They tried being a platform, then a different platform, then a slightly different platform again. Leadership changed. Strategy changed. The story they told investors changed every year or two.


They were sitting on the single most-adopted piece of infrastructure software of the decade, and they were bleeding cash trying to figure out how to charge for it. It's like owning the water that every human on earth drinks, and slowly going bankrupt, because you already promised everyone the water was free and you can't take it back.


Adoption is not revenue. Say it with me. *Adoption is not revenue.* The whale had all the love in the world and an empty treasure chest. And love doesn't make payroll.

## The War for the Helm


Now, while the whale was busy trying to find its wallet, a much bigger, much more dangerous problem was swimming up from the deep.


See, containers were wonderful, but people quickly had thousands of them. And something needed to be the captain. Something needed to decide where they all go, restart the ones that die, wire them together. Orchestration. Whoever controlled *that* controlled the future.


The whale had its own answer. It was called Swarm. And it was good. It was simpler, it was cleaner, it was right there built in.


But out of the deep came a leviathan. Kubernetes. Born inside Google, from people who'd been running containers at a scale the whale could barely dream of. And behind it, a whole alliance of the biggest companies in tech, all deciding, together, that *this* was going to be the standard.

And the war was... not close. Not even a little bit.


Kubernetes ate everything. It became the captain of the entire ocean. And the whale's own answer, Swarm, got left behind, gasping on the beach.


And here's the moment that should make you wince. 

The whale had to give up. Docker ended up shipping *Kubernetes itself* inside its own tools, because that's what everyone demanded. The whale had to carry its conqueror on its own back. It lost the single most important battle of the container era, using its own technology, on its own turf.


The crown was slipping. And the whale still couldn't pay its bills.

## The Night They Sold the Ship


And so we come to the dark part of the story. The part I warned you about.


The founder, the one who'd stood on that stage and given the whale to the world, left the company. The dream had a father, and the father walked away into the rain.


And then, in late 2019, came the moment nobody dancing in the streets back in the glory days would have believed. The company that everyone thought would be the next great giant of infrastructure... sold itself off for parts.

They sold the entire enterprise business, the thing they'd bet the company on, to a company called Mirantis. Chopped the whale in half and sold the bigger half. What was left was a smaller, quieter thing. Refocused. Humbled. Told to go back to just making tools for individual developers, the thing it was always actually good at.


No billion-dollar giant. No next VMware. No triumphant IPO with confetti and yachts. The unicorn got quietly led out the back door.


The whale survived. It's still here today, honestly doing okay. Profitable, even. Beloved by developers, still. But these days it mostly plays dress-up — a little whale in a lion costume, roaring at the mirror, pretending to be the king it was supposed to be. Because once upon a time, this thing really was going to be a lion.


That's where the story should end. The humbled little company, making its little tools, quietly profitable. A sad ending, sure — but a clean one. Except this story doesn't get a clean ending.

## The Nightmare


And now, children, here is the nightmare. The one you'll think about after the lights go off.


The whale won.

I mean it. Go look. Right now, this very second, the whale's idea is running the entire internet. Every cloud, every pipeline, every deployment. It didn't just succeed, it became the literal air the whole industry breathes. The technology is immortal.


But the company that made it is a ghost. And that's the horror. Not to fail, but to win so completely that your victory floats free of you entirely and drifts off into the world without you. To be everywhere, and nowhere. Immortal, and forgotten, at exactly the same time.

They built the future. They just forgot to keep a room in it for themselves.

## Goodnight


Sweet dreams. Try not to think about how everything you use was built by someone who never got paid for it.
