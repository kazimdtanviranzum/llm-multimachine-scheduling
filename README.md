# Interpretable and Transferable LLM Heuristics for Multi-Machine Scheduling

Code and data accompanying the paper *"Interpretable and Transferable Large
Language Model Heuristics for Multi-Machine Flow-Shop and Job-Shop
Scheduling."*

The framework has three coupled modules: an island-based **discovery** loop
that uses a frozen LLM as a mutation operator, an **interpretability** module
that distills each discovered heuristic into a surrogate decision tree plus a
feature-attribution vector, and a **transfer** module that measures and
predicts generalization across problem size and shop type.

## Results in this repository are measured, not projected

Every number in Section 6 of the paper is produced by the code here, on the
released instance sets, and is reproducible. Specifically:

* The **discovered heuristics** that are evaluated (FMDDC and JPC, with their
  augmented variants) are the released functions in `src/heuristics.py`, i.e.
  Algorithms 2–4 of the paper.
* **Exact references** are computed with the exact dynamic program and with the
  open-source **CBC** mixed-integer solver via PuLP (no commercial solver
  needed). On small instances the two agree, which validates the references;
  on larger instances, gaps are reported against the best result found by any
  evaluated method.
* The **discovery loop** (`src/discovery.py`) is the full island-based pipeline
  with an `LLMMutator` interface. To reproduce the LLM-in-the-loop discovery
  itself, plug in a local instruction-tuned model (see *Plugging in a real LLM*
  below). The bundled `NoOpMutator` lets the pipeline be exercised offline.

Measured outputs are saved in `results/results.json` and summarized in
`RESULTS.md`.

## Repository layout

```
.
├── README.md
├── RESULTS.md                # measured headline numbers (Tables 2–12)
├── LICENSE                   # MIT
├── requirements.txt
├── src/
│   ├── data_generation.py    # flow-shop / job-shop instance generator
│   ├── mip_models.py         # MIP (PuLP/CBC) + exact flow-shop DP
│   ├── heuristics.py         # EDD, MDD-load, ATC, FMDDC, JPC, augmented
│   ├── evaluator.py          # feasibility-checking 'evaluate' function
│   ├── discovery.py          # island-based loop (Algorithm 1) + LLM hook
│   ├── interpretability.py   # surrogate tree + attribution + pruning
│   └── transfer.py           # transfer meta-model (gradient boosting)
├── scripts/
│   ├── run_experiments.py    # quick end-to-end smoke test
│   ├── full_experiments.py   # measures Tables 2–12 (phases A–I)
│   ├── jobshop_remeasure.py  # faithful operation-level JPC (Algorithm 3)
│   └── make_figures.py       # regenerates the data figures (8, 9, 10)
├── results/
│   └── results.json          # measured values used in the paper
└── data/
    └── sample_instances/     # small flow-shop and job-shop instances (JSON)
```

## Installation

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Reproducing the measured results

```bash
# 1. quick smoke test on the bundled samples
python scripts/run_experiments.py

# 2. full measurement of Tables 2–12 (writes results/results.json)
python scripts/full_experiments.py --phases ABCDEFGHI
python scripts/jobshop_remeasure.py     # faithful operation-level job-shop

# 3. regenerate the data-bearing figures (8, 9, 10)
python scripts/make_figures.py
```

`full_experiments.py` checkpoints to `results/results.json` after every phase:
A reference validation, B/C exact optimality gaps, D scaling/transfer,
E interpretability, F transfer meta-model, G ablation, H robustness,
I paired statistics (Wilcoxon signed-rank, bootstrap CIs).

## Plugging in a real LLM

The discovery loop calls an `LLMMutator`. Subclass it, build a best-shot prompt
from the two parent functions and the docstring (Appendix A of the paper), call
your local model, parse the returned function body, and return the callable:

```python
from src.discovery import LLMMutator, discover, DiscoveryConfig

class MyMutator(LLMMutator):
    def mutate(self, parents, docstring):
        code = call_local_llm(build_best_shot_prompt(parents, docstring))
        return compile_function(code)

result = discover(seed_fn, evaluate_fn, docstring,
                  mutator=MyMutator(), config=DiscoveryConfig())
```

## Citing

If you use this code, please cite the accompanying paper.

## License

Released under the MIT License (see `LICENSE`).
