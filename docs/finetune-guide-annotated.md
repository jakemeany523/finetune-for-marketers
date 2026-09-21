# The fine-tuning guide, annotated (2026-09-20)

Jake asked for a guide he can learn from, with other cases to run. This is it. Everything here is
either from the source guide, from our own verification, or clearly marked as our judgement.

Source: https://x.com/sairahul1/status/2101250446329364542 (931K impressions)
Full guide: https://x.com/i/article/2100882424343265527

## The core idea, and the one sentence that matters

> Fine-tuning makes a model **consistently** better at ONE narrow task. It does not make a model
> smarter.

That is the whole value proposition. If you take one line from this guide, take that. People skip
it and then wonder why their fine-tuned model is worse at general questions than the base one.

## The mental model: form, not facts

Fine-tuning teaches a model a SHAPE: how to answer, in what register, with what structure, what to
refuse to invent. It does not teach it WHAT HAPPENED THIS WEEK.

| Fine-tuning can improve | Fine-tuning cannot do |
|---|---|
| tone and voice consistency | current events, live data |
| structure and format compliance | competitor intelligence |
| domain vocabulary | real-time facts |
| instruction compliance, reliably | reasoning it could not do before |
| refusal of unsupported claims | |

Rule from the guide, and it is right: **if the gap is knowledge, use retrieval. If the gap is
form, use fine-tuning.** Most "my AI needs fine-tuning" problems are actually knowledge problems
and need RAG instead.

## The three baselines, before ANY training

The guide's strongest instruction, and the one almost everyone skips:

```
Baseline 1: strong prompt-only       (the model you were already using, best possible prompt)
Baseline 2: prompt + retrieval       (same, plus your documents in context)
Baseline 3: fine-tuned model         (only if 1 and 2 fail at something specific)
```

If a good prompt gets you 80% of the way, the fine-tune has to earn its complexity against that
number, not against a lazy prompt. This is exactly the eval discipline LayerLens already runs:
baseline first, bar set in advance, no moving the goalposts.

**The trap:** people compare their fine-tune against "the base model with no prompt" and declare
victory. The comparison is against your BEST prompt, because that is what you were already doing.

## When to reach for it (the real decision tree)

Reach for fine-tuning when ALL of these are true:

1. You have a NARROW, repeated task (same shape, hundreds of times).
2. You have or can build 200-500 EXAMPLES of the task done well.
3. Prompting is failing at something SPECIFIC and measurable (consistency, format compliance,
   tone drift, invented claims).
4. That failure costs you something real (time, quality, trust).

