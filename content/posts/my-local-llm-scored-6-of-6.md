---
title: "My Local LLM Scored 6/6. It Was Wrong Every Time."
date: 2026-07-27T22:25:45-06:00
draft: false
slug: "my-local-llm-scored-6-of-6"
description: "Six months of trying to make a 1.2B model useful, and the measurement mistakes I made along the way."
tags: ["local-llm", "evaluation", "ai-agents", "mmlu", "sir-thaddeus"]
ShowToc: true
---

One of the tasks in my benchmark asks this:

> How many 5-person committees can be chosen from 12 people?
> Reply with only the integer.

My 1.2B local model answered `60`. The correct answer is 792.

My evaluator passed it.

It passed all six questions in that probe. The model had gotten every single one
wrong. The scorer checked whether the answer *looked* like a bare integer, never
whether it was the right one — so **6/6 passed, 0/6 correct**.

I had not made a small model smarter. I had built an instrument that could be
flattered by good formatting.

## What I actually found

Here's where I landed after six months, before I walk back through how I got
here.

| Question | Answer |
| --- | --- |
| Did scaffolding raise closed-book knowledge scores? | **Not in my tests.** Frozen-runtime MMLU-Pro: raw 10/20, same-prompt direct 10/20, harness 10/20. An earlier 13/20 did not reproduce. |
| Did it raise verified, tool-backed task completion? | **Yes, substantially.** On a sealed 100-case bank, Gemma 4 26B went from 5 to 56 completed cases with the same fixed model. |
| Does more completion mean more reliability? | **Not automatically.** The 1.2B model went 12 → 32 cases and stayed at 1 of 25 complete task families. |
| Did the underlying model matter? | **Yes.** Complete families: 1.2B held at 1/25, an 8B went 3 → 9, Gemma went 0 → 10. |
| What was the mistake underneath all of it? | Measuring answer shape instead of answer value, and treating a knowledge benchmark as a product benchmark. |

Everything below is how I got each of those wrong before I got them right.

## Starting with the easy wins

I started with a hypothesis: a well-designed harness on a small local model could
handle most ordinary tasks and *feel* like something much larger. Not because the
model got smarter — because the system around it did.

The early evidence was encouraging, and in retrospect that was the problem.

Ask a small model whether to walk or drive to a car wash 50 meters away and it
fumbles. Structure the reasoning — what is the user trying to accomplish, what
does that require, what follows — and it gets there. Ask how many R's are in
"strawberry" and it needs a counting tool, not better spelling. Ask what's
heavier, a pound of feathers or a pound of gold, and watch it convert gold to
troy ounces, convert again, and solemnly compare two numbers that were never
different. Hand it a unit converter and the trap closes.

Three puzzles, three fixes, and a growing sense that this was going to work.

## Then I asked an AI to make the tests pass

I wrote about fifteen logic tests and a rough scoring rubric, and told Codex to
iterate until everything came back correct.

It did. That was the problem.

Opus and Codex both have the same habit: they find the easiest path to the
correct output. In this case, the easiest path was hard-coding the answers
because I'd handed them the correct information. Imagine my dismay when I caught
them red-handed in code review.

A stern talking-to only goes so far. I wagged my finger and told them to knock it
off. They obeyed — begrudgingly — and after many, many attempts, I got to
something like the right shape. But it was brittle. Too much of it leaned on
regex formatting; word a question slightly differently and everything would
break. It made me want to pull my hair out.

I was, without knowing the term for it yet, overfitting — with an enthusiastic
assistant doing the overfitting on my behalf.

## The realization that reframed everything

I was reading benchmark results for a newly released model when the thought
landed:

**I could not improve anything until I could measure it.**

Obvious in hindsight. Embarrassing in the moment. I had spent weeks generating
ideas and evaluating them against a rubric I'd written casually, with an agent
that would happily satisfy it by any means available. Every conclusion I'd drawn
was downstream of an instrument I had never validated.

So I went looking for a real benchmark and landed on MMLU-Pro, a more difficult
version of the MMLU benchmark everybody quotes. Closed-book, widely used,
publicly reported. I built a workbench around it and set the agent loose.

## MMLU-Pro was the wrong shape and I didn't know it

