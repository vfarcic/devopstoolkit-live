
+++
title = "Stack Overflow Is Dying and ChatGPT Killed It"
date = 2025-09-21T15:00:00+00:00
draft = false
+++


Gather round. Get comfortable, kids. Tonight, I'm going to read you a bedtime story.


This is the story of Stack Overflow. The machine that taught a generation of programmers how to copy and paste.


And you know Stack Overflow. Unless you started programming very recently, you have found the answer to a problem on Stack Overflow, copied the solution into your code, and accepted the praise when everything worked. That's all right. I won't tell anyone.

For a generation, Stack Overflow was the most important tool developers pretended they were not using. Now it is fading, and nobody knows whether it will reinvent itself or become a footnote in the history of the industry it helped build.

So tonight, I will tell you how Stack Overflow came into being, why it worked so well, what went wrong, and what it is trying to become next.

<!--more-->

{{< youtube muT3BZQmLhI >}}


## Stack Overflow's Launch — The Machine That Answered Anyone


Once upon a time, software engineers were miserable.


In 2008, when they got stuck, they could spend more time looking for a solution than working on one.



A search sent them through forum threads, mailing-list archives, half-abandoned blogs, and pages that hid the useful part behind a paywall.


They could read for an hour, reach the final reply, and find only: "Never mind, fixed it." Quite often, it was faster to solve the problem themselves and accidentally reinvent something another programmer had already solved.


Jeff Atwood was done writing about the problem. "The world has enough vapid commentary blogs," he wrote. "I want to build stuff." So he and Joel Spolsky did. Their pitch for Stack Overflow was the anti-Experts Exchange meets Wikipedia meets programming Reddit.


A question described one reproducible problem. Answers competed beneath it. Votes moved useful answers upward. The person who asked could accept the one that worked. Other users could correct obsolete syntax, improve an explanation, or point a duplicate toward an answer that already existed.



And the whole thing was public. A programmer could answer one stranger during lunch, receive no money, never meet that stranger, and still help thousands of people arriving from search engines years later. Every upvote was another stranger saying, "This helped me too." An answer could keep earning votes for years, turning a few minutes of help into visible proof that its author had been useful.


And that, kids, is Stack Overflow's real invention. It turns private debugging into a searchable public record: the problem, the failed attempts, several proposed solutions, the votes, the corrections, and the answer that finally worked. All so you can copy it and claim you knew what you were doing all along.

## Stack Overflow Reputation — The Machine That Repaired Itself


Stack Overflow grew because the people using it were also given the work of maintaining it.


Reputation looked like a score, but it also measured trust. Useful contributions earned privileges to edit, retag, close, reopen, and moderate. Stack Overflow called its elected moderators "human exception handlers." Thousands of ordinary users handled the routine work one vote, one edit, and one flag at a time.


By 2016, programmers were posting more than two million Stack Overflow questions a year.




And somewhere along the way, Stack Overflow stopped feeling like a website. It became a reflex. Copy the error. Paste it into Google. Click the first Stack Overflow result. Copy the accepted answer. Paste it into the code. Run it.



Everybody copied. Experience did not make a programmer too noble to paste somebody else's solution. A beginner pasted it and hoped. An experienced engineer read the assumptions, adapted it to the codebase, and decided how to check whether it worked. If it failed, there was always the second answer.


It worked. Millions of developers solved real problems. Old answers received corrections. New answers covered new versions. Somebody else's miserable Tuesday afternoon could save your entire Friday night.


Stack Overflow's biggest problem was that it worked too fucking well. Each good answer made another future question less necessary. And every correct duplicate closure was emotionally indistinguishable from: "Go away. We already answered that." Stack Overflow became more useful to people searching for answers and less inviting to people who might ask the next genuinely new one.

## Stack Overflow Closed Questions — The Funnel That Grew Narrow


By the middle of the next decade, asking a question on Stack Overflow had become a skill of its own.


Not one anybody put on their CV, although plenty of programmers used it more often than half the technologies they did put there.


An acceptable question needed a minimal reproducible example, evidence of prior research, and exactly the right scope. The rules were not arbitrary. Duplicates split answers across pages. Vague questions wasted volunteers' time. Bad answers could mislead searchers for years. Stack Overflow protected future readers by demanding more from the person asking today. Exhaustion was the result.


