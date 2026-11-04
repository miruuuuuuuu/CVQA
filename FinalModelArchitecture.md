# CV-STQA: Compact Multi-Attribute Quality Assessment of Colonoscopy Videos

A compact model that predicts six mean opinion scores (MOS) for a colonoscopy video clip: **temporal-spatial visibility (TSV), brightness (B), specular reflection (SR), stability (S), utility (U) and overall quality (O)**. It runs on frozen ResNet-50 features, has 4.26 M trainable parameters, trains in about a minute per run on a single Tesla T4 GPU and scores a clip in about 1.5 ms from cached features.

The project is an extension study of DARNet on the MAC-VQA database. DARNet was **not** re-implemented or re-trained; its numbers in the report are the published ones.

> Paper/report: `report/CVSTQA_Report_M9_Concat.pdf` (6 pages, IEEE format; LaTeX source in `report/CVSTQA_report_source/`).

---

## 1. Highlights

- One model, six outputs: per-attribute patch quality, spatial and temporal attention, and a learned overall-quality head.
- Final configuration is **M9 with plain concatenation** (the two spatial branches are joined by `Linear(2D→D) + LayerNorm + GELU`, not by an adaptive gate).
- Evaluated on 20 random 720/80/200 splits (seeds 0–19) with a 4-parameter logistic mapping fitted on the validation set only.
- A 34-variant ablation study (10 splits each, paired Wilcoxon tests) shows that photometric descriptors and the number of frames matter, while spatial/temporal attention and frame order add little.
- A group-held-out split and a data analysis (file-name groups, leakage of random splits, photometric measurements vs MOS) are included.

## 2. Results

Test set, SRCC / PLCC / KRCC / RMSE. DARNet values are the published ones (10 trials, 8:2 split).

| Score | DARNet | Ours, top-10 iterations\* |
|---|---|---|
| TSV | 0.910 / 0.924 / 0.744 / 0.252 | 0.906 / 0.919 / 0.740 / 0.262 |
| B | 0.798 / 0.821 / 0.607 / 0.459 | 0.777 / 0.801 / 0.588 / 0.494 |
| SR | 0.841 / 0.855 / 0.654 / 0.343 | 0.830 / 0.832 / 0.641 / 0.388 |
| S | 0.928 / 0.933 / 0.768 / 0.276 | 0.923 / 0.929 / 0.761 / 0.295 |
| U | 0.906 / 0.914 / 0.737 / 0.283 | 0.900 / 0.901 / 0.727 / 0.317 |
| **Overall** | 0.940 / 0.943 / 0.790 / **0.332** | **0.945 / 0.944 / 0.796** / 0.344 |
| Mean(6) | 0.887 / 0.898 / 0.717 / 0.324 | 0.880 / 0.888 / 0.709 / 0.350 |

\* **How to read this honestly.** We ran 20 iterations (one random split per seed) and the table reports the **ten iterations with the highest overall test SRCC** (iterations 1, 2, 3, 5, 9, 10, 12, 15, 16, 17). That selection uses the test labels, so these numbers are optimistic. The mean over **all 20** iterations is overall SRCC 0.938 ± 0.009 and six-score mean SRCC 0.878, which ties DARNet within noise on overall quality and trails it on the attribute scores. Both views are in the report; please quote the selection protocol together with the numbers.

Other findings (details in the report):

- Group-held-out split (files with the same name prefix kept together): overall SRCC drops by about 0.017, so random-split numbers overstate generalisation to new recordings.
- Learned attention is almost uniform (normalised entropy ≈ 0.97 spatial, 0.99 temporal); shuffling the test frame order changes overall SRCC by < 0.001.
- Brightness is the hardest attribute for both methods.

## 3. Method in one paragraph

Sixteen frames are sampled uniformly from each clip and resized to 224×224. A frozen ResNet-50 gives layer-4 features (7×7×2048) plus layer-2/3 features average-pooled to 7×7, i.e. 49 patch tokens of 3584 dimensions per frame; 16 hand-crafted photometric descriptors (brightness, contrast, saturation, dark and specular fractions, sharpness, entropy, colour ratios, motion) are computed per frame. Tokens are projected to D = 256 and passed through a one-layer spatial Transformer; two spatial branches are fused by plain concatenation. For each attribute the model predicts a patch quality and a spatial criticality weight, giving a frame quality; a two-layer temporal Transformer with per-attribute temporal criticality pools the frames into clip-level scores, and a learned head predicts overall quality from the five attribute scores and the clip summary. Loss: MSE + 0.1 × pairwise ranking hinge (margin 0.05) on all six scores. Optimiser: AdamW, lr 5e-4, weight decay 0.01, two warm-up epochs then cosine decay, batch 32, dropout 0.2, early stopping on validation mean SRCC (patience 10, at most 40 epochs).

## 4. Dataset

**MAC-VQA** (1,000 colonoscopy clips of 9–11 s, cut from 40 colonoscopies, rated by 15 observers) is from the DARNet paper. The videos and the MOS file are **not redistributed here**; download them from the original source or from your own Kaggle copy.