Do NOT fine-tune when:
- the task changes weekly (your examples encode last week's shape),
- you have fewer than ~50 good examples (you will overfit),
- the problem is missing knowledge (retrieval, not training),
- you cannot measure the improvement (if you cannot score it, you cannot prove it helped).

## The 14 steps, annotated with what we verified

| # | Step | The thing that matters | Our note |
|---|---|---|---|
| 1 | Pick a small base model | 0.5B-3B, not 7B+ | Qwen2.5-1.5B-Instruct is the guide's pick; fits a free T4 or a Mac |
| 2 | Data rights BEFORE scraping | public does not mean permission | we made this a repo-level rule; no approved source, no download |
| 3 | Reproducible download | allowlist + hash + manifest | every file sha256'd; duplicates skipped by content |
| 4 | Extract text from hard docs | record failures, do not silently drop | empty results are the hard cases |
| 5 | Clean without destroying | never auto-correct numbers | OCR $12.SM goes to review, not to a language model |
| 6 | Build the training pairs | the task is brief->output, not page->text | someone has to build the bridge |
| 7 | Split by ENTITY, not by page | the mistake that makes evals meaningless | same company in train and test = fake generalization |
| 8 | Right machine for each job | a VPS is not a training machine | Colab for experiments, spot GPU for long runs |
| 9 | QLoRA | 4-bit base + small adapter | ~1/10 the memory of full fine-tune, near-same results |
| 10 | Colab reality | sessions die mid-run | checkpoint every 50 steps, not at the end |
| 11 | GPU strategy | rates change, check live | $0.40/hr T4 is the benchmark rate |
| 12 | EVAL against the baseline | loss is not the metric | unsupported-claim rate, numeric preservation, repetition |
| 13 | Serve the adapter | never expose a raw server | TLS + auth + queue in front, always |
| 14 | Monitor after deploy | drift is silent | retrain on reviewed examples only, with a rollback |

## The costs, from the guide's own numbers

```
first run          ~$1.70   (0.40/hr T4 x 4h incl. retries + storage)
realistic budget   5-10x    = $8.50-17, because the first run is never the last
Colab free tier    $0, flaky, sessions disconnect, storage resets
RunPod / Vast      ~$0.40/hr T4, per-second billing
```

The expensive part is not training once. It is the loop: rebuild the dataset after a bug, try
rank 8 vs 16 vs 32, re-evaluate. Budget for the loop, not the first run.

## The evaluation half is where WE are already strong

The guide's eval metrics, which you will recognize:

- plain-text compliance (no JSON leaking, no slide structure)
- unsupported-claim rate (a number not in the source is a hallucination, catch it before users)
- numeric preservation accuracy
- repetition rate
- readability
- latency and cost per request

That is the hostile gate plus a couple of additions we could add this week. **The guide is
explicit that fine-tuning must be EVALUATED against the prompt-only baseline with a test set that
was burned (never touched) before training started.** Peeking at the test set turns it into
training data and the results become meaningless.

## Other cases worth running (ranked, ours to choose)

Ranked by fit: we have the data, we have the eval rails, the task is narrow and repeated.

1. **Rough product note -> LayerLens-voice social post, no invented claims.**
   We have hundreds of real examples across social-drafts and campaign files. The hostile gate
   and claim-rate checks already exist. This is the fastest to a real verdict and the most
   on-brand for the claim lab. Cost: ~$0-17.

2. **The guide's own demo: startup brief -> funding-pitch copy.**
   Directly comparable to the guide's results, which makes our numbers interpretable against
   theirs. Needs rights-cleared deck sources, which is the slow half. Cost: similar.

3. **Reply drafting in a single author's voice.**
   Narrow, repeated, and we have the reply-style corpus (44 replies analyzed, weekly). Fine-tune
   on one voice, evaluate on whether the reply rate matches the corpus distribution. Medium risk:
   the corpus is small.

4. **Claim classification for the claim lab.**
   A fine-tuned small model that labels a post as claim/workflow/both/etc, trained on our claim
   board. This is the "Jev could do this cheaper" test from the other direction: if a 1.5B model
   we trained does the classification for $0.00001 a call, Jev's $0.04/M input has to beat that.

5. **SEO blog outline -> section drafts that pass the fact gate.**
   We have the programmatic SEO corpus. Risk: broad task, likely needs retrieval too, so it
   tests the guide's "form vs facts" boundary honestly. Good as a SECOND experiment after #1,
   because it is the case most likely to show fine-tuning NOT being enough.

## What to actually do next (the practical sequence)

1. Build the Task A dataset: 200-500 (rough note -> finished post) pairs from our real drafts.
   Half a day of work, mostly scripting plus a manual quality pass.
2. Split by campaign, burn the test set, write the eval scorecard BEFORE training.
3. Baseline 1 and 2 first (one afternoon, uses the existing rails).
4. Train on Colab free tier or a $0.40/hr T4. ~$2-4 for the first adapter.
5. Evaluate all arms. Only then does the verdict exist.
6. If A3 wins: merge the adapter to GGUF, run it in Ollama locally, and the demo is free forever.

## The honest framing for any post

The infrastructure runs the training. The eval design, the dataset judgment, and the verdict are
the human work. A post that says "I fine-tuned a model" would be the fabrication the persona rule
exists to prevent. A post that says "I designed the test and the system ran the training; here is
what survived" is stronger AND true, and it is the claim lab's voice.
