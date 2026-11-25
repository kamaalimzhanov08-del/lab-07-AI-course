# Lab 07 — Three routes to a better shop: decompose, choose, justify

Content: Reinforcement Learning Foundations, Value-Based Reinforcement Learning · Narxoz University

In Lecture 11.1 a shop ordered stock every morning and you measured policies for it. This lab asks the
question the lecture ended on: **how do you find a good ordering policy, and which way of finding it should
you trust?** There are three ways, and they need different things:

| Route | What it needs | What it gives you |
|---|---|---|
| **Order-up-to sweep** | to be able to *simulate* the shop | the best "top the shelf up to S" rule it tried |
| **Value iteration** | the transition table `P` and the rewards `R` (a *model*) | the optimal policy, exactly |
| **Q-learning** | days of experience only, no model | a policy that improves with the days it has seen |

You write the two learning routes, run **all three under one scoring rule**, change **one condition** of the
shop, and **defend a choice in writing**. Before any code you also write down how you break the task into
parts: that document is graded too, and it has to come first.

Everything in the lab is **synthetic**: invented prices (in tenge), an invented demand, a shop that does not exist.

**No API key. No GPU. No cost.** The notebook needs numpy only, which Colab already has.

**Due: Sunday 6 December 2026, 23:59 (Asia/Almaty).** 15 points.

## Setup

1. **Fork this repository** on GitHub. Everything you hand in goes into your fork.
2. **Write `DECOMPOSITION.md` and commit it first**, before you open the notebook (details below).
3. Open the notebook in Google Colab, from your fork or from here:

   [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/unreal-kz/lab-07-AI-course/blob/main/lab07_three_routes.ipynb)

4. Run the cells in order, one at a time. A **✏️ cell** is a cell where **you** write or choose.

**What it costs you.** The instructor's reference run of every cell took about half a minute on an
Apple-silicon Mac: 32.6 s for the last run, and 26 to 33 s over all the runs so far. **It has not been timed in Colab.** Expect it to be slower
there, but of the same order. Your own cells (value iteration's backup, Q-learning's choice and update) are
short; the time is spent in the supplied harness.

## What the notebook does

- **Part 1: the shop, the scoring rule, the cheapest route.** The Lecture 11.1 shop with one addition: an
  optional **delivery fee** `order_fixed_cost`, paid on every day on which anything is ordered. Every route is
  scored by one rule (below). The first cell prints what the order-up-to levels score.
- **Part 2a: value iteration.** You write the **backup**: the function that turns a value vector `V` into the
  one-step lookahead `Q(s, a)`. The supplied driver repeats it until `V` settles. `check_bellman_backup` tests
  your function on three small cases, one of them computed from the shop's own day-by-day rules.
- **Part 2b: Q-learning.** You write **`epsilon_greedy`** (the choice) and **`q_update`** (the update). The
  supplied driver runs the days. `check_q_learning` tests both. The notebook states the two equations you
  implement, so you can start before Week 12's lecture.
- **Part 3: all three routes, side by side,** on the lecture shop: one shop, one γ, one scoring rule, and
  the two routes that use randomness repeated over the same ten seeds.
- **Part 4: change one condition.** 4a shows, for a range of delivery fees, how far the best order-up-to rule
  is from the optimum. **You choose a fee `K > 0`** (4b) and run all three routes there. 4c gives Q-learning and the
  sweep the same number of days, from 1,000 to 100,000 (the sweep needs at least 2,200: the table says what it used),
  and shows how the answer depends on the budget.
- **Part 5: `results.json`.** The last cell writes your numbers to a file and lists anything missing from the notebook.

## How the measurement works

**The scoring rule (the same for every route).** A policy's score is its **exact expected discounted return**
from the starting stock (5 units), in an *endless* shop with γ = 0.95: `score_policy`, which is Lecture 11.1's
`exact_value` with `horizon=None`. The score has no simulation noise in it. The lecture used 100-day shops,
so its numbers differ a little from these (the notebook prints the difference for order-up-to-4). Value
iteration solves the endless-shop problem, so under this rule its policy is exactly optimal: **no policy scores
above it**, and every other route can be read as "how far from the optimum".

**Regret** is the optimum minus a route's score, in tenge. With a delivery fee the scores can fall below zero, and
a percentage of a negative number means nothing, so the tables report regret.

**Cost.** The tables count **env steps** (days drawn from the shop: samples), **backups** (one update of one
state-and-order pair; value iteration's are *expected* updates over the whole table, Q-learning's is one
*sampled* day) and **sec/run** (wall-clock on your machine, one run: indicative only). Value iteration uses
no days at all because it is given the table. Whether a route can *have* that table is part of what you weigh.

**Seeds.** Routes that use randomness are run on the same ten seeds. The spread across seeds (`sd`, `min`,
`max`) tells you how far one run can be trusted. A difference between two routes that is smaller than that
spread is not yet a finding.

**Q-learning's step size.** The harness hands `q_update` a step size `alpha` that **decays** with the number
of times that state-and-order pair has been updated: `alpha / (1 + n / 50)`, starting from 0.5, with ε = 0.2.
These defaults were picked from a small hand sweep, not tuned further. A *constant* step size is not
equivalent: on the shop with a fee of 200, ten seeds, the instructor measured the mean regret of a constant
α = 0.1 (ε = 0.1) at 69.0, 200.9 and 375.6 tenge after 30,000, 100,000 and 300,000 days (it grew with the
days), against 46.2, 86.2 and 37.2 for the defaults. So "more days" does not by itself mean "a better
policy". You may change `alpha`, `alpha_half_life` and `epsilon`; if you do, say so in your write-up.

