
+++
title = "AI Just Killed the Software Engineer (And Created Something Better)"
date = 2026-04-13T16:00:00+00:00
draft = false
+++

The software engineering profession is being rewritten right now. Not slowly. Not in some distant future. Right now. And most companies are responding to it in exactly the wrong way.

This video is going to cover a lot of ground, so let me tell you upfront what you are getting into.

We will start with the numbers. Hard data on what AI is already doing to productivity, revenue, and the way code gets written. Then we will get into the human side of this, because behind those numbers are real people going through an identity crisis, and I do not think enough people are talking about that honestly.

From there, we will dig into what actually changes about how software gets built. New development cycles, spec-first workflows, how much autonomy you give AI agents, and why context engineering is the single biggest factor most teams are ignoring.

Then comes the part that makes people uncomfortable. How you restructure your company. Org design, team composition, pod models, platform teams, which roles expand, which ones shrink, and what happens to junior developers when nobody is hiring them anymore.

We will wrap up with concrete steps you can take Monday morning. No vague advice. Specific moves.

This is based on my personal experiences, on working with and talking to a lot of companies, seeing what works and what does not, and doing a lot of analysis. Let's get into it.

<!--more-->

{{< youtube sDy55DIkIJ0 >}}

## AI Impact on Software Engineers


Let me hit you with some numbers that should make you uncomfortable.


Cursor, the AI coding tool, generates roughly three point three million dollars in revenue per employee. The median SaaS company? A hundred and thirty thousand. That is a twenty-five-x gap. Let that sink in for a moment.


OpenAI shipped the Sora Android app with four engineers in twenty-eight days. Not forty. Not four hundred. Four people.


AI is now involved in writing forty-one percent of all code globally.

If your company has not fundamentally restructured how it builds software, you are already falling behind. And not by a little. By an order of magnitude.



So what actually changed? The bottleneck moved. Writing code used to be the hard part, the expensive part, the thing that took forever. That is no longer the case. Code is now cheap and abundant. AI can generate it faster than any human ever could.


The scarce resource is now everything around the code. Before it: architecture, specifications, context, making sure AI has the right inputs to produce something useful. After it: verification, review, judgment. Can you tell whether the code AI just produced is actually correct? Does it fit with the rest of the system? Will it hold up under real-world conditions?



Microsoft ran an experiment with AI-native engineering and found that engineers ended up spending seventy-three percent of their time on strategic work: planning, architecture, and reviews. Pure implementation dropped to single digits. Down from roughly fifty percent.


The role of a software engineer has shifted from coder to orchestrator. If your entire value was being a glorified typist who translates English into Go or Python, I have bad news for you. That job is gone. You were never an engineer. You were a code monkey, and AI is a better code monkey than you will ever be. Your value now is in the thinking, not the typing.


Now, let's talk about what this actually feels like. Because behind all those numbers are real people going through something genuinely disorienting.


An anonymous junior engineer put it bluntly: "I'm basically a proxy to Claude Code. My manager tells me what to do, and I tell Claude to do it." That is a real person describing their real job. Meanwhile, senior engineers are thriving, reporting massive productivity gains. The gap between those two experiences is staggering.


Computer science enrollment across the University of California system declined six percent in 2025. First time that has happened since the dot-com bust. Students and parents are looking at this industry and wondering if there is still a career here.


And here is the paradox. Eighty-four percent of developers use AI tools, but trust in their accuracy dropped from forty-three percent to twenty-nine percent. Developers are using tools they do not trust because they feel they have no choice.


There is grief in this moment. Anxiety. An identity crisis for an entire profession.

And people are splitting into three camps.

First, those who adapt and thrive. Mostly seniors who lean into the shift and see their output multiply.

Second, those who are anxious but engaging. Trying to keep up, unsure if what they are learning today will matter tomorrow.

And then there is a third group that might be in the worst position of all. People at every level of seniority who reject AI outright or simply are not becoming proficient with it. Some out of principle, some out of stubbornness, some because nobody forced them to.

At least the anxious ones know the ground is shifting. The ones who ignore it are the ones who will be blindsided.

