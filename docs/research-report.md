# Machine Learning Research Workbench: Research Report

> **Where the code is.** The experiment scripts that produced these results are part of the source package, which is distributed separately (see [SOURCE-ACCESS.md](../SOURCE-ACCESS.md)). The raw results they wrote are included in this repository under [`results/`](../results/).

<p align="center"><img src="assets/banner.svg" alt="Machine Learning Research Workbench" width="100%"></p>

> **Status: protocol v0.2, results recorded.** The protocol was written, and the code that implements it was frozen (code content hash `9f3b9641dee014a1`, recorded with every run), before the full experiment run. The tables below are filled by a script from files in `results/`. Verdicts: H1 supported, H2 inconclusive, H3 not supported, H4 not supported (see Section 8). Nothing here reports an outcome that was not measured by code in this repository.

## Abstract

This study asks how large seed-to-seed variance is relative to the differences between baseline models, and how many seeds and what tuning budget are needed for reliable conclusions. It is run on five small public tabular classification datasets with five CPU-friendly model families. The protocol below defines, in advance, how every hypothesis is turned into a number and what counts as support. Scope is deliberately small: five datasets and five models cannot establish what holds for "common benchmarks" in general.

## 1. Research question

How large is seed-to-seed variance relative to the differences between baseline models, and how many seeds and what tuning budget are needed for reliable conclusions?

## 2. Hypotheses and their operationalisation