**A day costs a day.** 100,000 days of a real shop is about 274 years. Everything the sample-based routes
use here comes from a simulator. Part 4c asks what a budget of days buys; what a real manager can afford is
part of the argument you make in `JUSTIFICATION.md`.

## What you hand in

Everything is committed to **your fork**. Submit the URL of your last commit in Canvas. Late commits are flagged, not hidden.

1. **`DECOMPOSITION.md`**, committed **first**: the first commit in your fork after the fork itself. The
   template is in this repository.
2. **`lab07_three_routes.ipynb`**: run from top to bottom, with all outputs saved. *File → Download → .ipynb*,
   and replace the template in your fork. The last cell reports anything still missing.
3. **`results.json`**: written by the notebook's last cell (download it from Colab's file browser and add it to your fork).
4. **`JUSTIFICATION.md`**: the template in this repository, filled in.

In both templates the first line after the title is `<!-- STUDENT: replace -->`. **Delete that line** when you
replace the guidance with your own text; a file that still carries it counts as not written.

**Every number in your write-up must be one your own notebook printed.** Say which shop (the fee), how many seeds
and how many days each number comes from.

## What is assessed

The assignment is evidence for two ABET Performance Indicators. **PI 1.1**, Problem Decomposition, is **25 %** of
the task (the decomposition, written before any code: 3.75 of the 15 points). **PI 1.3**, Application of Computing
Principles, is **75 %** (the three-route comparison and its written justification: 11.25 of the 15 points).
Each indicator is graded dimension by dimension; the weight is the share of **that indicator**. (The dimensions
and weights are those of the ABET PI assessment document for CSS-4007, tasks HW4, pp. 2-3. This lab is the
course's evidence for those indicators; its task is a reinforcement-learning one rather than the question-answering
task that document describes.)

| PI 1.1 dimension | Weight | Where the evidence is |
|---|---|---|
| Subproblem Identification and Boundaries | 40 % | `DECOMPOSITION.md` §1 |
| Specification of Inputs, Outputs and Data Flow | 35 % | `DECOMPOSITION.md` §2 |
| Constraints, Assumptions and Justification of the Split | 25 % | `DECOMPOSITION.md` §3 |

| PI 1.3 dimension | Weight | Where the evidence is |
|---|---|---|
| Correct Application of the Candidate Strategies | 25 % | the notebook (your code and outputs), `JUSTIFICATION.md` §1 |
| Grounding in Computational Principles | 25 % | `JUSTIFICATION.md` §2 |
| Empirical Evidence for the Choice | 20 % | the notebook, `results.json`, `JUSTIFICATION.md` §3 |
| Defence of the Selected Solution | 20 % | `JUSTIFICATION.md` §4 |
| Limits and Sensitivity of the Conclusion | 10 % | `JUSTIFICATION.md` §5 |

Each dimension is placed at one of four levels by the share of its marks you earn: **Exemplary** from 90 %,
**Satisfactory** from 70 %, **Developing** from 50 %, **Unsatisfactory** below 50 %. The bands are the ones in
the ABET document; the 0 / 50 / 70 / 90 % reading of marks is still awaiting the coordinator's confirmation.
The guiding questions in the two templates are drawn from the descriptions of the Satisfactory and Exemplary levels.

There is **no bonus** for extensions in this lab: the marks sit on the dimensions above.

## AI use

Add half a page saying which AI tools you used, and for what, as the last section of `JUSTIFICATION.md`
(headed `AI-use declaration`); the rules are on the course's AI use policy page in Canvas. Whatever tools you
use, **every number you quote must be one your own notebook printed**, and the principles you cite must be ones
you can explain: a chat model's statement about why an algorithm converges is not evidence that you understand it.

## Glossary

- **State, action, reward**: the stock on the shelf in the morning; the number of units ordered; the day's profit in tenge.
- **Policy**: a rule from states to actions. **Order-up-to S**: every morning order enough to bring the shelf up to `S` units.
- **Delivery fee** (`order_fixed_cost`): a cost paid on every day on which `order > 0`, whatever the size of the order.
- **Backup**: replacing an estimate of a value by a better one computed from the estimates of the next states.
- **Value iteration**: repeating the Bellman optimality backup over the whole table until the values stop moving.
- **Q-learning**: updating one entry of `Q(s, a)` from one observed day, with no model of the shop.
- **ε-greedy**: with probability ε take a random legal action; otherwise the best-looking one.
- **Regret**: optimum minus a route's score, in tenge.
- **Truncation vs termination**: the calendar cutting off a 100-day episode is *truncation*; the shop has no end state, so the update on the last day still looks ahead.

## Files

| File | What it is |
|---|---|
| `lab07_three_routes.ipynb` | The lab: Parts 1-5, no outputs saved |
| `lab07_core.py` | The shop, the harness (drivers, scoring, comparison) and the checks (supplied; pasted unchanged into the notebook's first code cell) |
| `DECOMPOSITION.md`, `JUSTIFICATION.md` | The templates you replace with your own text |
| `README.md` | This file |

Instructor-side material (reference solution, reference run, tests) is kept in the instructor's course
repository and is not needed here.