And yes, there are real issues with AI and agents today. Nobody is denying that. But here is the thing. It has been barely over a year since we got proper coding agents. A year. That is nothing. We are supposed to see issues this early in any technology. The fact that it already makes a massive difference despite being this immature should tell you something about where it is heading.


The question is not just whether you adopt AI. It is whether you invest in it properly, whether you learn to do it well, whether you keep adapting as the landscape shifts underneath you. There are people and companies doing this right, and they are thriving. There are those doing it poorly or not at all, and they are falling behind. At this point, **denying the benefits is just as dangerous as ignoring the problems**.


Let me ask you something. What do you actually want to be doing? Do you want to be thinking about problems, designing systems, figuring out what users actually need? Or are you genuinely passionate about writing getters and setters in Java? Is cranking out boilerplate code really what gets you out of bed in the morning, or is it just the tedious means to an end, the part you slog through to get to the work you actually care about?

Because AI just took that tedious part off your plate. Boilerplate, scaffolding, test generation, documentation, routine refactoring. The repetitive, well-defined work that frankly was never the interesting part of the job anyway. AI handles that now, and it handles it well.

What humans are still irreplaceable for is the work you probably wanted to be doing all along. Architecture, business context, creative problem-solving, debugging complex systems, ethical judgment. The things that require **understanding why a system exists**, not just how it works.

Then there is a whole category where you need both. Code review, complex testing, multi-component integration, large-scale refactoring. AI can do a first pass, but a human has to validate and make the final call.

And finally, there is work that needs neither human nor AI intelligence. Pure automation. CI/CD execution, linting, scaling, monitoring. These should just run. No brain required, carbon or silicon.


The thing most teams are discovering the hard way is that the majority of developers rate the effort of reviewing, testing, and correcting AI output as moderate or substantial. You do not just press a button and ship. The generation is fast. Everything around it still takes serious work.

So if humans handle the thinking and AI handles the typing, where do the ideas and direction actually come from? That part has not changed. Humans still own strategy. But AI has dramatically accelerated how fast you can go from idea to something tangible.


Discovery cycles that used to take ten weeks are getting compressed to four. AI can synthesize customer feedback, cluster themes, and generate prototypes in hours instead of weeks. The culture is shifting from "let's talk about it first and build later" to "show me what it looks like right now."



And here is what really changes. You no longer have to be sure you are doing the right thing before you start building. The cost of being wrong just dropped through the floor. You can produce something fast, put it in front of people, see if it sticks, and if it does not, throw it away and try something else. And then something else. And then something else. You keep iterating until you land on a hit. What used to be a carefully planned, months-long bet is now a rapid sequence of cheap experiments. The winners are not the ones with the best first idea. They are the ones who can try ten ideas in the time it used to take to plan one.


And that cheapness of building does not just speed up execution from the top. It changes who can initiate ideas in the first place. When creating something takes days instead of months, you do not need executive approval to explore. Claude Code started as a side project inside Anthropic. Nobody directed it from the top. Now it is a core part of their strategy. That kind of bottom-up innovation becomes far more viable when the cost of trying is close to zero. Engineers building tools that help their own work, small teams spinning up prototypes for new products, individuals scratching their own itch and discovering it is everyone's itch. The cheapness of building flattens the hierarchy of innovation itself.

## AI Software Development Lifecycle

All of this changes how we build software at a fundamental level. The traditional SDLC is dead. Or at least, it should be.


The new cycle looks something like this: specify, plan, generate, validate, deploy. The specification is the source of truth. Code is a generated artifact. Disposable. Regenerable. If the spec is right, you can throw the code away and regenerate it. That sounds radical, but it is already how the most productive teams operate.


The scope of each cycle matters too. You are not building entire features in one pass. You are not writing single lines either. The right size is a small, isolated, reviewable, testable task. Each one is essentially a TDD checkpoint. You specify what it should do, AI generates it, tests verify it, you move on. The cycle time is hours or days, not weeks or sprints. And when the cycle compresses that much, it forces you to rethink everything. How you organize teams, who does what, what roles still make sense. We will get to all of that.


And when I say spec first, I mean it literally. Not design first. Not MVP first. Spec first. Lightweight, testable specifications before anyone writes or generates a single line of code. Then AI generates a prototype in hours. Design and code converge because the gap between "what it should do" and "a working version" shrinks to almost nothing.

