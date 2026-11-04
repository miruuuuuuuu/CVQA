# Final model (FINAL = M9 + plain concatenation) - 20 iterations (random 720/80/200 splits, seeds 0..19)

Values are mean ± sample std over iterations. PLCC and RMSE use the 4-parameter logistic mapping fitted on the validation set and applied unchanged to the test set. The test set is used once per iteration.

| Score | Metric | DARNet | Ours (mean ± std) | Δ | iterations better | Wilcoxon p (vs DARNet mean) |
|---|---|---|---|---|---|---|
| TSV | SRCC | 0.910 | 0.897±0.015 | -0.0129 | 6/20 | 0.002 |
| TSV | PLCC | 0.924 | 0.912±0.011 | -0.0118 | 3/20 | 0.000 |
| TSV | KRCC | 0.744 | 0.727±0.020 | -0.0167 | 5/20 | 0.001 |
| TSV | RMSE | 0.252 | 0.271±0.015 | +0.0189 | 2/20 | 0.000 |
| B | SRCC | 0.798 | 0.786±0.034 | -0.0120 | 8/20 | 0.202 |
| B | PLCC | 0.821 | 0.805±0.027 | -0.0157 | 4/20 | 0.024 |
| B | KRCC | 0.607 | 0.595±0.031 | -0.0120 | 8/20 | 0.177 |
| B | RMSE | 0.459 | 0.488±0.032 | +0.0290 | 4/20 | 0.000 |
| SR | SRCC | 0.841 | 0.823±0.028 | -0.0185 | 5/20 | 0.012 |
| SR | PLCC | 0.855 | 0.831±0.022 | -0.0240 | 3/20 | 0.000 |
| SR | KRCC | 0.654 | 0.631±0.031 | -0.0230 | 4/20 | 0.004 |
| SR | RMSE | 0.343 | 0.383±0.024 | +0.0402 | 0/20 | 0.000 |
| S | SRCC | 0.928 | 0.923±0.012 | -0.0052 | 7/20 | 0.053 |
| S | PLCC | 0.933 | 0.927±0.009 | -0.0060 | 5/20 | 0.017 |
| S | KRCC | 0.768 | 0.760±0.017 | -0.0082 | 6/20 | 0.053 |
| S | RMSE | 0.276 | 0.297±0.018 | +0.0208 | 3/20 | 0.000 |
| U | SRCC | 0.906 | 0.900±0.014 | -0.0058 | 6/20 | 0.105 |
| U | PLCC | 0.914 | 0.900±0.013 | -0.0139 | 4/20 | 0.000 |
| U | KRCC | 0.737 | 0.727±0.021 | -0.0099 | 7/20 | 0.076 |
| U | RMSE | 0.283 | 0.318±0.018 | +0.0351 | 1/20 | 0.000 |
| Overall | SRCC | 0.940 | 0.938±0.009 | -0.0020 | 10/20 | 0.409 |
| Overall | PLCC | 0.943 | 0.938±0.007 | -0.0052 | 5/20 | 0.006 |
| Overall | KRCC | 0.790 | 0.783±0.016 | -0.0069 | 7/20 | 0.083 |
| Overall | RMSE | 0.332 | 0.358±0.016 | +0.0257 | 1/20 | 0.000 |

The p-value is a one-sample signed-rank test of the per-iteration values against DARNet's reported mean (DARNet's per-split values are not published, so a paired test is not possible). Treat it as an indication, not a proof.

Mean of the six scores: SRCC 0.878±0.009 (DARNet 0.887) | PLCC 0.886±0.008 (DARNet 0.898) | KRCC 0.704±0.011 (DARNet 0.717) | RMSE 0.352±0.010 (DARNet 0.324)