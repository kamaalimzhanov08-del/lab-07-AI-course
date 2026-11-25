# JUSTIFICATION — which route, and why

All numbers below were printed by my notebook and are in `results.json`. Unless stated otherwise: γ = 0.95,
start stock 5, endless-shop scoring, ten seeds (0–9), Q-learning with the default step rule
`alpha / (1 + n / 50)` from 0.5 and ε = 0.2.

## 1. What was held identical

**What I wrote.** `bellman_backup` (the one-step lookahead `Q = R + γ·P·V`), `epsilon_greedy` (the choice)
and `q_update` (one Q-learning step, in place). The shop, the drivers `run_value_iteration`,
`run_q_learning`, `run_order_up_to_sweep`, the scoring and `compare_strategies` were supplied and not changed.
I did not change `alpha`, `alpha_half_life` or `epsilon`.

**Checks.** `check_bellman_backup` printed "all three tests passed" and `check_q_learning` printed "all tests
passed"; the last cell reported "nothing missing in the notebook itself". The checks test that the backup
gives the right numbers on three small cases, that ε-greedy picks the best legal order at ε = 0, only legal
orders at ε = 1 and the greedy order about 80–90 % of the time at ε = 0.2, and that the update moves one
entry by the right amount and touches nothing else. They do not test that Q-learning *converges* or that
the final policy is good; that is seen only in the regret tables.

**Identical for all three routes.** The shop (`SHOP` in Part 3, `SHOP_K` in Part 4), `GAMMA = 0.95`, the
scoring rule `score_policy`, and `SEEDS = tuple(range(10))` for the two routes that use randomness. All are
set once in the first cells of Part 1 and passed to `compare_strategies`.

**Not identical.** The days each route used. In Parts 3 and 4b the sweep used 33,000 simulated days,
Q-learning 100,000, and value iteration 0 (it did about 25,000 expected backups instead: 25,476 on the
lecture shop, 24,882 at fee 200). Part 4c removes this difference for the two sample-based routes; even
there, at the two smallest budgets the sweep used 2,200 days against Q-learning's 1,000 and 3,000.

## 2. Why each route behaves as it does

**Value iteration: dynamic programming on a known model.** The Bellman optimality backup is a contraction
with factor γ, so repeating it from any start converges to the one fixed point `V*`, and the greedy policy
of `V*` is optimal. I therefore expected regret exactly 0 on any shop, with no dependence on seeds. The run
confirmed it: 386 sweeps on the lecture shop, score 8,460.2, and at fee 200 the optimum 6,065.5 with sd 0.0
and 0 days of experience. Where it stops applying: when `P` and `R` are not known, or are known wrongly. It
would then be optimal for the wrong shop, and nothing in its own output would show it. It also stops being
practical when the table is too large to sweep.

**Q-learning: a sampled version of the same backup.** It applies the same optimality equation, but to one
`(s, a)` pair per day, with the expectation replaced by the one next stock that actually happened. Its
convergence needs every pair to be tried again and again (ε-greedy) and a step size that decays. With a
finite number of days I expected it to get close to the optimum but not onto it, and to differ from seed to
seed. The run showed both: at fee 200 after 100,000 days the mean regret was 86.2 with sd 108.2, best run
6,044.3, worst run 5,691.6; the mean regret fell from 641.8 at 1,000 days to 46.2 at 30,000 days. Because
the target uses the max over next orders, it learns about the greedy policy while behaving ε-greedily, so
the 20 % of random orders do not spoil the policy it reports. Where it stops applying: when days are
expensive (it has nothing but days), when exploration costs real money, or when the shop changes faster
than it can learn.