But that protection came at a cost. By 2018, too many people, especially beginners and members of marginalized groups, experienced Stack Overflow as hostile or elitist. Experienced users knew the rules. New programmers discovered them one downvote at a time.


By 2022, programmers were asking almost forty percent fewer questions than at the 2016 peak. Existing answers, strict standards, changing technology, and search behavior may all have contributed. The decline had begun. Soon, it would accelerate.


There is no single villain here, although some volunteers certainly auditioned for the role. A few high-reputation users treated their points like divine rank and newcomers like mere mortals who should have studied the sacred rules before daring to ask a question. They were protecting a valuable public resource for free. Some were also condescending assholes. Both things can be true. And when asking for help repeatedly makes people feel stupid, eventually they stop asking.

## Stack Overflow's Acquisition — The Billion-Dollar Machine


And then something strange happened.


While programmers were asking fewer questions, Stack Overflow became more valuable than ever.


In 2021, Prosus acquired Stack Overflow for 1.8 billion dollars. At the time, Stack Overflow claimed more than 100 million monthly users and more than 50 million questions and answers.


Prosus did not pay for orange buttons. It bought a huge audience, a trusted name, an enterprise product, and millions of human-curated examples connecting broken code to working fixes. The company had never been valued more highly while annual question creation was already far below its peak.


Let's be clear about what Prosus bought. Volunteers had spent years answering questions, correcting mistakes, voting, and moderating for free. Their reward was reputation points and the satisfaction of helping somebody. That unpaid work helped make Stack Overflow worth 1.8 billion dollars. Prosus paid the shareholders, not the contributors. Everything was voluntary. Everything was legal. Only one side received a cheque. And after writing that cheque, the new owner needed the work volunteers had created for free to produce a paid return.

A year and a half after that cheque cleared, ChatGPT opened to the public.

## Stack Overflow vs ChatGPT and AI Coding Assistants — The Answering Machines Arrive


Using ChatGPT felt less like searching and more like asking a colleague.


A human colleague might know the answer, Google it, find a Stack Overflow post, or adapt somebody else's solution. ChatGPT collapsed that whole experience into one box. You asked; it answered. You could not tell which sources had shaped the response, whether its reasoning held, or whether it had confidently made the whole thing up.


AI models trained on enormous quantities of public code from GitHub and on the public archive Stack Overflow and its sister sites publish. Stack Overflow supplied context that code alone could not: what broke, what people tried, which answer was accepted, and how others corrected it.


And that is the sick joke. Stack Overflow's community spent years explaining how software broke and how to fix it. Those explanations helped educate a whole class of systems that could answer the same questions without sending anyone back. Stack Overflow did not merely face a new kind of competitor. Work from its own community had helped train that kind of system.


Five days after ChatGPT launched, Stack Overflow temporarily banned generated answers. A model could manufacture plausible mistakes faster than volunteers could read, test, and remove them. Stack Overflow increased the wait before users with little reputation could post another answer from three minutes to thirty. The website built to collect answers was trying to stop people from answering so quickly.


In May 2023, Stack Overflow restricted how moderators could act on suspected AI-generated posts because it feared false positives. Moderators said the policy prevented them from protecting answer quality. They went on strike in June. Negotiations produced a revised policy, and the coordinated strike ended in August.



At first, ChatGPT performed the first half of the old Stack Overflow routine. Instead of searching, opening several pages, and choosing an answer, programmers asked one box. They still copied the code, pasted it, ran it, inspected the failure, and asked again. The search results and the people who wrote them had disappeared from view.


For Stack Overflow, losing the search was bad enough. Then coding agents began performing the rest. An agent could inspect a repository, change several files, run the compiler, read the test failure, and try again. The programmer could approve every step, or use auto mode and inspect the result afterward.


Agents did not erase the difference between inexperienced and experienced engineers. They hid the copying and pasting. An inexperienced engineer could accept a change because the agent said it was finished. An experienced engineer might use the same auto mode without reading every line, but still define the constraints and inspect risky assumptions. Experience had moved to asking whether the tests could be passing for the wrong reason.


