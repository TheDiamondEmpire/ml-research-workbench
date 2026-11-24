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

<!-- BEGIN:E4_TABLE -->
| Condition | n | Result | 95% CI | Notes |
|---|---|---|---|---|
| A1b vs A1 | 75 | bit-identical 75/75 (1.000); within tolerance 75/75 | Wilson: identical [0.951, 1.000], within tol. [0.951, 1.000] | max |d acc| 0.0000, max |d log-loss| 0.00e+00 |
| A2 vs A1 | 75 | bit-identical 75/75 (1.000); within tolerance 75/75 | Wilson: identical [0.951, 1.000], within tol. [0.951, 1.000] | max |d acc| 0.0000, max |d log-loss| 0.00e+00 |
| B1 vs A1 | 75 | bit-identical 30/75 (0.400); within tolerance 72/75 | Wilson: identical [0.297, 0.513], within tol. [0.889, 0.986] | max |d acc| 0.0312, max |d log-loss| 1.85e-02 |
| B2 vs A1 | 75 | bit-identical 30/75 (0.400); within tolerance 72/75 | Wilson: identical [0.297, 0.513], within tol. [0.889, 0.986] | max |d acc| 0.0312, max |d log-loss| 1.85e-02 |
| **H4** | 75 runs per configuration | **not supported** | n/a | tolerance |d acc| <= 0.01; two software environments on one machine (not different hardware) |
<!-- END:E4_TABLE -->

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

<!-- BEGIN:E4_ENVS -->
| Configuration | Env | Threads | python | numpy | scipy | pandas | scikit-learn | lightgbm |
|---|---|---|---|---|---|---|---|---|
| A1 | A | 1 | 3.12.13 | 2.5.4 | 1.18.1 | 3.1.0 | 1.9.1 | 4.7.0 |
| A1b | A | 1 | 3.12.13 | 2.5.4 | 1.18.1 | 3.1.0 | 1.9.1 | 4.7.0 |
| A2 | A | 2 | 3.12.13 | 2.5.4 | 1.18.1 | 3.1.0 | 1.9.1 | 4.7.0 |
| B1 | B | 1 | 3.12.13 | 1.26.4 | 1.12.0 | 2.1.4 | 1.4.2 | 4.3.0 |
| B2 | B | 2 | 3.12.13 | 1.26.4 | 1.12.0 | 2.1.4 | 1.4.2 | 4.3.0 |
<!-- END:E4_ENVS -->

<!-- BEGIN:HARDWARE -->
| Experiment | Logical CPUs | Worker processes x threads | CPU load before / after | Wall clock (min) |
|---|---|---|---|---|
| E1 | 8 | 2 x 1 | 78.0% / 100.0% | 17.6 |
| E3 | 8 | 2 x 1 | 99.9% / 99.5% | 89.8 |
| E4 | 8 | 2 x 1 | 97.9% / 100.0% | 6.4 |
<!-- END:HARDWARE -->
