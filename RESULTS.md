# Measured results

All values below are produced by `scripts/full_experiments.py` and
`scripts/jobshop_remeasure.py` and stored in `results/results.json`.
Gaps are percentage optimality gaps unless noted.

## Reference validation (Table 2, flow-shop, m=5)
The exact dynamic program and the CBC MIP agree on the optimum on every class
where the solver proves optimality (e.g. n=8: 1709 from both). On the hardest
class (n=10, RDD=0.8, TF=0.8) the solver does not close the gap in 120 s, so
the DP value (3947) is the reference. DP times: 0.02–0.16 s; MIP times:
6–120 s.

## Flow-shop gaps vs exact optima (Table 3, 15 instances/class)
| Method | Average gap |
|---|---|
| EDD | 21.2% |
| MDD-load | 16.2% |
| ATC | 16.2% |
| FMDDC | 15.7% |
| Aug. FMDDC | 5.6% |

FMDDC beats EDD with a large, significant margin and edges past its MDD seed
and ATC; local search (Aug. FMDDC) gives the main accuracy gain.

## Job-shop gaps vs exact optima (Table 4, faithful Algorithm 3, 10 instances/class)
| Method | Average gap |
|---|---|
| EDD (operation-level) | 84.3% |
| JPC | 71.0% |
| Aug. JPC | 70.7% |

Absolute gaps are large because greedy dispatching is far from optimal on tight
job-shop instances; JPC improves on EDD in every class.

## Statistical comparison (paired, Wilcoxon signed-rank + bootstrap CI)
| Comparison | p-value | mean diff | 95% CI | win rate |
|---|---|---|---|---|
| FMDDC vs EDD | 1.3e-5 | 5.5 pp | [3.3, 7.9] | 0.73 |
| FMDDC vs MDD-load/ATC | 0.066 | 0.6 pp | [0.0, 1.3] | 0.64 |
| JPC vs EDD | 2.6e-5 | 13.4 pp | [7.0, 19.5] | 0.87 |

## Interpretability (Table 7)
| Heuristic | Tree depth | Fidelity | Model-added sub-scores | Redundant |
|---|---|---|---|---|
| FMDDC | 4 | 92.1% | 3 | 2 |
| JPC | 5 | 92.7% | 2 | 1 |

Ablation confirms the flagged terms are redundant: removing them leaves the
exact-optimum gap essentially unchanged (Table 10).

## Transfer meta-model (Table 9 / Section 6.5)
Leave-one-out MAE ≈ 0.23 percentage points over 15 condition profiles; feature
importance dominated by job count (0.71) and machine count (0.24).

## Scaling / runtime (Tables 5, 6, 8)
Plain heuristics run well under 0.15 s at n=500. Augmentation is accurate but
costly (Aug. FMDDC ≈ 30 s at n=500; Aug. JPC ≈ 143 s at n=200), so it suits
small-to-medium instances. Against the best result found, FMDDC stays near
3–7% and JPC widens its lead over EDD as n grows.

## Robustness (Table 11)
Out-of-sample (normal processing times) preserves the method ranking
(FMDDC 13.4%→6.4% in-/out-of-sample; JPC 70.7%→68.5%). Instance-level standard
deviation: FMDDC 7.3 pp, JPC 18.6 pp.