**Order-up-to sweep: Monte Carlo evaluation of a small family of rules.** It averages simulated returns for
11 rules and keeps the best. Two things follow. By the law of large numbers the averages become reliable
with more episodes, and it can never do better than the best rule in its family. On the lecture shop the
optimal policy is itself "order up to 4 when stock <= 3", which is in the family, so the sweep had regret
0.0 on all ten seeds. With a fee the optimal policy is "order up to 6 when stock <= 2": it waits before
ordering so that the fee is paid less often. That is not an order-up-to rule, so the sweep's regret was
983.3 on every seed, and more days did not help: 983.3 at both 29,700 and 99,000 days. With only 2 or 9
episodes per level the averages were still noisy and it sometimes picked a worse level (mean regret 1,004.2,
max 1,192.5). Where it stops applying: whenever the best policy lies outside the family it searches.

## 3. What was measured

**Conditions.** Score = exact expected discounted return from stock 5, endless shop, γ = 0.95 (no
simulation noise in the score). Seeds 0–9. Sweep: 30 episodes of 100 days per level = 33,000 days.
Q-learning: 100,000 days, defaults as above. The notebook runs top to bottom with fixed seeds, so the
numbers can be reproduced from it.

**Lecture shop (fee 0), Part 3.**

| route | mean score | sd | min | max | mean regret | days |
|---|---|---|---|---|---|---|
| order-up-to sweep | 8,460.2 | 0.0 | 8,460.2 | 8,460.2 | 0.0 | 33,000 |
| value iteration | 8,460.2 | 0.0 | 8,460.2 | 8,460.2 | 0.0 | 0 |
| Q-learning | 8,421.0 | 41.5 | 8,341.5 | 8,460.2 | 39.2 | 100,000 |

**Fee 200, Part 4b.**

| route | mean score | sd | min | max | mean regret | days |
|---|---|---|---|---|---|---|
| order-up-to sweep | 5,082.2 | 0.0 | 5,082.2 | 5,082.2 | 983.3 | 33,000 |
| value iteration | 6,065.5 | 0.0 | 6,065.5 | 6,065.5 | 0.0 | 0 |
| Q-learning | 5,979.3 | 108.2 | 5,691.6 | 6,044.3 | 86.2 | 100,000 |

**Fee 200, equal budgets of days, Part 4c (regret, ten seeds).**

| days | Q-learning mean | sd | max | sweep days used | sweep mean | sweep max |
|---|---|---|---|---|---|---|
| 1,000 | 641.8 | 355.3 | 1,424.8 | 2,200 | 1,004.2 | 1,192.5 |
| 3,000 | 351.4 | 221.3 | 747.6 | 2,200 | 1,004.2 | 1,192.5 |
| 10,000 | 204.6 | 203.2 | 687.3 | 9,900 | 1,004.2 | 1,192.5 |
| 30,000 | 46.2 | 40.5 | 135.5 | 29,700 | 983.3 | 983.3 |
| 100,000 | 86.2 | 108.2 | 373.9 | 99,000 | 983.3 | 983.3 |

Q-learning's mean regret was below the sweep's at every budget.

**Why fee 200.** The task asks for a fee where the best order-up-to rule loses clearly. In the table of
Part 4a its regret is 54.3 at fee 50, 352.1 at fee 100 and 983.3 at fee 200. At 50 the loss is about the
size of Q-learning's spread between seeds, so it would be hard to tell the routes apart. At 200 the fee
equals the margin of one unit, the optimal policy changes shape ("order up to 6 when stock <= 2"), and the
sweep's loss is far larger than any seed-to-seed spread. Fees of 300 and 400 make the gap larger still but
add nothing new.

**Why regret and not score.** The optimum differs between shops (8,460.2 without a fee, 6,065.5 with fee
200), so raw scores cannot be compared across conditions. Regret puts every route on one scale, "tenge lost
against the best possible". With a fee, scores can also fall below zero, where a percentage would mean nothing.

**Why the same seeds.** Each seed gives the two random routes the same stream of random numbers to start
from, so a difference between them comes from the route and not from one of them having a luckier draw. It
also makes the run repeatable.

## 4. The choice, and what would reverse it

**Recommendation: value iteration, for a manager who has sales records and knows their own prices and costs.**
Such a manager can build the table: price, unit cost, holding cost and the delivery fee are in the accounts,
and the demand probabilities can be counted from past daily sales. For this manager a day of experience is
expensive: it is a real day, with real unsold stock or real lost sales, and it cannot be repeated.

