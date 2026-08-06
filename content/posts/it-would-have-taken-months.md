---
title: "The AI-Built App Worked. Then I Opened the Repo."
date: 2026-08-06T08:40:43-06:00
draft: false
slug: "it-would-have-taken-months"
description: "A year ago, I would have sent it back. This time, closing the gap between useful and releasable took an afternoon."
tags: ["ai-assisted-development", "internal-tools", "platform-engineering", "software-delivery"]
ShowToc: true
hideArticleFooter: true
---

The first app I picked up came from a builder in another department.

He had distinguished himself over the past year by making tools that got his
job done faster, and the department wanted his work opened up to everyone else
on the team. So it came to me to onboard.

And it was good. I want to establish that before anything else, because it is
the part that surprised me. The UI was clean. Every link went somewhere. It was
in daily use and people liked it, which---if we are being honest about internal
tooling---already puts it ahead of a fair amount of software I have shipped
with a proper pipeline behind it.

He had built it with an AI assistant, and it worked.

Then I actually opened the repo.

I am keeping the identifying details broad here. The engineering pattern is the
point.

**The database was a mix of CSV and JSON files.** Sitting in the repo.

**In terms of security, there was no security.** Plain HTTP. No OIDC. No access
gates. Possession of the URL was effectively the access control.

**It was shaped for local use, not production.** One process. A hodgepodge of
file storage. No queue. Which is exactly what it was for, until suddenly it
wasn't.

None of that is unusual. I have onboarded several of these in a short stretch,
and they fail in the same three places every single time. Not similar places---
the same three:

1. A data layer that is not a database.
2. Security that does not exist.
3. An application shaped for one user on one machine.

That sounds harsh if you read it as a review of the builder. It is not. The app
was useful because he understood the work well enough to build the right thing.
The missing pieces were the parts nobody had given him a reason to ask about.

## What I would have said a year ago

Absolutely not.

I would have sent it back with a note: this is an incomplete project. It is fine
locally. It is not fine company-wide. Come back when it has a real database and
someone has thought about auth.

And that was not gatekeeping for its own sake. It was arithmetic. Fixing those
three categories properly meant weeks of work at minimum. I had a healthy
backlog already, and the honest answer to "can we ship this?" was that somebody
would have to rebuild it.

Which really meant it would never ship, because nobody was going to fund a
rebuild of a tool that already worked.

So the old answer was: everything goes through developers. Not because business
users cannot build useful things---clearly they can; that is the whole
problem---but because the gap between *useful* and *releasable* was too
expensive to close.

That is the part that changed. Not the gap. The cost of closing it.

## The three propositions

Zooming out from the one app, here is the shape of the thing every company I
talk to is now standing in.

**1. They want subject-matter experts building tools for their own
departments.** And they should. The person closest to the work knows which edge
case actually happens on Thursdays. They know two of the five official statuses
have never been used. They know the field labeled "optional" is the one everyone
searches on.

That knowledge does not survive a requirements document. When they build the
tool themselves, it goes in directly.

**2. Those experts are not coders, so they rely on AI to do the coding.** This
is fine, and it works. It is also the only reason proposition one is possible at
all.

**3. With the context available to it, the assistant will not produce the
development team's standard.**

That third one is the whole series, so let me slow down on it.

{{< diagram src="/diagrams/missing-context-gap.svg" mobile="/diagrams/missing-context-gap-mobile.svg" alt="A subject-matter expert supplies domain context to an AI assistant, which can produce a useful tool. Production standards remain outside that shared context, leaving a gap between useful and releasable." caption="The assistant can only build with the context in the room. The missing layer is organizational, not generative." >}}

The operative words are *context available to it*. This is not a claim that the
models are bad. Ask directly and any current model will tell you that secrets
belong in a vault, that a CSV is a poor database under concurrent writes, and
that you want TLS. It knows all of it.

But nobody asked.

Nobody asked because the person typing did not know it was a question. You
cannot request expertise you do not know exists. Meanwhile, the assistant is
being helpful in exactly the way it was asked to be helpful: someone said "add
a place to store the records," and a CSV stores records.

So this is not a competence failure on anyone's part. Not the builder's. Not the
model's. It is a **missing-context failure**.

That reframing is load-bearing for everything that follows, because the fix for
missing context is not better models or more discipline. It is supplying the
context.

Which turns out to be a thing you can actually build.

## Why I am the one telling you this

I got handed the onboarding. Several apps in a short stretch, from different
parts of the business, all with the same three problems in different
arrangements.

That is the whole credential. I am not theorizing about this. I am just the
person the tools land on.

## The afternoon

Here is the thing I did not expect.

That first app---the one I would have bounced a year ago without a second
thought---took an afternoon.

Here is what changed:

1. **The files became a real database**, provisioned and managed through the
   standard pipeline instead of living beside the application.
2. **The code got rails.** Conventions, formatting rules, and documentation
   expectations became part of the project instead of knowledge the builder
   was expected to somehow infer.
3. **Identity and secrets moved onto managed infrastructure.** Organizational
   sign-in came through OIDC. Access stopped being possession of a URL, and
   secrets moved into a managed vault instead of belonging to the application.
4. **History moved into durable object storage**, so historical data had a home
   designed to retain it instead of accumulating inside the running app.
