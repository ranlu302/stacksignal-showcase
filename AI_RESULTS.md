# StackSignal AI: measured results

StackSignal adds two language-model features to the deterministic pipeline: **structured
extraction** of job-advertisement facts, and a **question-answering analyst** over the published
market figures. This page reports what was measured, how, and where the figures stop.

[← Back to the project overview](README.md)

## At a glance

- **Extraction:** Claude Haiku reads each advertisement into a fixed schema: skills with a
  requirement level, seniority, minimum years, work mode, work rights and salary. Every value
  must quote the advertisement as evidence. A skill is kept only when the deterministic
  word-boundary matcher finds it too.
- **Analyst:** a model interprets the question and explains the answer. Every statistic comes
  from tested, deterministic code. Each number and citation in the explanation is checked
  against the computed result, and an explanation that fails the check is withheld.
- **Measured on:** a 120-advertisement labelled set (dev and held-out splits), a blind second
  labeller from a different model family, and 36 analyst evaluation cases.
- **Cost:** extracting all 8,194 stored advertisements cost **$5.22** (Batch API, $0.64 per
  1,000 advertisements). The full analyst evaluation cost under $1.

Both features run locally. The hosted demo shows the deterministic dashboard only.

## Extraction

### How it was measured

120 advertisements were sampled across four role groups (Data Engineer, Data Analyst, AI/ML
Engineer, Software Engineer) and split into 30 dev and 90 held-out items. Repeated
advertisements are scored once, which leaves **86 held-out items**. Prompt changes were tuned
on dev only.

The labels were written against explicit rules, for example: options in a list of
alternatives are nice-to-have, "Exposure to X" asks for X, and seniority comes only from
explicit wording. A second labeller from a different model family then labelled the same
advertisements **blind**, with no labels or drafts visible. Disagreements were reviewed
against the rules, and 35 labels were corrected.

| Agreement between the two labellers, before review | Score |
|---|---|
| Skills found (F1) | 0.950 |
| Requirement level, on skills both found | 0.922 |
| Seniority | 0.957 |
| Minimum years of experience | 0.983 |
| Work mode | 0.983 |
| Salary | 0.991 |

### Model comparison (held-out, 86 advertisements)

| Approach | Skills F1 | Requirement level | Cost per 1,000 ads |
|---|---|---|---|
| Deterministic matcher only | 0.917 | — | $0 |
| Claude Opus (earlier prompt) | 0.951 | 0.861 | $42.98 |
| **Claude Haiku, skills kept only where the matcher agrees (chosen)** | **0.938** | **0.896** | **$1.58** |

The chosen setup also scored 0.977 on seniority, 0.988 on minimum years, 0.953 on work mode
and 0.977 on salary. Costs are measured from real runs at list prices without the batch
discount; the batch run costs about half.

**Why this setup:** the deterministic matcher already finds skills well. The model adds what
pattern matching cannot: whether a skill is required or optional, plus seniority, experience,
work mode, work rights and salary. Haiku alone over-reports skills, so the chosen setup keeps
only the skills both methods find. That improved precision without lowering
requirement-level accuracy.

### Whole-store run

| Measure | Result |
|---|---|
| Advertisements submitted | 8,194 |
| Stored | 8,189; 5 replies did not finish and wait for the next run |
| Batch duration | 13 minutes |
| Cost | $5.22 |
| Records with an evidence quote not found verbatim in the advertisement | 335 (4.1%), flagged |
| Skills reported by the model, then kept after the matcher check | 63,349 → 56,837 |

Extraction output is stored beside the authoritative pipeline fields in a separate table. It
never replaces them, and it never feeds the published market metrics.

## Analyst

1. **Interpret:** the model turns the question into a structured intent (summarise or compare;
   role; city), or asks a clarifying question, or declines.
2. **Validate:** the intent is checked against the roles and cities that exist in the data.
3. **Compute:** deterministic tools compute skill-demand shares with 95% intervals, the salary
   quartiles (only when the sample is large enough), role overlap and supporting postings.
4. **Explain and check:** the model explains the result it was given. Every number and posting
   reference in the explanation must appear in that result, allowing ordinary rounding. If one
   doesn't, the explanation is withheld and only the computed figures are shown.

The analyst reads only the published, allowlisted data. Text quoted from postings is treated as
untrusted: numbers inside a posting title cannot justify a number in the explanation.

### Evaluation (Claude Opus, 36 cases)

The cases cover summaries, comparisons, ambiguous questions, too-small samples, unsupported
requests (forecasts, other countries), instructions hidden in posting text, and simulated
failures.

| Measure | Dev (15) | Held-out (21, run once) |
|---|---|---|
| Correct outcome (answered, clarify, insufficient data, unsupported, error) | 15/15 | 21/21 |
| Question interpreted correctly | 14/14 | 20/20 |
| Explanations passing the number and citation check | 8/8 | 8/12 |
| Hidden instructions that changed an answer | 0 | 0 |
| Median latency | 6.3 s | 6.7 s |

**Reading the held-out score:** of the four withheld explanations, three were rejected by the
checker itself. It rounded halves to even, so a correct "53%" for 52.5% failed. The checker now
accepts conventional rounding, with a regression test. The fourth wrote 19.5 as "19" and is
still rejected. The held-out score is reported as measured: the held-out cases were not
re-run after the fix.

## Limits

- The labelled set is small (86 scored held-out items), and the held-out results were inspected
  repeatedly during development. The final model check on held-out is a confirmation, not an
  untouched estimate.
- Agreement between two model labellers measures consistency, not ground truth. 45 of the 120
  labels were written by hand; the rest were model-assisted and checked against the rules.
- The analyst's checker catches invented or altered numbers and references. It cannot catch a
  wrong claim made without a number.
- Role groups come from job titles only. The advertisements reflect one person's search and
  the source restrictions; they are not a representative sample of the Australian market.
- Costs are list prices on the measured date and depend on advertisement length.

These are summaries of recorded internal runs. Advertisement text, labels and per-posting
model output remain private; this page reports aggregate scores only.

---

Built by [Ran Lu](https://github.com/ranlu302) · **StackSignal — Data Engineering Job Market Intelligence**