This changes what every role produces. Architects do not need to argue over diagrams anymore. They can spin up a proof-of-concept that actually proves the architecture works. API designers do not need to start with markdown files and OpenAPI schemas. They can generate a working API and test it directly. Product managers do not need slide decks to get stakeholder feedback. They can show something that actually runs.

Across the board, the preparation work that different roles used to do, the diagrams, the specs, the schemas, the wireframes, all of that is collapsing into producing working code that validates ideas right away. Not production-ready code. But real, running code that proves the concept.


Now, to be clear, AI belongs in both prototyping and production. The difference is not whether you use AI but how much rigor you wrap around what it produces. For prototyping, you can move fast and loose. Generate, test the idea, throw it away if it does not work. For production, you need the full discipline: review, validation, testing, security checks, architectural oversight.


Sixteen out of eighteen CTOs in one survey reported production disasters directly caused by AI-generated code that skipped that discipline. The tool is the same. The process around it is what separates a prototype from a production system.


So how much autonomy do you actually give the agents? The operating model that is emerging is simple: **delegate, review, own**. AI handles the first pass. Humans validate. Humans own the outcome. AI is not here to replace people. It is here to augment them. At least when we are talking about production code, AI is not supposed to do it alone.



And that means something uncomfortable.


Everyone will produce more. But the quality of what you produce depends entirely on your experience, your judgment, your taste. AI is an amplifier. It multiplies whatever you already are. A senior engineer with strong architectural judgment and AI produces incredible output. A junior with no context and AI produces a mountain of code fast, but a lot of it will be wrong in ways they cannot even detect. Same tool. Wildly different results. The gap between good and bad engineers does not shrink with AI. It explodes.


Here is the other uncomfortable truth. AI gets you to eighty percent fast. Impressively fast. But that last twenty percent is all judgment. Integration, subtle bugs, performance edge cases, architectural fit. And if you check out mentally during the first eighty percent, that final twenty becomes an insurmountable wall. You cannot review what you do not understand.

Can you spec everything upfront and let agents run fully autonomously? No. Not yet, and maybe not ever for complex systems. You iterate. You stay in the loop. And you progress deliberately through three phases.

First, assistance. Agents support discrete tasks like code completion and test generation.

Second, augmentation. Agents manage multi-step workflows within a domain.

Third, autonomy. Agents operate across domains with human oversight at strategic control points.

Do not rush through these phases. Each one requires its own governance, its own trust-building, and its own lessons learned.



You have probably heard people claim that AI does not work. That it produces garbage. That it should not be used for serious engineering. And honestly, if all you do is type "build me a REST API" with zero context, the output will be garbage. But the people getting great results are doing something very different. They are feeding AI their architecture docs, coding conventions, security requirements, runbooks, everything the agent needs to understand how things are done in their specific codebase. The difference is not the model. It is the context you give it.


That is what context engineering is. Persistent context files. Architecture docs. Coding conventions. Rules files. Security policies. Without this, AI is a day contractor who shows up knowing absolutely nothing about your codebase, your patterns, or your constraints. It will produce code that works in isolation and breaks everything around it.


Most agent failures are not model failures. They are context failures. The model is capable enough. You just did not give it what it needed to do the job well. And there is a balance here. You need to provide what is relevant for the task at hand, not dump your entire knowledge base into the prompt and hope for the best. Too little context and the output misses the mark. Too much and the context explodes and the model loses focus. Scoping context to what matters for a given task is itself a skill.

This is also where tools like RAG, semantic search, and graph databases come into the picture. Instead of manually curating what goes into every prompt, RAG automatically retrieves relevant docs and code. Semantic search finds conceptually related content even when the naming does not match. Graph databases map relationships between components so the agent understands how things connect. These are the tools that make context engineering systematic rather than manual, and they are what let it scale beyond a single developer's memory.

This is not a one-time setup either. You have to refresh it continuously as your system evolves.

## Restructuring Teams for AI

But context engineering is only one piece of the puzzle. You can give AI perfect context, and it still will not save you if everything around it is stuck in the past. Your systems, your processes, your culture, all of it needs to change.