5. **Concurrency became an explicit design concern.** A queue went in, and the
   runtime stopped assuming that one person would finish one thing at a time.
6. **The application became deployable.** I containerized it, and GitHub Actions
   became the path from a reviewed change to a running release.

Security was the critical path by far. Not because adding OIDC or wiring a
vault was mechanically difficult, but because the important questions were not
code questions: what production access should this application have, what did
it actually need, and where should the boundary sit? Those discussions---plus
the ordinary friction of getting a new deployment working---were the part that
could not be generated away.

And then the app was releasable. Same app. Same UI. Same logic. Same domain
knowledge that I could not have supplied. Different foundation.

## Why it was an afternoon---which is not the obvious reason

I want to be careful here, because "AI did it; AI is fast" is about a third of
the answer. The other two-thirds are the interesting part.

**The labor got cheap.** True, and this is the part everybody assumes I mean.
Mechanical translation is exactly what these tools are best at: swap the data
layer, generate the deployment scaffolding, rewrite forty call sites so they go
through a repository instead of reading a file. That work used to be tedious
and slow. Now it is tedious and fast.

**There was somewhere to move it to.** This is the part people skip, and it
matters more than the first one. I was not inventing a security posture for this
app. I was not deciding what database we use, how we do identity, or how things
get deployed. Those decisions already existed. My afternoon consisted mostly
of connecting his app to answers we had already committed to.

Take that away---make me design the destination per app---and it is not an
afternoon. It is a design project, and we are back to weeks.

**I already knew what "right" looked like.** Ten years of .NET and Azure is what
let me open that repo and immediately know the things to fix and the order to
fix them in. Hand the same AI tools to the original builder and it is *not* an
afternoon, because the list is not the hard part to execute. It is the hard part
to know exists.

Which is a slightly uncomfortable thing to notice about my own value, and also
the entire argument for writing the list down.

The others did not arrive as copies of this one. They came in different shapes,
but most shared the same basic ambition: complete one task, from start to
finish, and do it very well. The foundations varied. The pattern did not.

One of them depended on credentials whose production reach was not clear from
the application. That one did not start with a migration. It started with a
plan: establish what the credentials could reach, decide what the application
actually needed, and work out how to replace that access without casually
breaking a production workflow.

That matters because "an afternoon" is not a universal unit of effort. The
mechanical work can collapse dramatically. Uncertainty at a real boundary---
production access, consequential data, an integration nobody fully owns---does
not. Sometimes the professional move is to go faster. Sometimes it is to stop
and find out what you are holding.

## The objection I get every time

"They are internal tools. Is this not over-engineering?"

I want to take that seriously, because for a genuine scratch tool---one person,
one machine, data nobody would miss---the answer is yes. Absolutely. Leave it
alone. Most things should live and die there and never come anywhere near me.

But *internal* is a statement about the URL, not about the data.

The app I have been describing holds records the business runs on. Other tools
like it sit inside workflows with customers, money, or operational decisions on
the far end. The moment a second person depends on one, "it works and everyone
likes it" stops being a control and starts being a hope.

And the three things I actually insist on are not gold-plating by any standard I
recognize. A real database instead of files. Sign-in. The ability to run on a
server and handle more than one person at once.

That is not enterprise architecture. That is the definition of *more than one
person can use this*.

## An afternoon is not a solution

Here is where I have landed, and it is less tidy than I would like.

The onboarding works. Genuinely. The economics have flipped, and something that
was impossible is now a half-day's work. But hardening an app is a **snapshot**,
and the app does not hold still.

Because here is the division of labor I think is actually correct: the builder
keeps the app. It is his. He will have features to add, UI to fix, and bugs to
chase---and he *should* do that work, because he is the one who knows what the
tool is for. I do not want to own his roadmap, and he does not want me to.

Which means someone who still does not know the list is going to keep changing
the app. Nothing about my afternoon prevents the next feature from writing to a
CSV file or the next integration from putting a key in source.

I did not fix the app's security. I fixed the app's security *as of Tuesday*.

You cannot onboard your way out of that. Onboarding is a one-time event. The
problem is continuous.

{{< diagram src="/diagrams/from-snapshot-to-road.svg" mobile="/diagrams/from-snapshot-to-road-mobile.svg" alt="A one-time hardening event closes today's gap, but a sustainable delivery road surrounds future changes with standards while building, graduation checks, and automated gates." caption="The goal is not a perfect handoff. It is a path that keeps ordinary changes inside the guardrails." >}}

So the target is not a checklist I run. It is something more like a road: the
security posture set once and inherited by default; the standards available at
the moment someone is building instead of at review time; and a gate on every
change afterward that does not need me awake to hold the line.

The operations side should be quiet and low-maintenance for the builder. That is
the deal I am offering. Setup, security, a real database, and something that
survives multiple users at once are mine. The tool itself stays his.

That is the next post: what the road actually consists of, in three parts---soft
rules while you build, hard rules at the point something graduates, and a gate
on every change after. Including the parts currently held together by me
personally, which is a design flaw, and I would rather name it than have you find
it.

I am Mark Hall. I write about local AI, agentic systems, and the engineering
around making useful software trustworthy. You can find the work on
[GitHub](https://github.com/raydeStar) or follow along on
[LinkedIn](https://www.linkedin.com/in/mhall0808/).