The early results looked good, because early results always do. Then they
flattened. I ran it for days. Movement was small, inconsistent, and never
reproduced cleanly.

Eventually the controls told me why. Under a frozen runtime:

| Condition | Result |
| --- | ---: |
| Raw baseline | 10/20 — 50% |
| Same-prompt direct | 10/20 — 50% |
| Current harness | 10/20 — 50% |
| Historical product version | 10/20 — 50% |

Identical, every arm. The earlier 13/20 did not reproduce once the runtime was
pinned, so it's a historical observation, not an uplift. A separate experiment
with sampled voting made things actively *worse* — 37.9% → 27.9% — and that
mechanism was removed.

I'll be straight: I didn't understand what MMLU-Pro was for. It is deliberately
closed-book. It measures what a model can produce from its own parameters, with
no files, no tools, and nothing to consult. That is precisely the thing a 1.2B
model is worst at, and precisely the thing my harness couldn't fix — you cannot
scaffold your way to knowledge that isn't in the weights.

I was measuring a real thing. It just wasn't my thing.

So I split my scoreboard in three and have kept it that way since: **model
capacity** (what it answers closed-book), **harness capability** (what
model-plus-tools completes and verifies), and **product quality** (whether the
whole thing is fast, safe, and controllable). Keeping them apart stops a tool
result from masquerading as learned knowledge, and stops a flat knowledge
benchmark from hiding a real product gain. Both errors were available to me. I
made one of each.

## Admitting I'm not that clever

When running experiments, there is a lot of ego in play — what if I find the
next big thing from a really good idea in my head? Even typing it out, it sounds
foolish. Much of my work was shotgun experimentation, figuring things out as I
went along. But the solid truth is that good work already exists, done by people
much more clever than me. What if I borrow their architecture? What if I steal
their best ideas and run with them?

The idea had merit, but it still left me in the loop — I would have to research,
try an idea, and see if it worked. Or maybe not. What if instead I looked up a
bank of best practices and saw which ones actually worked, in a closed loop,
without me in the system at all? I was coding with my ego and ignoring a pile of
research. Why not tap into that?

So I changed jobs. I stopped being the person who generates ideas and became the
person who filters them:

1. **Quantify.** Build a real benchmark first, so an idea's fate isn't decided
   by whether it feels smarter.
2. **Filter hard.** Use a strict ruleset driven by numbers: keep, discard, or
   test again.
3. **Harvest.** Pull candidates from published research instead of inventing
   them.

Roughly one candidate in ten survived. That felt like failure for about a week,
until I understood that a filter passing 90% of what enters it isn't a filter at
all.

Test ten things, keep one, move the score, keep going.

## Which is how I found the scorer bug

With a real filter running, outputs started getting inspected properly — which
is how I found the `60` that should have been `792`.

The rule checked shape. The task required a bare integer. `60` is a bare integer,
so it passed. The evaluator never established that the integer was correct. The
repair is small enough to be humbling:

```csharp
private static bool StrictAnswerMatches(string final, string expected)
{
    var actual = NormalizeStrictAnswer(final);
    var want = NormalizeStrictAnswer(expected);
    if (actual.Length == 0 || want.Length == 0)
        return false;

    // Numbers compare by value with a small relative tolerance, so the
    // same number written at different precision (0.222222222222 vs
    // 0.2222222222222222) matches while genuinely different values do not.
    if (double.TryParse(actual, System.Globalization.NumberStyles.Float,
            System.Globalization.CultureInfo.InvariantCulture,
            out var actualNumber) &&
        double.TryParse(want, System.Globalization.NumberStyles.Float,
            System.Globalization.CultureInfo.InvariantCulture,
            out var wantNumber))
    {
        var tolerance = 1e-6 * Math.Max(1.0, Math.Abs(wantNumber));
        return Math.Abs(actualNumber - wantNumber) <= tolerance;
    }

    // Letters / non-numeric tokens compare exactly (case-insensitive).
    return string.Equals(actual, want, StringComparison.OrdinalIgnoreCase);
}
```