**Against Q-learning.** At fee 200 value iteration had regret 0.0 with 0 days and 24,882 backups, which took
a fraction of a second. Q-learning after 100,000 days still lost 86.2 tenge on average and 373.9 in its
worst run, and after 1,000 days it lost 641.8 on average and 1,424.8 in the worst run. The README notes
that 100,000 days of a real shop is about 274 years. Q-learning is also less dependable: its result changes
with the seed (sd 108.2), while value iteration gives one answer.

**Against the sweep.** Without a fee they tie: both have regret 0.0. With fee 200 the sweep loses 983.3 on
every seed and no number of days repairs it. The sweep also needs a simulator of the shop, and a simulator
is already a model: a manager who can simulate the shop can write down the table and run value iteration
on it for less work.

**What would reverse the choice.**

- *No table and no way to estimate one, but a trustworthy simulator or very cheap days.* Then Q-learning:
  with a fee it beat the sweep at every budget in Part 4c (for example 204.6 against 1,004.2 at about
  10,000 days), and it needs no knowledge of the policy's shape.
- *No delivery fee, and the manager wants a rule a shop assistant can follow.* Then the sweep: regret 0.0
  with 33,000 simulated days, and the answer is one number, "top up to 4". Q-learning was worse here
  (39.2) and used three times the days.
- *The shop changes over time* (seasonal demand, new prices). A table estimated once goes stale; value
  iteration would have to be re-run on a fresh table, or a learning route kept running.
- *A much larger problem* (many products, large capacity). The table grows and full sweeps over it become
  expensive; sample-based methods then become the practical option.

## 5. What this comparison cannot settle

- **One shop, one demand distribution, one changed condition.** I ran the three-route comparison at two
  fees only (0 and 200). The ranking of Q-learning against the sweep reversed between them, so it should
  not be assumed for other shops.
- **Ten seeds.** Q-learning's spread is large next to its mean (86.2 with sd 108.2 at 100,000 days). Its
  mean regret at 30,000 days (46.2) is lower than at 100,000 days (86.2), but that difference is smaller
  than the spread, so it is not a finding that more days made it worse. The same holds for 39.2 on the
  lecture shop against 86.2 with the fee.
- **Differences that are safe.** The sweep's 983.3 at fee 200 is far beyond Q-learning's worst run (373.9),
  and value iteration's 0.0 is exact, not an estimate. So "value iteration first, sweep last when there is
  a fee" is firm; the size of Q-learning's gap is not.
- **Learner settings not varied.** I used only the default `alpha`, half-life and ε. Other settings could
  move Q-learning's numbers in either direction.
- **The scoring favours value iteration by construction.** It is scored on the very table it was given, so
  its zero regret shows that the method is correct, not that a real manager's estimated table would be.
  This is the main limit on the claim in section 4: the recommendation holds as far as the table is right.
- **Cost of exploring is not in the score.** Only the final policy is scored; the money Q-learning loses
  while trying random orders during learning is not counted.

**What I would run next:** value iteration on a table estimated from a limited number of observed days,
scored on the true shop, to see how many days of records it needs to beat Q-learning given the same days;
more seeds (for example 50) to narrow Q-learning's spread; and a few more fees and demand distributions.

## 6. AI-use declaration

I used an AI assistant (a chat-based agent) in this assignment, and its part was large:

- it wrote the three functions `bellman_backup`, `epsilon_greedy` and `q_update` from the equations given in
  the notebook;
- it proposed the delivery fee of 200 from the table in Part 4a;
- it drafted the text of `DECOMPOSITION.md` and of this document, including the explanations of the
  principles behind the three routes.

My own part: I ran the notebook from top to bottom in Google Colab, confirmed that both check functions
passed and that the last cell reported nothing missing, and compared the numbers quoted in this document
with the ones my run printed. They match, and they are the numbers saved in the notebook outputs and in
`results.json`. I read the code and the documents before submitting them.