The blueprint stated four hypotheses. Two of them referred to quantities ("typical reported gap between strong baselines", "architecture differences") that cannot be measured from this repository without citing literature numbers. They are therefore operationalised from data produced here, and the changes are listed in the [change log](#12-change-log).

- **H1.** For the benchmarks studied, seed variance is comparable to or larger than the typical gap between strong baselines.
  - *Seed variance* S: the median, over the 25 (dataset, model) cells, of the sample standard deviation of test accuracy across the 20 audit seeds (seeds 0 to 19), with all randomness varying (split and model).
  - *Typical gap* G_adj: the median, over datasets and adjacent ranks, of the difference between the seed-mean accuracies of adjacent-ranked models (5 models give 4 adjacent gaps per dataset, 20 gaps in total). "Typical reported gap" is thus operationalised as the *measured* typical gap between neighbouring models in this study, not as any literature figure.
  - *Statistic* R = S / G_adj. 95% CI by bootstrapping seeds (the same resampled seed indices for all models of a dataset, independent across datasets; 2000 resamples), conditional on the five datasets.
  - *Rule:* supported if the CI lower bound is at least 0.5 ("comparable" means the SD is at least half the typical gap); not supported if the CI upper bound is below 0.5; otherwise inconclusive. Sensitivity: the median over all 10 model pairs per dataset instead of adjacent ranks (R_all), and a fixed-split variant in which only model randomness varies.
- **H2.** Ranking models by a single seed disagrees with the ranking by a 20-seed mean on a substantial share of pairwise comparisons.
  - *Statistic:* pairwise disagreement, the share of the 10 model pairs in a dataset whose winner under a k-seed mean differs from the winner under the reference (a tie in exactly one of the two counts one half).
  - *Reference (primary, "disjoint"):* mean accuracy over 20 held-out seeds (seeds 20 to 39). The k-seed subsets are drawn from the audit seeds (0 to 19). The blueprint's literal design (subset drawn from the same 20 seeds as the reference) is reported as a secondary analysis; it is biased toward agreement because the subset is part of the reference.
  - *Aggregation:* mean over 1000 random k-subsets, then over the 5 datasets; 95% CI by percentile bootstrap over datasets.
  - *Rule:* "substantial" is fixed at 0.20 (one pair in five). Supported if the CI lower bound for k = 1 is at least 0.20; not supported if the upper bound is below 0.20; otherwise inconclusive.
- **H3.** Unequal tuning budgets change model rankings more often than architecture differences do.
  - The "architecture differences" comparator has no direct measurable counterpart. H3 is replaced by two pre-declared conditions, **both** required for "supported" ("partly supported" if one holds):
    - (a) *Flip rate above noise.* F_budget is the share of pairwise winners that flip when one model's tuning budget is changed threefold (3 or 30 trials) while every other model keeps the reference budget of 10; only the four pairs involving the perturbed model are counted. F_noise is the share of the 10 pairwise winners that flip when the reference scenario is re-scored with a different model seed (same split, same configurations). Per (dataset, tuning-seed) unit, d = F_budget minus F_noise; condition (a) holds if the bootstrap 95% CI lower bound of the mean of d over the 25 units is above 0.
    - (b) *Budget effect at least as large as the typical gap.* The median over the 25 (dataset, model) cells of the absolute change in mean test accuracy between budget 3 and budget 30 is at least the median adjacent-rank gap at the reference budget (10).
- **H4.** The workbench reproduces reference results within tolerance across two software environments. The blueprint said "two hardware environments"; no second machine is available, so this is tested as **two software environments on one machine** and no claim is made about hardware.
  - *Setup:* the same 75-run grid (3 datasets x 5 models x 5 seeds) in environment A (the locked project environment) at 1 thread, A again in a fresh process, A at 2 threads, environment B (older numpy, scipy, scikit-learn, lightgbm, pandas) at 1 and at 2 threads.
  - *Tolerance (declared before running):* per run, absolute accuracy difference at most 0.01 against configuration A at 1 thread. "Bit-identical" means equal SHA-256 of the predicted-probability array.
  - *Rule:* supported iff 100% of runs in every comparison are within tolerance. Bit-identity shares are reported with Wilson 95% intervals.

## 3. Background

Three starting references (see [References](#11-references)) motivate the study, as summarised from their abstracts: variance in machine-learning benchmarks comes from several sources, including data sampling, initialisation and hyperparameter choice (Bouthillier et al., 2021); reproducibility in ML research was addressed at NeurIPS 2019 through a code policy, a reproducibility challenge and a checklist (Pineau et al., 2021); and in deep reinforcement learning, reproducing results is hard because of non-determinism and variance, and significance metrics are needed to tell whether a new method beats a baseline (Henderson et al., 2018). This report takes those claims as the *motivation* only. No number from the literature is used anywhere in the analysis; every quantity is measured from the runs in `results/`. A fuller related-work section is not written yet.

## 4. Method

### 4.1 System under test

The workbench in this repository (tracker, dataset store, seeded runner, sweep module, statistics and diagnostics, report generator). The experiments call the same library code that the command-line interface uses.

### 4.2 Environment

- **Datasets** (all public tabular binary classification from OpenML, accessed through scikit-learn's `fetch_openml`; each is stored as canonical CSV under its SHA-256 and the hashes are in `results/data_manifest.json`): credit-g (id 31), diabetes (id 37), qsar-biodeg (id 1494), phoneme (id 1489), spambase (id 44), all version 1, licence field "Public" in the OpenML metadata. The bundled scikit-learn breast-cancer data is used only for the reproducibility check, as a download fallback and as the third E4 dataset. Generated data is used only in unit tests and is labelled `SYNTHETIC`.
- **Models** (scikit-learn 1.9.1 and LightGBM 4.7.0 in the locked environment): logistic regression, random forest, scikit-learn HistGradientBoosting, LightGBM, and a one-hidden-layer MLP (scikit-learn `MLPClassifier`, used instead of the blueprint's PyTorch to keep the install and runtime small). Features are standardised (numeric) or one-hot encoded (categorical) inside each pipeline.
- **Split:** stratified 75/25 train/test, seeded. With variance source "all" the seed controls both the split and the model randomness; with "model_only" the split is fixed and only the model seed varies.
- **Metric:** test accuracy (primary). Balanced accuracy, log-loss and ROC AUC are recorded in `results/e1_runs.csv` but are not part of any hypothesis test.
- **Seeds:** 40 per (dataset, model, variance source). Seeds 0 to 19 are the audit set; seeds 20 to 39 are the held-out replicate set used as the primary reference in E2.
- **Compute:** at most 2 worker processes with 1 BLAS/OpenMP thread each (E1 and E3; the E4 configurations ran one at a time), on a shared machine whose CPU was busy with other work (hardware and load are recorded in `results/*_hardware.json` and summarised below). The workbench was not run in a container: the Dockerfile in the repository was written but never built.

### 4.3 Baselines

- Single-seed comparison (k = 1 in E2)
- Untuned defaults (all of E1 and E2, budget 0 in E3)
- Equal-budget tuned models (E3 reference)

### 4.4 Independent variables

- Model family (5), dataset (5; 3 in E4), seed count k (E2), tuning budget in trials (E3: 3, 5, 10, 20, 30), software environment and thread count (E4)

### 4.5 Hyperparameter search (E3)

One seeded, sequential TPE study per (dataset, model, seed), 30 trials maximum, 5 startup (random) trials, objective = mean 3-fold stratified CV accuracy on the training split, no pruning. The incumbent after b trials equals the result of a budget-b study (verified by `tests/test_tracker_sweep_repro.py`), so every budget is read from one recorded trial history. Search spaces:

| Model | Space |
|---|---|
| logreg | C in [1e-3, 1e2] (log) |
| rf | n_estimators in [50, 200]; max_depth in [3, 20]; min_samples_leaf in [1, 10]; max_features in [0.2, 1.0] |
| hgb | learning_rate in [0.01, 0.3] (log); max_iter in [50, 200]; max_leaf_nodes in [8, 64]; l2_regularization in [1e-6, 1] (log) |
| lgbm | n_estimators in [50, 300]; learning_rate in [0.01, 0.3] (log); num_leaves in [4, 64]; min_child_samples in [5, 50]; subsample in [0.5, 1.0]; colsample_bytree in [0.5, 1.0] |
| mlp | hidden units in [16, 128] (one layer); alpha in [1e-6, 1e-1] (log); learning_rate_init in [1e-4, 1e-1] (log) |

### 4.6 Metrics

- Mean, standard deviation and 95% percentile-bootstrap CI of test accuracy across seeds
- Pairwise disagreement, Kendall tau-b and top-1 agreement between k-seed and reference rankings
- Share of pairwise winners that flip with tuning budget
- Accuracy and log-loss drift between environments; bit-identity of predicted probabilities

## 5. Experiment matrix

| ID | Experiment | Manipulation | Measured |
|---|---|---|---|
| E1 | Variance audit | 40 seeds per model and dataset (20 audit + 20 held out), two variance sources | SD versus the typical between-model gap (H1) |
| E2 | Seed sufficiency | Subsample 1 to 20 seeds | Ranking agreement versus a 20-seed reference (H2) |
| E3 | Budget fairness | Equal (10) versus unequal (3 or 30) trials | Pairwise winner flips (H3) |
| E4 | Cross-environment reproducibility | Software environment and thread count | Metric drift, bit-identity (H4) |

Each experiment has one script and one command, and a single command runs all of them; they are listed under [Verification](#verification).

## 6. Statistical plan

- **Replication:** 40 seeds per E1 cell (20 audit), 5 tuning seeds per E3 cell, 5 seeds per E4 cell; counts fixed in `experiments/common.py` before data collection.
- **Summaries:** means with 95% percentile-bootstrap CIs (2000 resamples) and SD.
- **Unit of replication and its limits:** seeds for E1 and E3, datasets for E2. With five datasets, dataset-level intervals are wide by construction, and dataset-conditional intervals do not generalise beyond these datasets.
- **Pairwise comparisons:** paired across seeds (same split for every model), Wilcoxon signed-rank, Holm correction within a dataset; effect sizes reported as Cliff's delta and Cohen's d_z.
- **Multiple comparisons:** Holm-Bonferroni over the 10 model pairs of each dataset. Each hypothesis has one primary statistic and one decision rule (Section 2), so no further correction across hypotheses is applied.
- **No silent exclusions:** no run is excluded. Non-converged MLP fits are kept (the warning is suppressed, not the run).
- **Negative results are results:** the verdict for each hypothesis follows the rule above, whatever the data show.
- **Reruns:** the experiment set is run once after the code is frozen; any rerun is disclosed in the change log.

## 7. Results

Tables are generated from `results/` by a script (see [Verification](#verification)). n is the number of independent replicates behind the interval; notes give the unit.

### E1. Variance audit

**Manipulation:** 40 seeds per model and dataset; the first 20 form the audit set.
**Measured:** seed SD versus the typical between-model gap.

<!-- BEGIN:E1_TABLE -->
| Condition | n | Result | 95% CI | Notes |
|---|---|---|---|---|
| Seed SD of test accuracy, all randomness (median over cells) | 25 cells x 20 seeds | 0.0204 | [0.0131, 0.0222] | S; bootstrap over seeds, datasets fixed |
| Adjacent-rank gap between seed-mean accuracies (median over datasets and ranks) | 20 gaps | 0.0054 | [0.0034, 0.0087] | G_adj, the measured 'typical gap' |
| **H1 statistic** R_adj = S / G_adj | 25 cells, 20 gaps | 3.770 | [1.820, 5.455] | verdict: **supported** (threshold 0.5) |
| Sensitivity: R_all = S / median all-pairs gap | 25 cells, 50 pairs | 1.537 | [0.986, 1.786] | G_all = 0.0133 |
| Fixed split, only model randomness varies: S | 25 cells x 20 seeds | 0.0000 | [0.0000, 0.0000] | logistic regression is deterministic, so some cells have SD 0 |
| Fixed split: R_adj | 25 cells, 20 gaps | 0.000 | [0.000, 0.000] | G_adj = 0.0067 |
| Adjacent gaps smaller than the pair's pooled seed SD (all randomness) | 20 gaps | 15 of 20 | n/a | descriptive |
| Model pairs significant after Holm (paired Wilcoxon over 20 seeds) | 50 pairs | 28 of 50 | n/a | Holm within each dataset; with n=20 the smallest attainable p is about 1.9e-6 |
<!-- END:E1_TABLE -->

Per-cell results (variance source "all", 20 audit seeds):

<!-- BEGIN:E1_CELLS -->
| Dataset | Model | n | Mean accuracy | SD | 95% CI of mean |
|---|---|---|---|---|---|
| credit-g | logreg | 20 | 0.7480 | 0.0283 | [0.7366, 0.7608] |
| credit-g | rf | 20 | 0.7556 | 0.0206 | [0.7472, 0.7644] |
| credit-g | hgb | 20 | 0.7480 | 0.0204 | [0.7394, 0.7568] |
| credit-g | lgbm | 20 | 0.7460 | 0.0275 | [0.7354, 0.7584] |
| credit-g | mlp | 20 | 0.7328 | 0.0286 | [0.7208, 0.7448] |
| diabetes | logreg | 20 | 0.7724 | 0.0281 | [0.7599, 0.7839] |
| diabetes | rf | 20 | 0.7591 | 0.0276 | [0.7471, 0.7711] |
| diabetes | hgb | 20 | 0.7411 | 0.0334 | [0.7271, 0.7549] |
| diabetes | lgbm | 20 | 0.7432 | 0.0305 | [0.7299, 0.7563] |
| diabetes | mlp | 20 | 0.7617 | 0.0271 | [0.7503, 0.7732] |
| qsar-biodeg | logreg | 20 | 0.8667 | 0.0267 | [0.8549, 0.8778] |
| qsar-biodeg | rf | 20 | 0.8672 | 0.0166 | [0.8598, 0.8748] |
| qsar-biodeg | hgb | 20 | 0.8634 | 0.0224 | [0.8534, 0.8725] |
| qsar-biodeg | lgbm | 20 | 0.8631 | 0.0161 | [0.8564, 0.8701] |
| qsar-biodeg | mlp | 20 | 0.8805 | 0.0242 | [0.8701, 0.8909] |
| phoneme | logreg | 20 | 0.7512 | 0.0063 | [0.7485, 0.7540] |
| phoneme | rf | 20 | 0.9028 | 0.0076 | [0.8995, 0.9060] |
| phoneme | hgb | 20 | 0.8909 | 0.0071 | [0.8882, 0.8943] |
| phoneme | lgbm | 20 | 0.8912 | 0.0077 | [0.8879, 0.8946] |
| phoneme | mlp | 20 | 0.8550 | 0.0085 | [0.8512, 0.8586] |
| spambase | logreg | 20 | 0.9233 | 0.0083 | [0.9197, 0.9268] |
| spambase | rf | 20 | 0.9528 | 0.0052 | [0.9506, 0.9550] |
| spambase | hgb | 20 | 0.9552 | 0.0050 | [0.9530, 0.9573] |
| spambase | lgbm | 20 | 0.9556 | 0.0054 | [0.9531, 0.9578] |
| spambase | mlp | 20 | 0.9421 | 0.0057 | [0.9397, 0.9444] |
<!-- END:E1_CELLS -->

### E2. Seed sufficiency

**Manipulation:** subsample 1 to 20 seeds.
**Measured:** ranking agreement with a 20-seed reference.

<!-- BEGIN:E2_TABLE -->
| Condition | n | Result | 95% CI | Notes |
|---|---|---|---|---|
| k = 1 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.235 | [0.120, 0.343] | pairwise disagreement; Kendall tau-b 0.537; top-1 agreement 0.550 |
| k = 2 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.195 | [0.093, 0.297] | pairwise disagreement; Kendall tau-b 0.621; top-1 agreement 0.661 |
| k = 3 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.169 | [0.076, 0.267] | pairwise disagreement; Kendall tau-b 0.671; top-1 agreement 0.728 |
| k = 5 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.146 | [0.059, 0.247] | pairwise disagreement; Kendall tau-b 0.718; top-1 agreement 0.811 |
| k = 8 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.131 | [0.048, 0.236] | pairwise disagreement; Kendall tau-b 0.745; top-1 agreement 0.888 |
| k = 10 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.124 | [0.040, 0.231] | pairwise disagreement; Kendall tau-b 0.760; top-1 agreement 0.929 |
| k = 15 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.116 | [0.029, 0.229] | pairwise disagreement; Kendall tau-b 0.776; top-1 agreement 0.966 |
| k = 20 seed(s), disjoint 20-seed reference | 5 datasets x 1000 draws | 0.096 | [0.000, 0.216] | pairwise disagreement; Kendall tau-b 0.818; top-1 agreement 1.000 |
| k = 1, overlapping reference (secondary, biased toward agreement) | 5 datasets x 1000 draws | 0.218 | [0.121, 0.315] | reference includes the subsampled seeds |
| **H2** (k = 1 vs disjoint reference) | 5 datasets | 0.235 | [0.120, 0.343] | verdict: **inconclusive** (threshold 0.2) |
| Smallest k with mean disagreement <= 0.1 (disjoint) | 5 datasets | 20 | n/a | from the point estimates above |
| Seeds needed (80% power, paired t) for the top-2 gap, credit-g (rf vs logreg) | 20 seeds observed | 48 | n/a | mean gap 0.0076, SD of paired difference 0.0183 |
| Seeds needed (80% power, paired t) for the top-2 gap, diabetes (logreg vs mlp) | 20 seeds observed | 47 | n/a | mean gap 0.0107, SD of paired difference 0.0255 |
| Seeds needed (80% power, paired t) for the top-2 gap, phoneme (rf vs lgbm) | 20 seeds observed | 5 | n/a | mean gap 0.0116, SD of paired difference 0.0068 |
| Seeds needed (80% power, paired t) for the top-2 gap, qsar-biodeg (mlp vs rf) | 20 seeds observed | 13 | n/a | mean gap 0.0133, SD of paired difference 0.0154 |
| Seeds needed (80% power, paired t) for the top-2 gap, spambase (lgbm vs hgb) | 20 seeds observed | 486 | n/a | mean gap 0.0004, SD of paired difference 0.0031 |
<!-- END:E2_TABLE -->

### E3. Budget fairness

**Manipulation:** equal versus unequal tuning budgets.
**Measured:** pairwise winner flips.

<!-- BEGIN:E3_TABLE -->
| Condition | n | Result | 95% CI | Notes |
|---|---|---|---|---|
| F_budget: pairwise winners flipped by a 3x budget perturbation of one model (budget 3 or 30 vs reference 10) | 25 units | 0.136 | [0.097, 0.180] | starved 0.147, boosted 0.125 |
| F_noise: winners flipped by re-scoring the reference with another model seed | 25 units | 0.122 | [0.078, 0.172] | same split, same configurations |
| F_budget - F_noise (H3 condition a) | 25 units | 0.014 | [-0.020, 0.048] | condition met: False |
| Median |accuracy(30 trials) - accuracy(3 trials)| per dataset x model cell | 25 cells x 5 seeds | 0.0042 | [0.0024, 0.0070] | budget effect |
| Median adjacent-rank gap at the reference budget (10 trials) | 20 gaps | 0.0053 | [0.0030, 0.0066] | H3 condition b (effect >= gap) met: False |
| Units where starving the reference winner to 3 trials loses it first place | 25 units | 0.320 | n/a | descriptive |
| **H3** verdict | 25 units | **not supported** | n/a | both conditions are required for 'supported' |
<!-- END:E3_TABLE -->

### E4. Cross-environment reproducibility (two software environments, one machine)

**Manipulation:** library versions and thread count.
**Measured:** metric drift and bit-identity of predicted probabilities.

<!-- BEGIN:E4_TABLE -->
| Condition | n | Result | 95% CI | Notes |
|---|---|---|---|---|
| A1b vs A1 | 75 | bit-identical 75/75 (1.000); within tolerance 75/75 | Wilson: identical [0.951, 1.000], within tol. [0.951, 1.000] | max |d acc| 0.0000, max |d log-loss| 0.00e+00 |
| A2 vs A1 | 75 | bit-identical 75/75 (1.000); within tolerance 75/75 | Wilson: identical [0.951, 1.000], within tol. [0.951, 1.000] | max |d acc| 0.0000, max |d log-loss| 0.00e+00 |
| B1 vs A1 | 75 | bit-identical 30/75 (0.400); within tolerance 72/75 | Wilson: identical [0.297, 0.513], within tol. [0.889, 0.986] | max |d acc| 0.0312, max |d log-loss| 1.85e-02 |
| B2 vs A1 | 75 | bit-identical 30/75 (0.400); within tolerance 72/75 | Wilson: identical [0.297, 0.513], within tol. [0.889, 0.986] | max |d acc| 0.0312, max |d log-loss| 1.85e-02 |
| **H4** | 75 runs per configuration | **not supported** | n/a | tolerance |d acc| <= 0.01; two software environments on one machine (not different hardware) |
<!-- END:E4_TABLE -->

Environments used:

<!-- BEGIN:E4_ENVS -->
| Configuration | Env | Threads | python | numpy | scipy | pandas | scikit-learn | lightgbm |
|---|---|---|---|---|---|---|---|---|
| A1 | A | 1 | 3.12.13 | 2.5.4 | 1.18.1 | 3.1.0 | 1.9.1 | 4.7.0 |
| A1b | A | 1 | 3.12.13 | 2.5.4 | 1.18.1 | 3.1.0 | 1.9.1 | 4.7.0 |
| A2 | A | 2 | 3.12.13 | 2.5.4 | 1.18.1 | 3.1.0 | 1.9.1 | 4.7.0 |
| B1 | B | 1 | 3.12.13 | 1.26.4 | 1.12.0 | 2.1.4 | 1.4.2 | 4.3.0 |
| B2 | B | 2 | 3.12.13 | 1.26.4 | 1.12.0 | 2.1.4 | 1.4.2 | 4.3.0 |
<!-- END:E4_ENVS -->

Per-model detail:

<!-- BEGIN:E4_MODELS -->
| Comparison | Model | Bit-identical runs | Max abs accuracy difference |
|---|---|---|---|
| A1b vs A1 | hgb | 15/15 | 0.0000 |
| A1b vs A1 | lgbm | 15/15 | 0.0000 |
| A1b vs A1 | logreg | 15/15 | 0.0000 |
| A1b vs A1 | mlp | 15/15 | 0.0000 |
| A1b vs A1 | rf | 15/15 | 0.0000 |
| A2 vs A1 | hgb | 15/15 | 0.0000 |
| A2 vs A1 | lgbm | 15/15 | 0.0000 |
| A2 vs A1 | logreg | 15/15 | 0.0000 |
| A2 vs A1 | mlp | 15/15 | 0.0000 |
| A2 vs A1 | rf | 15/15 | 0.0000 |
| B1 vs A1 | hgb | 0/15 | 0.0312 |
| B1 vs A1 | lgbm | 15/15 | 0.0000 |
| B1 vs A1 | logreg | 0/15 | 0.0000 |
| B1 vs A1 | mlp | 0/15 | 0.0000 |
| B1 vs A1 | rf | 15/15 | 0.0000 |
| B2 vs A1 | hgb | 0/15 | 0.0312 |
| B2 vs A1 | lgbm | 15/15 | 0.0000 |
| B2 vs A1 | logreg | 0/15 | 0.0000 |
| B2 vs A1 | mlp | 0/15 | 0.0000 |
| B2 vs A1 | rf | 15/15 | 0.0000 |
<!-- END:E4_MODELS -->

### Compute and load

<!-- BEGIN:HARDWARE -->
| Experiment | Logical CPUs | Worker processes x threads | CPU load before / after | Wall clock (min) |
|---|---|---|---|---|
| E1 | 8 | 2 x 1 | 78.0% / 100.0% | 17.6 |
| E3 | 8 | 2 x 1 | 99.9% / 99.5% | 89.8 |
| E4 | 8 | 2 x 1 | 97.9% / 100.0% | 6.4 |
<!-- END:HARDWARE -->

## 8. Discussion

All numbers below are in the Section 7 tables or in the files named; the verdicts follow the rules fixed in Section 2.

**H1 (seed variance versus typical gap): supported, for this setup.** With the split and the model randomness both varying, the median seed SD of test accuracy is 0.0204 while the median gap between adjacent-ranked models is 0.0054, a ratio of 3.77 (95% CI 1.82 to 5.46, well above the 0.5 threshold). In 15 of the 20 adjacent-rank gaps the gap is smaller than the pooled seed SD of the pair. The all-pairs sensitivity ratio is 1.54 (CI 0.99 to 1.79), so the conclusion does not hinge on using adjacent ranks. Seed-paired tests are more sensitive than single runs: 28 of 50 model pairs are significant after Holm with 20 paired seeds, even though a single run's noise usually exceeds the neighbouring gap. Three caveats matter. First, the "seed" here includes the train/test split, and the test sets are small (the datasets have between 768 and 5404 rows and 25% is held out), so finite-test-set sampling is probably a large part of the SD; this was not measured separately. Second, with the split fixed and only model randomness varying, logistic regression, HistGradientBoosting and LightGBM have SD exactly 0 under their default hyperparameters (no early stopping or subsampling is active), so the median SD is 0 and R is 0 by construction; the two stochastic models have median SDs of 0.0070 (random forest) and 0.0076 (MLP) in `results/e1_cells.csv`, compared with a fixed-split typical gap of 0.0067. Model-initialisation noise alone is therefore comparable to the gaps only for the stochastic models. Third, E1 uses untuned defaults on five datasets.

**H2 (single-seed rankings): inconclusive.** A single seed disagrees with the disjoint 20-seed reference on 0.235 of model pairs (CI 0.120 to 0.343), above the 0.20 threshold as a point estimate but with an interval that includes values below it; the rule therefore gives "inconclusive", and five datasets cannot narrow it. Descriptively, Kendall tau-b between a one-seed and the reference ranking is 0.54 and the one-seed top model matches the reference top model 55% of the time; with 10 seeds the top-1 agreement is 0.93. Agreement does not reach the 0.10 disagreement target until k = 20 (0.096), because two independent 20-seed means still disagree about near-ties: for spambase the gap between the two best models is 0.0004, which would need about 486 paired seeds for 80% power, while phoneme needs 5. The seeds required for the top-2 gap therefore range from 5 to 486 across the five datasets, so "how many seeds are enough" has no single answer even in this small set. The overlapping-reference variant (k = 1) gives 0.218, slightly lower than the disjoint 0.235, which is the direction expected from the overlap, but the difference is small relative to the intervals.

**H3 (tuning budgets): not supported.** Changing one model's budget threefold flips 0.136 of the affected pairwise winners (CI 0.097 to 0.180), against 0.122 (CI 0.078 to 0.172) for re-scoring the reference with another model seed; the difference is 0.014 (CI -0.020 to 0.048), so condition (a) fails. The median budget effect on a model's own accuracy (0.0042) is below the median adjacent gap at the reference budget (0.0053), so condition (b) fails too, although the two intervals overlap. Starving the reference winner to 3 trials cost it first place in 32% of the 25 units. In this setup tuning gained little: pooled over datasets, mean test accuracy for HistGradientBoosting goes from 0.8397 untuned to 0.8439 at 30 trials, while the random forest's goes from 0.8521 untuned to 0.8505 at 30 trials (`results/e3_summary.json`). The result should be read with the design limits in Section 9: small budgets, narrow search spaces, five tuning seeds, and a noise floor (F_noise) that is itself low because several models are deterministic once their configuration is fixed.

**H4 (cross-environment reproducibility): not supported under the declared rule.** Within environment A, a fresh-process rerun and a 2-thread run were bit-identical to the 1-thread reference on all 75 runs. Environment B (older numpy, scipy, scikit-learn, lightgbm, pandas) was bit-identical on all 30 random-forest and LightGBM runs, not bit-identical on any logistic-regression, MLP or HistGradientBoosting run (0 of 45), and within the 0.01 accuracy tolerance on 72 of 75 runs. The three runs outside tolerance are all HistGradientBoosting (|difference| 0.0312 on diabetes, 0.0189 on qsar-biodeg, 0.0140 on breast-cancer). For logistic regression and the MLP the predicted probabilities differ in floating-point detail but the accuracy is identical on every run. I did not investigate the cause of the HistGradientBoosting difference; a change in that estimator between scikit-learn 1.4.2 and 1.9.1 is a plausible but untested explanation, and because the whole stack differs in environment B the effect cannot be attributed to a single library. This is a software comparison on one machine; nothing is claimed about different hardware.

**Practical guidance, limited to what was measured here.** On small tabular benchmarks with default models, do not rank models from one seed; compare paired across seeds and report an interval; expect close pairs to need far more than 20 seeds; pin the environment, because bit-exactness held for fresh processes and thread counts but not across library versions.

## 9. Threats to validity

- **Small benchmark set.** Five datasets and five models; dataset-level intervals (E2) rest on five clusters and are wide, and all intervals are conditional on these datasets. Nothing here generalises to "common benchmarks" or to deep models.
- **Untuned defaults in E1 and E2.** Variance and gaps of tuned models may differ. Several defaults are deterministic given the split, which shapes the fixed-split result.
- **Small test sets.** Accuracy on 25% of 768 to 5404 rows is noisy in itself; "seed variance" mixes split, test-set and model noise (only partly separated by the fixed-split variant).
- **Operationalisation choices.** The thresholds (0.5 for H1, 0.20 for H2), the adjacent-rank definition of "typical gap", the replacement of H3's architecture comparator, and the declared E4 tolerance are my choices; they were fixed before the run but remain judgement calls, and other choices could change a verdict (H2's point estimate is above its threshold, for instance).
- **E3 design limits.** TPE with 5 random startup trials (budgets of 3 and 5 are random search), 3-fold CV as the objective, budgets of 3 to 30 trials, narrow published search spaces, 5 tuning seeds. The noise floor F_noise uses only a model-seed change on a fixed split and understates total noise.
- **E4 is two software environments on one machine.** Environment B differs from A in five libraries at once, so differences cannot be attributed to one; the same CPU and BLAS kernels were available to both. No GPU and no second machine were used. Container reproducibility was not tested (the Dockerfile was never built).
- **Shared, heavily loaded machine.** CPU load was 78% to 100% during the runs. Results do not depend on timing (no timing is analysed), but contention can make native processes fail: the first E4 attempt crashed (see change log).
- **Researcher degrees of freedom.** Analysis choices are in Section 2 and the code was frozen before the run; deviations are in the change log. The Discussion's descriptive remarks were written after seeing the results.
- **Hand-typed numbers.** Section 8 quotes some values that are not in the generated tables (per-model fixed-split SDs, mean accuracy by budget, the three out-of-tolerance runs); they were read from `results/e1_cells.csv`, `results/e3_summary.json` and `results/e4/`.

## 10. Reproducibility checklist

- [ ] Code tagged at the commit used for the reported results. *There is no version-control repository here; the code identity is a content hash (`9f3b9641dee014a1`) recorded with every run in the tracker.*
- [x] Environment pinned (lockfile `uv.lock`; recorded as a hash with every run). *No container image: the Dockerfile was never built.*
- [x] Random seeds and run counts recorded (`results/e1_runs.csv`, `e3_scores.csv`, `e4/`).
- [x] Raw results published as CSV and JSON in `results/` alongside the analysis scripts (no notebook).
- [x] Hardware and load described (`results/*_hardware.json`; 8 logical CPUs, 28 GB RAM, CPU busy with other work).
- [ ] One command reproduces every figure and table. *`experiments.all` exists, but in this session it stopped at E4 (native crash) and E4 and the tables were re-run separately, so it has not completed end to end in one go. The report has tables only, no figures.*
- [x] Data sources, licences and versions listed (`results/data_manifest.json`).
- [ ] Independently re-run on another machine. *Not done.*

## Verification

Instructions that need the source repository. From a clean checkout, with `uv` installed:

```bash
uv sync                                  # create the locked environment (Python 3.12)
uv run python -m experiments.e1          # E1 variance audit        -> results/e1_*.csv, e1_summary.json
uv run python -m experiments.e2          # E2 seed sufficiency      -> results/e2_*.csv, e2_summary.json
uv run python -m experiments.e3          # E3 budget fairness       -> results/e3_*.csv, e3_summary.json
uv run python -m experiments.e4          # E4 two software envs     -> results/e4/, e4_summary.json
uv run python -m experiments.tables --fill   # render the tables above from results/
# or everything in one go:
uv run python -m experiments.all
```

`experiments.e4` builds a second environment (environment B) under `.tmp/` with `uv`. Re-analysing without re-running the models: add `--analyze` to e1, e3 or e4; e2 and the tables only read `results/`.

## 11. References

- Bouthillier, X., et al. (2021). Accounting for Variance in Machine Learning Benchmarks. Proceedings of Machine Learning and Systems 3 (MLSys). arXiv:2103.03098.
- Pineau, J., et al. (2021). Improving Reproducibility in Machine Learning Research (A Report from the NeurIPS 2019 Reproducibility Program). Journal of Machine Learning Research 22(164), 1-20.
- Henderson, P., et al. (2018). Deep Reinforcement Learning That Matters. Proceedings of the AAAI Conference on Artificial Intelligence 32(1). DOI 10.1609/aaai.v32i1.11694.

> Title, venue, year and identifiers of these three entries were checked against the publishers' or arXiv listing pages on 2026-10-22 (abstract-level only; the papers were not read in full for this study). No figure from any of them is used here.

## 12. Change log

| Date | Version | Change |
|---|---|---|
| 2026-10-22 | v0.1 | Protocol drafted as a blueprint; no results. |
| 2026-10-22 | v0.2 | Protocol made executable and fixed before the full run. Changes from v0.1: (1) H1 "typical reported gap" operationalised as the median adjacent-rank gap measured in E1; (2) H2 reference made a disjoint 20-seed set (seeds 20 to 39), literal overlapping reference kept as a secondary analysis; (3) H3 "architecture differences" comparator replaced by the two conditions in Section 2; (4) H4 changed from two hardware environments to two software environments on one machine; (5) datasets reduced to five small tabular sets (no image data), five model families (scikit-learn MLP instead of PyTorch), seeds 40 per E1 cell; (6) no CPU/GPU pair; the Dockerfile was written but not built. |
| 2026-10-22 | v0.2 results | Full experiment set run once after the code was frozen (`experiments.all`). E1, E2 and E3 completed. E4 attempt 1 failed: the A1b worker process died with Windows access-violation exit code 3221225477 after it had written its result CSV and before it wrote its environment file; `experiments.all` stopped there. **Rerun:** E4 was run again in full with the same command and unchanged code; the first attempt's partial files are kept in `results/e4_attempt1/` (its A1 and A1b runs were bit-identical to each other, 75 of 75, and to the rerun's A1). The tables were then rendered with `experiments.tables --fill`. No protocol rule, threshold or analysis code was changed after seeing results. |

---

[README](../README.md) · [Client case study](case-study.md) · [Research report](research-report.md)