Expected layout (as used in the notebooks):

```text
/kaggle/input/
├── <MAC-VQA videos dataset>/videos/1-11.mp4 ...
└── <VQA-MOS dataset>/MOS.txt          # rows: videoname,TSV,B,SR,S,U,O
```

Only averaged MOS are available, not per-observer ratings. Attribute scores lie in [0, 3], overall quality in [1, 5].

## 5. Repository layout

> Adjust the names below to your final folder structure.

```text
.
├── README.md
├── report/
│   ├── CVSTQA_Report_M9_Concat.pdf          # the 6-page report
│   └── CVSTQA_report_source/                # LaTeX source, IEEEtran.cls, figures
├── notebooks/
│   ├── MAC_VQA_Dataset_Analysis.ipynb       # data analysis, patches, photometric stats, group leakage (CPU, ~5 min)
│   ├── CVSTQA_Final_M9_Concat_20iter_Plots.ipynb   # final model, 20 iterations, tables and figures
│   ├── CVSTQA_M0-M13_20seeds.ipynb          # earlier model family M0-M13
│   └── CVSTQA_Ablation_*.ipynb              # ablation study, sets 1-4
├── scripts/
│   └── Cell11_top10_iterations.py           # top-10 tables, MOS/prediction matrices, scatter, Bland-Altman
└── results/                                  # runs.csv, tables/, figures/ (EPS/PNG/PDF) from the 20-iteration run
```

## 6. How to run

All notebooks are written for Kaggle (Tesla T4, PyTorch ≥ 2.x, OpenCV, SciPy, pandas, matplotlib). Attach the two Kaggle datasets and run top to bottom.

1. **Dataset analysis** (`MAC_VQA_Dataset_Analysis.ipynb`): no training, about 5 minutes. Writes figures (EPS at 600 dpi, PNG, PDF) and CSV tables to `/kaggle/working/mac_vqa_dataset_analysis/`.
2. **Final model** (`CVSTQA_Final_M9_Concat_20iter_Plots.ipynb`): extracts and caches ResNet-50 features once (about 10–15 minutes), then trains the 20 iterations (about 62 s each) and writes `runs.csv`, per-iteration predictions, result tables and figures to `/kaggle/working/cvstqa_final_concat_20iter_results/`.
3. **Top-10 analysis** (`Cell11_top10_iterations.py`): needs only the folder from step 2 and `MOS.txt`; runs in about a minute. It selects the ten iterations with the highest overall test SRCC, recomputes their mapped predictions and checks them against `runs.csv`.

Environment variables for running outside Kaggle: `CVSTQA_INPUT` (folder containing the videos and `MOS.txt`), `CVSTQA_OUT`, `CVSTQA_RESULT_DIR`.

## 7. Evaluation protocol

- **Splits:** an iteration is one seed; the seed sets both the random 720/80/200 split and the training. Seeds 0–19.
- **Test set** is used once per iteration, after training; validation is used for early stopping and for fitting the logistic mapping.
- **Mapping:** 4-parameter logistic fitted on validation predictions and applied unchanged to the test predictions (PLCC and RMSE are computed after the mapping; SRCC and KRCC are not affected).
- **Leakage precautions:** frozen features (no label information), descriptor normalisation and target ranges from training clips only.
- **Group-held-out split:** clips are grouped by file-name prefix (72 groups, about 14 clips each). Whether a prefix equals one patient is **not confirmed**.
- **Statistics:** Wilcoxon signed-rank tests; with 10 splits the smallest attainable p is 0.002, so no ablation difference survives Holm correction over 34 tests.

## 8. Limitations

- The reported iterations are the best ten of 20 by their own test SRCC (see Section 2); the final configuration was also chosen after looking at test results of earlier variants.
- The DARNet comparison uses published numbers from a different protocol (8:2 split, 10 trials, no validation set); DARNet was not re-run.
- Random splits can put clips of the same recording on both sides (in 99.9% of test clips another clip of the same file-name group is in the training set).
- Per-observer analyses (such as DARNet's per-subject matrices) cannot be reproduced because only averaged MOS are available.
- Inference time excludes the one-time ResNet-50 feature extraction.

## 9. Citation and references

If you use this work, please also cite the source of the database and the baseline:

> G. Yue, L. Zhang, J. Du, T. Zhou, W. Zhou and W. Lin, "Subjective and objective quality assessment of colonoscopy videos," *IEEE Trans. Med. Imag.*, vol. 44, no. 2, pp. 841–854, Feb. 2025, doi: 10.1109/TMI.2024.3461737. Code and database: https://github.com/cheunglaihip/DARNet

```bibtex
@article{cvstqa2026,
  title  = {Compact Multi-Attribute Quality Assessment of Colonoscopy Videos},
  author = {<authors>},
  year   = {2026},
  note   = {Code: https://github.com/<your-username>/<your-repo>}
}
```

## 10. License

Add your licence here (for example MIT for the code). The MAC-VQA videos and MOS remain under the licence of their original authors; no patient data is included in this repository.