Think about what happened with Kubernetes. In the early days, companies did lift-and-shift. They took their legacy applications, shoved them into containers, deployed them on Kubernetes, and expected magic. The benefits were negligible. Sometimes even negative. They bolted new technology onto old architectures and wondered why nothing improved.

The same thing is happening with AI right now. Companies are adopting AI tools without modernizing anything else. Their systems do not expose proper APIs, so agents have nothing to work with. Their processes still require two-week PR review cycles, so it does not matter that code generation is ten times faster. Their culture still treats AI as a toy instead of a core part of how work gets done.



Gartner predicts over forty percent of agentic AI projects will be canceled by the end of 2027. Not because AI does not work, but because legacy systems cannot support it. Seventy percent of developers report hitting integration walls due to architectural mismatch.


The chain is only as strong as its weakest link. If everything is improved but one bottleneck remains, whether it is a system that has no API, a process that requires manual sign-offs from three managers, or a culture that refuses to let engineers make decisions, you do not get the real benefits. And ironically, AI itself can help you modernize. It can help you refactor legacy code, generate API layers, automate manual processes. But you have to commit to the transformation. Bolting AI onto the status quo is the new lift-and-shift, and it will end the same way.

So if everything needs to change, let's talk about how you actually organize the company.


Flatten ruthlessly. Kill management layers that exist only to coordinate other managers. The unit of delivery is a product pod of three to five people owning an area end-to-end. Not a department. Not a feature team. A small, autonomous group that can make decisions and ship without waiting for approval from six layers above them.


Linear is a good example. They run with one Head of Product for the entire company. Not a product department. One person. Teams of two to four form around projects, a designer plus a couple of engineers, then disband and reform. No permanent feature teams except for mobile and infrastructure. No daily standups, no sprint planning in the traditional sense. Everything optimizes for focus and output.

The scaling principle is critical. **Never make pods bigger.** When a product area grows too complex, split it into two pods. Duplicate horizontally, never vertically. And pod leads are engineers, not managers. The person shepherding delivery should be the one who understands the system best, not someone whose primary skill is scheduling meetings.

So what does one of these pods actually look like inside?

The dedicated core is one senior engineer or architect plus one or two AI-augmented engineers. That is it. That is your delivery engine. Then you have roles shared across two or three pods: a designer, a quality engineer, a Code Architect who owns technical standards. A product lead, ideally dedicated, but in practice this can be an engineer with strong product sense. You do not necessarily need a traditional product manager in every pod.

What the pod owns is a user-facing product area end-to-end. All of it. Frontend, backend, database, infrastructure, deployment, monitoring. Not a technical slice like "the API layer" or "the database team." A vertical slice of the product that a real user touches. The team that builds a feature is the same team that deploys it and gets paged when it breaks at three in the morning.

The one exception is the platform team. That is the only team that should be permanent and specialized. They own the internal developer platform, CI/CD, dev environments, observability, and increasingly, the AI tooling layer.

This is also where your deep specialists live. Your DBAs, your networking experts, your security engineers. But their role is not to be gatekeepers that pods wait on for weeks. Their job is to convert their specialized knowledge into self-service capabilities. A DBA does not manually provision databases for every pod. They build a service or a golden path that lets pods do it themselves, with the DBA's best practices baked in. Same for networking, security, observability. The specialist's value shifts from doing the work to encoding their expertise into something others can consume autonomously.

And this goes beyond traditional platform services. The platform team should build shared AI tooling. Custom MCP servers that expose internal services. Specialized agents that know your deployment patterns. Guardrails that enforce your security policies automatically.

But that does not mean only the platform team builds AI tools. Pods should absolutely build their own. An engineer who creates a tool that speeds up their own workflow is exactly the kind of bottom-up innovation you want. The platform team's other job is to watch what pods build and pull the best tools into the platform so everyone benefits. The platform team encodes organizational knowledge into shared tooling. They serve all the pods, and without them, every pod reinvents the wheel.