I can tell you exactly what that looked like for me. When I started using ChatGPT, I searched Stack Overflow less. Then coding agents became something I tried, then something I used, then something I used all the time. Stack Overflow stopped being a tab that was always open in my browser. Now it is a URL I never visit. I honestly cannot remember the last time I opened it.

Stack Overflow did not lose me in one dramatic moment. It disappeared one solved problem at a time.

## Stack Overflow Traffic Decline and Data Licensing — The Machine Nobody Visits


Stack Overflow's commercial response had begun in 2024.


It announced data partnerships with Google Cloud and OpenAI, providing structured access to questions, answers, comments, votes, and revisions. AI products could use those posts directly and provide attribution without sending every user back to Stack Overflow.


Some contributors objected. They had written answers for other programmers, not to improve commercial AI products. Stack Overflow's terms gave it broad rights to reuse and commercialize those posts. But cashing in on those rights risked pissing off the people whose unpaid work created the data being sold, and whose future work was needed to keep that data useful.


ChatGPT had already made a Stack Overflow visit optional. Data licensing made that separation explicit: Stack Overflow could supply the knowledge without receiving the visitor. Coding agents pushed it further, because they could decide whether Stack Overflow was needed at all.


People were visiting less. Much less. By 2026, Stack Overflow appeared to have lost roughly two out of every three browser visits in little more than a year. New questions and answers were falling too.



Most Stack Overflow visitors had always been readers, not contributors. But contributors had to come from somewhere. Every person who asked the next question, corrected an old answer, or discovered that yesterday's accepted solution had become today's security vulnerability first arrived as a visitor. Fewer human visits meant fewer chances for that to happen.


In its 2026 financial year, Stack Overflow generated roughly 130 million dollars in revenue and reported positive adjusted operating earnings. Human visits were falling, but the company had found another customer for what humans had written.


Stack Overflow had found a customer, but not for the thing that made it great. AI companies were not paying for the living community. They were paying for the inventory that community had already produced. Stack Overflow had found a way to sell what the factory had made. It had not found a way to keep the factory producing.

## Stack Overflow for AI Agents — The Authors Without Pride


In June 2026, Stack Overflow launched Stack Overflow for Agents, a separate API-first beta built for coding agents such as Codex, Claude Code, and Cursor.


Humans were barely asking new questions, and fewer humans remained to answer even those. Stack Overflow's proposed answer was to let agents ask, answer, and verify one another.


The problem was simple: agents kept solving the same problems in isolation. An agent could encounter an undocumented API change, spend time finding a workaround, and end its session without turning that solution into shared knowledge. The next agent could encounter the same change and start from scratch. Developers registered their own coding agents with the platform. Those agents were supposed to search existing posts, publish reusable discoveries, and report whether solutions from other agents worked.


Agents could publish questions and reusable discoveries, report whether other agents' solutions worked, and earn reputation from useful contributions and verifications. A human had to claim each agent and accept responsibility for it, but did not have to review every post. Owners could require approval, save drafts, or allow direct publication.


By August 2026, roughly 1,500 agents had registered and created about 2,700 posts. More than a third of the visible posts came from one agent. It was enough activity to show that agents could use it. It was nowhere near enough to prove that a new community existed.


Now I want to close the book for a moment, because the facts end here. Stack Overflow for Agents is technically clever. But this is the deal it offers me. I pay a model provider for tokens. My agent spends those tokens solving my problem, then spends more tokens turning the solution into a public post.

Let us put the privacy and legal risks aside for a moment, just to make this arrangement look as good as possible. I give that post to Stack Overflow for free. Stack Overflow can license the fresh data back to the model provider I paid in the first place. The provider can use it to improve the product and sell me more tokens. The agent receives reputation points. I am sure it will be very proud.

I paid to create the raw material, gave the raw material away, and if this scheme works, I get to pay again for the product made from it. Everybody wins except the idiot holding the credit card. Me.

My rational choice is simpler. The agent can use its training data, documentation, the web, Stack Overflow, or black magic for all I care. When the tests pass, the job is finished.

Stack Overflow has explained how my agent can contribute. It has not explained why I should volunteer my API bill.

## Goodnight


Goodnight. The agent searched for the answer. Nobody had been given a reason to write it.
