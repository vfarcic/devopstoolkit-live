
+++
title = "How I Hooked AI Video Generation Into My Dev Workflow (with Higgsfield)"
date = 2026-05-11T14:00:00+00:00
draft = false
+++

Real productivity out of an AI agent starts with where it lives. Agents that run inside your terminal, your IDE, or your desktop beat agents that sit behind a web UI. [Codex](https://github.com/openai/codex), [Cursor](https://cursor.com), [Claude Desktop](https://claude.com). The surface around the model is what decides how much work you actually get out of it.


And it's not just code. Calendars, email, web research, summarizing long PDFs, drafting writing. All of it routinely happens inside the same agent session, through MCP connectors that wire the agent into Gmail, Google Calendar, Notion, Slack, and whatever else you've plugged in.

<!--more-->

{{< youtube H8vcbLTBhco >}}

The agent in your terminal is already a general-purpose tool. MCPs hook it into outside services. Skills turn it into a workflow engine you can shape. Code is just one of its workloads. Far from the only one.


And the bigger productivity win isn't capability. It's **consolidation**. Think about how a normal day looks. An editor open for code. A separate writing tool. A media service in another tab. A scheduling app. A browser with a dozen more tabs for everything else. All of that splits your attention. All of that splits your context. With an agent sitting in your terminal, all of that work happens in one interface, in one conversation, with one shared context.

There are three concrete reasons consolidation actually works, instead of just sounding nice on a slide. First, the files the agent produces land next to the work you're already doing. Not in some vendor sandbox, not in a downloads folder you'll never open again. They land in the project you have open. Second, the agent has your local context. It knows your git state, the project you're in, the environment you're running in, the files you've been editing. Third, it composes with everything else already on your machine. Other tools, other scripts, other skills you've layered on top. And those three reasons hold for every one of those workloads I just mentioned. Not just code.

And the piece that makes all of this possible is MCP. 

MCPs flip the direction of integration. The old model was that you exported your work to a vendor's website. You opened their app, did the thing inside their tab, copied the result back. The new model is the opposite. You bring the vendor's capability into your loop. The agent calls them. Your files, your context, your conversation, all stay where they are.

And the next workload to slot into that loop, the one this whole video is about, is media.


Which brings me to the part I've been holding back. Some of the clips you've been watching haven't even existed before I started writing this script. Even the one you're watching right now.


## AI Video Generation in Claude Code


Those few clips that broke up the talking head: the slate at the start, the scattered desk in the middle, the shot of me from behind a moment ago. None of them existed before I started writing this script. I generated them from inside Claude Code, talking to [Higgsfield](https://higgsfield.ai) through their MCP server, and they landed right here, in this project directory.

I've been using AI to generate media for a while. Image, video, audio, you name it. Every single one of those tools assumed I'd live inside their browser tab. Generate something, download it, drag it into my project, copy-paste prompts and outputs back and forth, then bounce back to the tab and do it again. Multiply that by every iteration of every clip and the friction adds up fast.

The thing that changed isn't the model quality. It's that the model is now reachable from where I already work, through MCP, from inside the same agent that already knows about my files, my project, my conversation.

I'll show you how the setup works, what it feels like to use, and where it falls short. Because it does.

Setup has two parts. An MCP connector, which is a one-time, thirty-second drop-in. And a custom skill, which is where the actual workflow lives.

Connector first. It's six lines.

```sh
cat .mcp.json
```

The output is as follows.

```json
{
  "mcpServers": {
    "higgsfield": {
      "type": "http",
      "url": "https://mcp.higgsfield.ai/mcp"
    }
  }
}
```


A server name, `type: http`, and a URL. That's the entire connector. The first time anything calls a Higgsfield tool, Claude Code prompts for an OAuth login. One time, then you forget about it. Same shape, same endpoint plugs into [Claude Desktop](https://claude.com), [OpenCode](https://opencode.ai), or whatever MCP-aware agent you happen to use. Different agents, more or less the same result.

The connector is six lines and you're done. The skill is where the actual workflow lives. Let's take a look.

```sh
cat .claude/skills/manuscript-broll-suggest/SKILL.md
```

The output is as follows (trimmed for brevity).

~~~md
---
name: "manuscript-broll-suggest"
description: "Scan a manuscript for overlay-free narration segments and suggest cinematic b-roll concepts to generate via Higgsfield AI; on selection, generate the clip into ./tmp and open it in QuickTime, then offer to refine or pick another."
---

# Suggest and Generate B-Roll for Manuscript via Higgsfield

Analyze a manuscript, find segments that have no other visual overlay, propose movie-like b-roll concepts for them, and on user selection generate the clip with Higgsfield AI, save it under `./tmp/`, and open it in QuickTime. Then loop: refine the current clip or pick another suggestion.

## Input

- **Manuscript path**: $1

## Workflow Overview

1. Resolve manuscript path (ask if missing).
2. Detect overlay-free narration segments.
3. Suggest 3–8 b-roll concepts (each tied to a specific segment).
4. Wait for the user to pick one (or accept all the suggestions in sequence).
5. Generate the clip via Higgsfield, poll until done.
6. Save the result to `./tmp/<manuscript-stem>-broll-<slug>.mp4` and open in QuickTime.
7. Ask: pick another suggestion, or refine the just-generated one (re-prompt and regenerate).

...
(relatively long set of instructions)
~~~

This file is long. Most of it is implementation detail — prompt rules, the headshot upload pipeline, how the skill marks kept clips in the manuscript. Things the agent needs, not me. Skip all that. The part that matters is `## Workflow Overview`. Seven steps. The skill scans my manuscript for paragraphs that are pure narration with no other visual overlay planned, and proposes between three and eight cinematic b-roll concepts for those segments. I pick one, or tell it to walk through all of them. The skill calls Higgsfield, waits for the clip to render, saves it to `./tmp/`, opens it in QuickTime. Then it asks: keep, throw away, or refine. Refine means regenerate with my feedback. Then it loops to the next suggestion. That's the entire workflow.

Time to actually run it. I open Claude Code in this exact repo, on this exact manuscript file.

```sh
claude
```

I invoke the skill, point it at this manuscript file, and ask for at least one clip where I'm the subject.

[user]
```text
/manuscript-broll-suggest , @manuscript/ai/higgsfield.md, only Section 01, include me in at least one of them.
```

[agent]
```text
 I'll analyze Section 01 (lines 13–38) of manuscript/ai/higgsfield.md for overlay-free
   narration and propose b-roll concepts.

  Section 01 scan:
  - Lines 19, 26 — adjacent to TODO: Logo: blocks above (within 2 lines) →
  overlay-occupied
  - Lines 28, 30, 32, 34, 36, 38 — clean narration, no overlay markers within 2 lines
  - No existing TODO: clip: lines in this section → no adjacency warnings

  Here are 5 candidates. One is tagged narrator-as-subject as you asked.

  ---
  Suggestion 1 — Holding back the reveal  [👤 you as subject]

  Section: Section 01
  Narration excerpt: "Which brings me to the part I've been holding back. All those
  clips you've been watching. None of them existed before I started writing this
  script."
  Why this segment: The reveal beat of the whole video — autobiographical,
  conspiratorial, first-person.
  Concept (8s): A man sits at a writing desk in a low-lit home office, late evening,
  lit by a desk lamp and the soft glow of a laptop. He leans back from the keyboard,
  hands loosely clasped, looking past the camera with a faint, knowing half-smile — the
   look of someone about to let the audience in on something. For the first 3 seconds
  he stays still, fingers tapping once on the desk. At 4s, he tilts his gaze toward the
   lens. By 7s, he gives the smallest nod and looks back down at the screen. Handheld,
  slow drift in. Warm tungsten lamp, soft window spill behind, shallow depth of field.
  Quiet, intimate, slightly mischievous.

  ---
  Suggestion 2 — Scattered tabs, divided attention

  Section: Section 01
  Narration excerpt: "An editor open for code. A separate writing tool. A media service
   in another tab. A scheduling app. A browser with a dozen more tabs for everything
  else."
  Why this segment: Vivid concrete list — fragmented digital life, instantly
  visualizable as a person drowning in app windows.
  Concept (8s): Wide shot of a cluttered home office at night. A woman sits at a
  triple-monitor setup, every screen full of overlapping windows, sticky notes peeling
  off the bezels, a half-empty coffee mug, a phone buzzing face-up beside her. The
  camera dollies in slowly across the desk. For the first 3 seconds her eyes flick
  screen to screen, fingers hovering over the trackpad. At 4s she rubs her temples and
  exhales. By 7s she leans back, motionless, the chaos around her unchanged. Practical
  lamp, blue screen glow on her face, motivated lighting, warm/cool contrast.
  Overwhelmed, still, almost defeated.
  ...
  ---
  Which suggestion should I generate? (number, or all to walk through them)

✻ Worked for 1m 44s
```

Five cinematic concepts, each tied to a specific paragraph of narration. One of them, the desk shot, is tagged "you as subject". That tag triggers the headshot pipeline I built into the skill: before it generates that clip, it'll need a reference image of me to feed Higgsfield.

Rather than picking one suggestion at a time, I tell it to walk through all five in sequence.

[user]
```text
All
```

The output is as follows (trimmed for brevity).

[agent]
```text
⏺ Starting the walkthrough with Suggestion 1, which requires a headshot UUID since it's
   tagged [👤 you as subject]. Let me check the headshot config and balance.
  ...
⏺ screenshot-03 is the cleanest front-facing neutral reference — using it for
  Suggestion 1. Setting up task tracking, then starting the upload + generation flow.

  Called higgsfield (ctrl+o to expand)

⏺ Uploading screenshot-03.jpg as the headshot reference for Suggestion 1.
...
⏺ Suggestion 1 — Holding back the reveal is open in QuickTime.
...
  Keep this clip? (yes / no / refine)
...
❯ Keep
...
⏺ Suggestion 2 — Scattered tabs, divided attention is open in QuickTime.
...
❯ She should be facing monitors. Instead, they are behind her. Show could be from
  behind her.
...
⏺ Suggestion 2 v2 — Scattered tabs (over-the-shoulder) is open in QuickTime. v1
  deleted.

❯ Keep
...
⏺ Suggestion 3 — Files land where the work is is open in QuickTime.
...
  Keep this clip? (yes / no / refine)

❯ No
...
⏺ Suggestion 4 — Direction of integration flips is open in QuickTime.
...
  Keep this clip? (yes / no / refine)

❯ No
...
⏺ Suggestion 5 — Pivot to media (clapperboard) is open in QuickTime.
...
  Keep this clip? (yes / no / refine)

❯ Clap is wrong. It should be one used in movies.
...
⏺ Suggestion 5 v2 — Pivot to media (film slate) is open in QuickTime. v1 deleted.
...
❯ Clap is empty and wooden color. Make it exactly like in movies. You can use "DevOps
  & AI Toolkit" as the name of the movie and "Viktor Farcic" as director as well as
  the actor and cameraman.
...
⏺ All 5 suggestions walked through.
...
```

For any clip with me as the subject, the skill looks in `./tmp/` for screenshot files, picks the cleanest front-facing one, uploads it to Higgsfield, and caches the resulting UUID locally. The cache means I pay for the upload once. Subsequent you-as-subject clips reuse the same identity reference.

Each generation runs between 60 and 180 seconds. The agent polls in the background, so I can keep working while it waits. When the clip is ready, it lands in `./tmp/`, the same directory as the manuscript that asked for it. QuickTime opens it automatically. This is the moment the whole video is about. The clip isn't sitting in a browser tab. It's a file in the project, alongside everything else.

After watching each clip, I have three options: keep, throw it away, or refine.

Throwing it away just deletes the file. Refining is the interesting one, and it's where the honest friction shows up. I tell the agent in plain English what's wrong with the current take. It rewrites the prompt, generates a new version, deletes the old one once the new one lands.

Two of the three keepers in this run needed refines. Suggestion 2, the cluttered desk shot, first came back with the actor facing away from her monitors. I asked for over-the-shoulder framing instead. The second take is what you saw earlier. Suggestion 5, the movie slate at the start of the video, took two refines. First take had a generic clapper. Second take got the shape right but produced a blank, wooden-looking slate, like an art-class prop. The third take, after I specified the movie title and the names on the slate, is what you saw at the start of the video.

Five concepts in. Three keepers out. Two discards. Eight generations total. First-take output is not trustworthy, and the iteration cost is real.


Cost. I'm on Higgsfield's Ultra annual plan. $125 a month for 6,000 credits. A Seedance 2.0 clip at 1080p, eight seconds, costs 72 credits, or **roughly $1.50**. The whole walkthrough, including the discards and the regens, landed around $12. Buying credits à la carte instead of subscribing pushes that closer to $24. Either number is a fraction of stock licensing, hiring a videographer, or filming it myself.

Time. About 30 minutes for 8 generations. Not nothing. But I wasn't sitting and waiting. I'd kick off a generation, switch to another window, check back when it was ready.

That's the workflow, friction and all. I'm an engineer. I live in a terminal. But the workflow itself isn't engineer-specific.

## Why MCP Is Eating Single-Purpose AI Apps

Claude Code is what I happen to use, but it doesn't matter which proper agent you pick. TUI like Claude Code, IDE like Cursor, desktop app like Claude Desktop, take your pick. They get marketed as developer tools because that's where they were built first. But underneath, they're general-purpose. They can work with almost anything you point them at.


The same agent I use to write code is a perfectly viable creative interface. For a video producer. For a journalist. For a researcher generating diagrams or summaries. For a marketer drafting campaign visuals. For anyone whose day involves generating media.

And it's not a stretch. Higgsfield's actual customer base is mostly creators and studios. That's who's paying for video generation in the first place. What MCP does is let that audience finally meet the agent audience. The same protocol I use to wire Higgsfield into Claude Code is the protocol a video studio would use to wire Higgsfield into Claude Desktop.

Honest caveat before I get to the bigger argument. There's friction I haven't beaten. Visual continuity across multiple clips — making them look like they came from the same project, with the same character or the same color grade or the same lighting style — still needs its own multi-step setup. Higgsfield calls it Soul Character training. I haven't done it. The workflow I just walked you through is great per-clip. It's not great at making twenty clips look like they belong together. That gap is real.

Now zoom out one more level. This isn't just Higgsfield's bet. It's where the whole AI industry is heading.

Specialized AI services — image, video, audio, search, design, take your pick — are realizing they can't compete with generic agents like Claude, Gemini, or whatever comes next. The generic agents have everything: the writing, the reasoning, the conversation, the local files, the tools. What's a single-purpose AI service supposed to do, build its own version of all of that? That's a losing battle. So they're flipping the strategy. Instead of trying to replicate what the agents do, they're enabling the agents to use them.

Higgsfield's model-selection feature is a small concrete example of what that looks like. Different shots want different models — Seedance for identity-preserving b-roll, Kling for cinematic motion, others for marketing-style ads. I don't have to know any of that. I never have to learn eight different APIs. The skill calls into Higgsfield, Higgsfield picks the right model. That's the specialist doing its job: being the best at media, including the parts I don't want to think about.

Focus on specialized intelligence. Delegate everything generic to the agent. That's Higgsfield's bet, and every specialist worth using will make the same bet. Trying to build a chat interface that competes with Claude or ChatGPT is not a fight worth picking.

So the new shape of AI products is this. The agent is the operating system. MCP is how specialized services plug into it.

The recipe is straightforward. A proper agent — terminal, IDE, or desktop, doesn't matter which — combined with your local files (project, manuscript, git state), remote capabilities plugged in via MCP, and skills that turn ad-hoc prompts into repeatable workflows. In this video specifically: the local file was a manuscript, the remote capability was Higgsfield, the skill was `manuscript-broll-suggest`, the agent was Claude Code, and the orchestrator was me. That's the pattern.


Generic browser chats like [Claude.ai](https://claude.ai) and [ChatGPT](https://chatgpt.com) can technically do most of this. They support MCPs, you can give them tools. They just suck at it. No clean view of your files, no composition with the rest of your machine, and the loop gets clumsy fast. Single-purpose web tools — Higgsfield's own UI, every other media service with a chat panel — suck even more. Locked to one specialty, no escape hatch when the work needs to combine with anything else.

One last thought. 

People always ask whether this stuff is going to replace creative professionals. Maybe not today. But givfe it a bit of time and yeah, it will. Same way it's already replacing parts of what software developers used to do. Some of us — at least some of us — are already specialized orchestrators. Managers, really. We don't write the code anymore; we manage the AI agent that does. And that shape is coming for every profession where the work involves generating something. Writers, designers, journalists, marketers. They're going to be orchestrators too.


So the real question isn't whether the change happens. The real question is which side of it you end up on. You're either an orchestrator — sitting next to your files, running a proper agent, plugging in the specialized capabilities you need — or you're whoever those orchestrators replace.
