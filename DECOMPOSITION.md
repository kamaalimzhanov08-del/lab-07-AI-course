# DECOMPOSITION — the task in named parts, before any code

Written from the README, before opening the notebook. The task: given the shop, obtain an ordering policy
by three routes, score all three by one rule, change one condition, and recommend one route in writing.

## 1. The subproblems

| # | Name | What it does |
|---|---|---|
| S1 | Shop model | Describes the shop as an MDP: stock levels as states, order sizes as actions, the transition table `P`, the expected rewards `R`, and which orders fit on the shelf. Also lets us draw one simulated day. |
| S2 | Scoring rule | Takes any policy and returns its exact expected discounted return from start stock 5 in the endless shop. Regret = optimum minus this score. |
| S3 | Route A: order-up-to sweep | Simulates each rule "top the shelf up to S" for S = 0..10 and keeps the level with the best average return. |
| S4 | Route B: value iteration | Repeats the Bellman optimality backup over the whole table until `V` stops moving, then reads off the greedy policy. |
| S5 | Route C: Q-learning | Lives in the shop day by day, chooses orders ε-greedily, updates one `Q(s, a)` entry per day, then reads off the greedy policy. |
| S6 | Comparison harness | Runs S3–S5 on the same shop, γ and seeds, sends every policy to S2, and collects score, regret, spread across seeds and cost (days, backups, seconds). |
| S7 | Changed condition | Repeats S6 on a shop with a delivery fee `K > 0`, and again with equal budgets of days for the two sample-based routes. |
| S8 | Recommendation | Turns the tables from S6 and S7 into a written choice of route for a named kind of manager. |

**Boundaries.** S1 owns everything about the shop; no route defines prices or demand itself. S2 is the only
place a policy gets a number, so no route scores itself. S3–S5 only *produce* a policy and report what they
spent. S6 and S7 only *run and collect*; they contain no learning. S8 contains no computation.

**Completeness check.** Each verb of the task maps to exactly one part: "obtain a policy by each of three
routes" → S3, S4, S5; "score by one rule" → S2 (applied by S6); "change one condition" → S7; "recommend" → S8;
"given the shop" → S1. Nothing is left over, and no verb is served by two parts.

**Could it be smaller?** S6 and S7 could be merged (S7 is S6 with another shop). I keep them apart because
S7 adds two decisions S6 does not have: which fee to choose and which budgets to compare. S3–S5 cannot be
merged: they need different things (a simulator, the table, experience), and that difference is the point of
the lab.

## 2. Inputs, outputs and data flow

Notation: `nS` = 11 states (stock 0..10), `nA` = 11 actions (order 0..10).

| # | Inputs | Outputs | If something is wrong |
|---|---|---|---|
| S1 | `ShopParams` (capacity, price, unit cost, holding and shortage cost, demand probabilities, start stock, `order_fixed_cost`) | `p_next`: float array `(nA, nS, nS)`; `reward`: float `(nS, nA)`; `mask`: bool `(nS, nA)`; a step function `(state, action, rng) → (reward, next_state)` | Demand probabilities must sum to 1 and every row of `p_next` must sum to 1; otherwise stop with an error. |
| S2 | `ShopParams`, a policy (array `(nS, nA)` of action probabilities), `gamma: float` | `score: float` (tenge) | A policy that puts probability on an order that does not fit is rejected, not scored. |
| S3 | `ShopParams`, `gamma`, `episodes: int`, `seed: int` | policy; chosen level `S: int`; days used `int` | With too few episodes the averages are noisy and the wrong level may be picked: report days used so the reader sees it. |
| S4 | `p_next`, `reward`, `mask`, `gamma`, tolerance; my function `bellman_backup(p_next, reward, mask, v, gamma) → q (nS, nA)` | policy; `V: (nS,)`; number of sweeps and backups; 0 days | If `V` has not settled within a sweep limit, report non-convergence instead of returning a policy. Orders that do not fit are ignored when taking the max. |
| S5 | step function from S1, `mask`, `gamma`, `n_steps: int`, `seed: int`, `alpha`, `epsilon`; my functions `epsilon_greedy(q, state, epsilon, mask, rng) → action: int` and `q_update(q, state, action, reward, next_state, alpha, gamma, mask) → None` (in place) | policy (greedy in `Q`); `Q: (nS, nA)` with `-inf` on illegal orders; days and backups used | Illegal orders are never chosen (mask). A pair that was never visited keeps its initial value; the result is still a legal policy but may be poor, which shows up as regret. |
| S6 | `ShopParams`, `gamma`, `seeds: tuple[int]` (ten), `q_steps`, the three routes | table: per route mean score, sd, min, max, mean regret, days, backups, seconds | Refuses to run if a route's check function fails. |
| S7 | everything S6 takes, plus `K: int > 0` and `budgets: tuple[int]` | fee table (optimum vs best order-up-to per fee), S6 table at `K`, regret-per-budget table | `K <= 0` is reported as missing. |
| S8 | tables from S6 and S7, `results.json` | `JUSTIFICATION.md` | A number not printed by the notebook is not allowed in the text. |

