# finetune-for-marketers

Does a 1.5B model fine-tuned on 200-500 good examples actually beat a frontier model prompted
generically at a narrow marketing task?

The claim (Mark Cuban via @sairahul1, 931K impressions): people who know how to fine-tune small
LLMs on private data are the next job wave. The method: QLoRA on Qwen2.5-1.5B-Instruct, ~$1.70
first run, from the guide this project tests
(https://x.com/sairahul1/status/2101250446329364542).

**This repo is the receipts.** Every command, every number, every arm of the test. If the claim
survives, we say so. If it does not, we say that too. Kill or confirm, both with receipts.

## The experiment, registered before the run

| Arm | What it is | Why it exists |
|---|---|---|
| A1 | Prompt-only GLM-5.3-Flash on held-out briefs | the baseline the claim must beat |
| A2 | Prompt + retrieval over the same corpus | the guide's own warning: use RAG for facts, training for form |
| A3 | QLoRA Qwen2.5-1.5B-Instruct, our pairs | the claim itself |
| A4 | Rubric-only, no model | proves the judge is not the effect |

**Success bar, set before any run:** A3 beats A1 by >= 15% on the hostile-gate score AND halves
the unsupported-claim rate on 30 held-out briefs. Split by campaign, never by example
(the guide's step 7, the mistake that makes evals meaningless).

## Two tasks, both approved

- **Task A (our data):** rough product note -> LayerLens-voice social post, no invented claims.
  ~300-500 pairs from real drafts already written. Evaluation rails already exist.
- **Task B (the guide's own demo):** startup brief -> funding-pitch copy. Only examples we have
  explicit rights to use; source, owner, rights_status recorded per file before download.

## Layout

- `data/` - pair construction scripts and schemas. Raw sources stay OUT of the repo; only
  hashes, manifests and processing code go in (the guide's data-rights rule, step 2).
- `training/` - QLoRA run scripts, hyperparameters, every run's config and loss curve.
- `evaluation/` - the three baselines, the hostile-gate harness, claim-rate and
  numeric-preservation checks, and the results table.
- `serving/` - merged GGUF via Ollama for the live demo.
- `docs/` - the guide itself, annotated; what we changed and why.

## Rules this repo runs under

- Every number is traceable to a command in this repo. If it is not reproducible here, it is not
  a result.
- Vendor benchmarks are claims. Our runs are the only results.
- Raw data never enters the repo; hashes and manifests do.
- Nothing about this project claims Jake hand-trained a model: the infrastructure runs the
  training, the eval design and the verdict are the human work. That distinction is the point.