And honestly, this is where the entire profession is heading. We are moving into an era where the primary job of an engineer is not writing code. It is making agents do the right thing. Building context files, curating knowledge bases, creating MCP servers, designing agents, refining prompts. The people who are great at this will produce ten times more than those who are still thinking of their job as "writing code that happens to use AI." The **work is shifting from producing code to producing the conditions under which AI produces the right code**. Everyone in the organization should be investing in this, not just the platform team.

Here is something that has quietly changed about tooling and language choices. In the past, switching from one language or tool to another was a massive investment. You had to learn new syntax, new idioms, new commands, new ecosystems. That friction locked teams into whatever they already knew, even when something else would have been a better fit.

That friction is collapsing. AI handles the syntax. The focus is shifting to knowing what to do and what a good outcome looks like, not whether to write it in Go or Rust, or manage infrastructure with Terraform or Crossplane, or format data in JSON or YAML. Whatever works best for the given task. That was always the rule, but it was never easy to follow. Now it is getting easier.


There are two factors that determine how well AI works with a given tool or language. The first is popularity. AI is trained on what is common, so it generates those languages better, so developers use them more, which creates more training data, which makes AI even better at them. A convenience loop. TypeScript surged sixty-six percent to become the number one language on GitHub in large part because of this cycle.

But popularity is only half the story. The more important factor is how well something is structurally designed for AI consumption. TypeScript does not just win because it is popular. It wins because static typing gives AI clear guardrails. The same principle applies everywhere. Tools that expose discoverable APIs are inherently better for AI than tools that require fiddling with config files, regardless of how popular either one is. The question is not just "what does AI know best" but "what is AI best at working with." Those are two different things, and the structural fit often matters more than the training data volume.

New languages and frameworks face a cold-start problem on both fronts. A few samples and a tutorial used to be enough for human developers, but nowhere near enough for AI models. The risk is a monoculture where we all converge on the same handful of stacks because that is what AI knows and works with best.

## Which Engineering Roles Survive AI

Now let's talk about the uncomfortable part. Who stays and who goes.


Shopify's CEO told employees that before any new hire, teams must demonstrate why AI cannot do the job. That is where this is heading for everyone. So let's be honest about it.

But first, a critical point. This is not about official titles. A "mid-level engineer" on paper might already think like an architect. A "QA engineer" might be the person who best understands system behavior end-to-end. What matters is what each individual is actually good at and how that fits the new picture, not what their badge says. Companies need to assess people on their real capabilities, not their job title.

With that in mind, here is how the landscape is shifting by capability, not by title.

**Architectural thinking: expand**. This is the biggest bottleneck now. AI cannot make cross-system tradeoffs or account for organizational constraints. If you have people who think this way, regardless of their current title, invest in them.

**Senior engineering and system design: retain and invest heavily**. People who can judge whether AI output is correct, secure, and architecturally sound are your highest-leverage asset.

**Platform engineering: grow**. More AI-generated code means more deployments, more infrastructure complexity, more need for golden paths. This is one of the few areas where you should be adding capacity.

**Product strategy: transform and keep**. Eliminate the ticket-writing and backlog-management work. AI does that now. Keep and grow the people who drive strategy, customer insight, and cross-functional alignment, wherever they currently sit in your org chart.


**Routine implementation: reduce drastically**. The tasks that juniors and mid-levels traditionally performed, boilerplate code, simple bug fixes, basic CRUD operations, are precisely what AI does best. People whose primary contribution is writing straightforward code need to level up into design, architecture, or domain expertise.

**Manual testing and QA execution: reduce drastically**. Transform what remains into quality engineering, defining testing strategies and building AI-powered test frameworks.

**Process facilitation: eliminate as a dedicated role**. Scrum masters, ceremony facilitators, status chasers. Distribute facilitation to the team itself. AI handles the documentation and reporting.

**Routine documentation: reduce drastically**. AI generates docs, API references, and changelogs well. Keep a small number of people for strategy and complex conceptual work.

**Production design work: reduce significantly at the junior level**. AI handles wireframing, basic prototyping, and design system components. People who think in systems and conduct meaningful user research become more valuable, not less.

How do you assess where each person fits? Four dimensions.

AI leverage: can this person multiply their output using AI tools?

Architectural judgment: can they catch the subtle bugs and design flaws AI introduces?