**Data flow.**

```
ShopParams ──► S1 ──► P, R, mask ──────────► S4 ─┐
                │                                 │
                └──► step function ──► S3, S5 ────┤  policies
                                                  ▼
                                    S2 (one scoring rule, same γ)
                                                  │ scores
                                                  ▼
                 S6 (lecture shop)  ──►  S7 (fee K, budgets)  ──►  results.json  ──►  S8
```

1. S1 builds the model from `ShopParams`.
2. S4 reads `P`, `R`, `mask`; S3 and S5 only call the step function.
3. Each route hands a policy to S2; S4's score is the optimum, used to compute regret for the others.
4. S6 gathers the rows; S7 repeats steps 1–3 with a new `ShopParams(order_fixed_cost=K)`.
5. S8 reads only the saved tables.

Every route has the same output type (a policy), so any one of them can be written and tested without
seeing the other two.

## 3. Constraints, assumptions and the reason for the split

**Constraints.**

- S3 and S5 may not read `P` or `R`; they may only sample days. S4 may not sample days.
- S2, `gamma` = 0.95, the start stock and the ten seeds must be identical for every route (S6, S7).
- S7 changes exactly one thing, the delivery fee. Everything else stays as in S6.
- The shop, the drivers and the scoring are supplied and must not be edited; I write only the backup, the
  ε-greedy choice and the Q-update.
- Numpy only; the whole run should take about a minute or less.
- S8 may quote only numbers printed by the notebook.

**Assumptions.**

- The shop is a stationary MDP: tomorrow's stock depends only on today's stock and order, and demand has the
  same distribution every day (matters for S1, and through it for S4 and S5).
- The table handed to S4 is the true one (matters for S4 and for S2, which also uses it).
- The simulator's days stand in for real days (S3, S5, S7).
- Ten seeds are enough to see the spread of a route (S6, S7).
- The supplied step size and ε for Q-learning are reasonable; I do not tune them (S5).

**Why this split.** The split follows who needs what: the model (S1), the judge (S2), three producers of
policies (S3–S5), and the experiment around them (S6, S7). Keeping the judge outside the routes is what
makes the comparison fair: a route cannot be scored in a way that flatters it.

**Alternative I rejected.** Three self-contained pipelines, one per route, each with its own shop setup,
training, evaluation and report. It is simpler to write route by route, but the scoring code would exist
three times, and any small difference between the copies (horizon, start stock, γ) would make the numbers
incomparable. It also has no natural place for "change one condition": the change would have to be made in
three places.

**The assumption that would hurt most.** That the table `P` is the true shop. If it is wrong, value
iteration is optimal for the wrong shop, and the scoring rule, which uses the same table, would not notice.

- Which assumptions are you making, and for which subproblem does each matter?
- Why this split and not another? Name **one specific alternative split you considered and rejected**,
  and say why you rejected it.
- Which single assumption would hurt your approach most if it were wrong?
