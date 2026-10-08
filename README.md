<div align="center">

# SHL Grammar Scoring Engine

### From spoken responses to grammar scores — with transcript structure, transformer representations, and OOF ensembling.

**769 labeled training clips · 216 test clips · 0–5 grammar score · RMSE + Pearson**

</div>

---

## The idea

A 45–60 second answer contains more information than its individual words. This project tests that hypothesis by extracting **what was said**, measuring **how it was said structurally**, and learning complementary representations of the transcript.

```text
                 45–60s spoken response
                           │
                           ▼
                    Whisper ASR
                           │
                    ┌──────┴──────┐
                    ▼             ▼
             Transcript       Transcript
              statistics        text
                    │             │
                    ▼             ▼
              Ridge model    DeBERTa-v3-base
                    │             │
                    └──────┬──────┘
                           ▼
                  OOF-weighted blend
                           │
                           ▼
                 Final grammar score
                         0–5
```

## What was built

| Signal | Model | What it captures |
|:---|:---|:---|
| **22 transcript features** | Ridge Regression | fluency, sentence structure, repetition, lexical diversity, pauses |
| **Full transcript** | DeBERTa-v3-base | contextual language patterns |
| **OOF predictions** | 49/51 ensemble | complementary errors and more stable predictions |

The transcript-statistics model uses **22 leakage-safe features** covering word counts, sentence structure, repetition, lexical diversity, speech rate, pauses, and ASR confidence.

---

## Results

### OOF validation

| Model | RMSE ↓ | Pearson ↑ |
|:---|---:|---:|
| Mean predictor | 1.238 | — |
| TF-IDF word | 1.156 | — |
| TF-IDF character | 1.093 | — |
| TF-IDF word + character | 1.109 | — |
| Transcript Statistics + Ridge | **0.869056** | **0.712327** |
| DeBERTa-v3-base | 0.873363 | 0.718217 |
| **DeBERTa + Statistics Ridge** | **0.776600** | **0.784835** |

### Why the ensemble worked

The two models were not making the same mistakes.

**Residual correlation: 0.592**

That diversity translated into a substantial OOF improvement:

```text
S5 DeBERTa             0.8734 RMSE
                         │
                         │  −0.0968
                         ▼
49% DeBERTa
51% Statistics Ridge   0.7766 RMSE
```

The blend improved RMSE on **all 5 validation folds**:

| Fold | DeBERTa | Blend | Improvement |
|---:|---:|---:|---:|
| 0 | 0.7550 | **0.6382** | +0.1167 |
| 1 | 0.9958 | **0.8922** | +0.1037 |
| 2 | 0.8541 | **0.7639** | +0.0903 |
| 3 | 0.9342 | **0.8460** | +0.0883 |
| 4 | 0.8057 | **0.7158** | +0.0898 |

> **5/5 folds improved.** The ensemble was selected from OOF validation rather than leaderboard probing.

---

## Engineering choices

**Leakage-safe validation**  
Fixed 5-fold stratification on score bins. Every candidate generates OOF predictions on the same folds.

**Cached ASR**  
Whisper transcripts and derived features are cached so expensive audio processing is not repeated unnecessarily.

**Complementary modeling**  
The pipeline combines a compact interpretable statistical model with a contextual transformer instead of relying on one model alone.

**OOF-first model selection**  
The ensemble weight was selected from training OOF predictions, not by repeatedly probing the public leaderboard.

**Reproducible artifacts**  
OOF predictions, test predictions, and intermediate results are stored under `/kaggle/working/shl/`.

---

## Final submission

**49% DeBERTa-v3-base + 51% Transcript Statistics Ridge**

- **216 / 216** test predictions
- **0 NaN**
- **0 Inf**
- Predictions clipped to the valid **0–5** range
- Submission file: `submission.csv`

---

<div align="center">

### The main takeaway

**Grammar scoring did not need a bigger model alone.  
The strongest improvement came from combining different views of the same speech.**

</div>