Domain knowledge: do they understand why the system is built the way it is?

Cross-functional range: can they work across multiple domains, not just one narrow specialty?



And here is the part that should scare everyone, even the seniors who are thriving right now. If we stop hiring juniors, where do the next architects come from? Where do the next senior engineers come from? The hiring cliff we are seeing now means dramatically fewer qualified technical leaders in five to ten years. The industry is cannibalizing its own future.


The answer is not to keep hiring juniors the way we used to. It is to maintain a small, deliberate apprenticeship track and redefine what junior work looks like. Junior engineers should be reviewing AI output, learning system thinking, building things twice, once with AI and once without, so they actually understand what is happening under the hood. Their ramp-up is about learning systems and domain context, not memorizing syntax that AI handles better anyway.

And there is another angle people miss. Young people who grew up with AI think about problems differently. They do not have twenty years of muscle memory telling them to write everything by hand. They are naturally inclined toward orchestration, toward describing what they want rather than typing every line. In some cases, that fresh perspective makes them a better fit than a senior engineer with two decades of experience in one thing who is fundamentally resistant to changing how they work. The senior has domain knowledge AI cannot replace, but the junior has a mindset that is native to the new paradigm. The smartest companies will find ways to combine both.

There is a question hanging over all of this. **Will we need fewer people?**


The layoffs happening right now are driven by a short-sighted question: "How many people do I need to do what I was doing before?" And yes, if that is all you are asking, the answer is fewer. But that is the wrong question.


Every major automation wave in history has expanded the total market rather than shrunk it. ATMs did not eliminate bank tellers. Banks opened more branches. Spreadsheets did not eliminate accountants. They expanded what accounting could do. When something becomes cheaper and faster to produce, demand for it increases, not decreases.

The same thing is happening with software. Right now, every company has a backlog of features they want but cannot build, bugs they know about but cannot fix, improvements they would love to make but cannot prioritize. AI does not eliminate the need for people. It lets you actually get to all that work. Short backlogs instead of ever-growing piles. Issues that get resolved instead of accumulating. Features shipped faster and more of them.

The companies that just cut headcount and keep the same roadmap will get eaten by competitors who use AI to build more. The winning move is to expand what you do, not just shrink what it costs.

If anything, AI will displace more jobs in physical and routine work, self-driving versus taxi drivers, robots stocking shelves versus retail clerks, auto-checkout versus cashiers, than in the software industry itself. We are heading toward an explosion of what software does, and that explosion needs people to guide it.

But, and this is the critical part, what those people do will change. The composition shifts even if headcount does not. Different skills, different roles, different ratios. The industry will keep needing software engineers in the broadest sense, encompassing almost everyone working in this space. Only those who adapt will stay. The job title might be the same. The job itself will be unrecognizable.

## What To Do Right Now

So what do you actually do with all of this? Here is what I would start with Monday morning.

First, assess your team. Not by job titles. By capabilities. Who can multiply their output with AI? Who has architectural judgment? Who understands the domain deeply? Who can work across multiple areas? That tells you who is in the right seat and who needs to move.

Second, pick one pod and restructure it as a pilot. Three to five people, owning a product area end-to-end. Give them proper AI tooling, context engineering, and autonomy. See what happens. Do not try to transform the entire company at once, but do not be slow about it either. Start with a pilot, learn fast, and expand aggressively. This transformation is not optional and every week you delay is a week your competitors are pulling ahead.

Third, invest in context engineering. Set up persistent context files, architecture docs, coding conventions, rules files. This is the single highest-leverage thing you can do to improve AI output quality, and most teams have not done it at all.

Fourth, start measuring what actually matters. Cycle time. Change failure rate. Revenue per engineer. These tell you whether you are getting better. And kill the metrics that do not. Story points, lines of code, number of commits. Those were always vanity metrics, but now they are even more obviously vanity metrics. When AI can generate a thousand lines of code in seconds, measuring lines of code is measuring nothing.

Nobody has this fully figured out yet. The landscape is shifting under everyone's feet. But the companies that start now, even imperfectly, will be miles ahead of those who wait for certainty. Certainty is not coming. Adapt or get left behind.