The fix is public in [commit `1b002911`](https://github.com/raydeStar/sir-thaddeus/commit/1b002911e607b11b568a2f7262fad1811c1ecd2d).
I kept the bad result in the record rather than quietly regrading history,
because a benchmark is software, and software can be wrong in ways that agree
with its author.

Rescored by value, those same six outputs graded 0/6.

Then I gave the harness a calculator and reran the probe. It reached 5/6, then
reproduced that result:

```text
Same task, tool-disabled:
60

Tool-equipped harness:
calculator("comb(12,5)") → 792
Final answer: 792
```

Two qualifications matter more than the number. This was the same *task* under
tool-disabled and tool-equipped conditions, not literally the same prompt — the
equipped version included an explicit tool instruction. And the model did not
learn combinatorics. The harness gave it a way to turn a word problem into a
verifiable operation and then use the result.

That distinction sounds pedantic right up until somebody writes "83.3%" in a
headline and lets the reader infer that reasoning improved.

## The sealed bank

Then I did it properly.

One hundred cases, private, human-reviewed, with fabricated entities and evidence
so nothing could be answered from training data. Twenty-five semantic task
families, four mutations each. Scored strictly: all four mutations pass or the
family fails. Bank hash published before a single measurement was taken. Two
complete repeats of every arm. Temperature zero, one model in memory at a time,
on one desktop with a 4090.

I reviewed all 100 cases myself before sealing them and had to fix more than
half.

| Fixed model | Raw | Before: prompt, no tools | Tools only | After: full harness | Strict families: before → after |
|---|---:|---:|---:|---:|---:|
| LFM 2.5 1.2B | 15/100 | 12/100 | 30/100 | **32/100** | 1/25 → **1/25** |
| LFM 2.5 8B-A1B | 24/100 | 22/100 | 42/100 | **48/100; 49/100** | 3/25 → **9/25** both |
| Gemma 4 26B-A4B | 23/100 | 5/100; 8/100 | 43/100; 46/100 | **56/100 both** | 0/25 → **10/25** both |
| Luna, low reasoning | 22/100; 21/100 | 24/100; 23/100 | 55/100; 58/100 | **66/100; 58/100** | 5/25 → **12/25; 9/25** |

Where a cell contains two numbers, they are the two repeat runs.

The 1.2B result is the uncomfortable one. Tool access rescued a lot of individual
cases — 12 to 30 — and full orchestration added two more. But complete families
never moved. One out of 25, in every single arm.

That's the difference between case lift and consistent capability. A system can
pass one wording of a task, fail three near-identical variants, and still post a
rising score while remaining brittle in practice. If you only track totals, you
will not see it.

The more capable models cleared the bar. The 8B went from 3 complete families to
9; Gemma went from 0 to 10, in both repeats. Orchestration appears to have more
to work with when the underlying model can follow a process consistently — it
converts competence the model already has into consistency it otherwise lacks.

And there is one result I'm not going to explain today. Look at Gemma's
"prompt, no tools" cell: 23 raw, then **5 and 8** once the production prompt is
applied without its tools. The prompt makes the model measurably worse when the
capabilities it promises aren't there. I did not expect that. It has real
product implications, and it needs its own post.

The full campaign — four arms, repeat stability, latency, and the analyzer's own
list of the audit gaps that keep it from being publication-ready — is in the
[sealed campaign report](https://github.com/raydeStar/sir-thaddeus/blob/master/docs/research/SEALED_2026S3_HARNESS_EVIDENCE.md).

## Rules I actually follow now

1. **Grade values, not answer-shaped objects.** A plausible integer is not
   evidence.
2. **Freeze the runtime before attributing anything.** Yesterday's gain can be
   today's missing dependency.
3. **Never let an agent optimize against your only test set.** It will succeed,
   and the success will mean nothing.
4. **Track task families, not totals.** Mutations expose accidental wins.
5. **Publish the failures.** A result that survived contact with an embarrassing
   bug is worth more than a clean graph with no artifacts behind it.

I still believe a small local model can do a large share of ordinary work. What
changed is that I no longer think one benchmark — or one percentage — can
establish it. That takes a weighted portfolio of real tasks, external
verification wherever it's available, and controls that separate what the model
knew from what the system supplied.

Benchmark the job, not the trivia.

The code, the scorer, and the current evidence are in
[Sir Thaddeus on GitHub](https://github.com/raydeStar/sir-thaddeus). If I'm still
measuring the wrong job, tell me which task family would expose it fastest.

