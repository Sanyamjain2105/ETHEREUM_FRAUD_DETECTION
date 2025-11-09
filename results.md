# Results and Observations

This document compiles evaluation metrics from two phases of the project:

0) Baseline (tabular features with SMOTE) — from `simple models/ETHEREUM FRAUD DETECTION.ipynb`
1) Embeddings (GCN, Node2Vec) + classical models — from `gnn/GNN.ipynb`

- GCN embeddings (GCN feature extractor → classical models)
- Node2Vec embeddings (node2vec feature extractor → classical models)

The same downstream models were used in both settings: XGBoost, Random Forest, and SVM. Metrics are reported on the same held-out test set.

Dataset snapshot from notebook output:
- Final feature matrix shape: 54,911 × 136
- Label distribution: 53,746 normal, 1,165 phishing (≈2.12% phishing in sampled set)

> Note: Precision/Recall/F1 in the tables below refer to the phishing class (binary metrics) as printed by the notebook under “Additional Metrics: Binary”. Overall accuracy and macro averages are also included when available.

---

## 0) Baseline (tabular features + SMOTE)

Source: `simple models/ETHEREUM FRAUD DETECTION.ipynb`

The notebook trains three classical models on the tabular transaction dataset after preprocessing, feature selection, PowerTransformer normalization and SMOTE oversampling. Reported metrics below are taken from the printed `classification_report` and confusion matrices:

| Model | Precision (phish) | Recall (phish) | F1 (phish) | Accuracy | Confusion Matrix (TN FP; FN TP) |
|---|---:|---:|---:|---:|---|
| Logistic Regression | 0.58 | 0.94 | 0.72 | 0.84 | [[1262, 285], [27, 395]] |
| Random Forest | 0.91 | 0.96 | 0.94 | 0.97 | [[1507, 40], [16, 406]] |
| XGBoost | 0.79 | 0.98 | 0.88 | 0.94 | [[1437, 110], [7, 415]] |

Baseline observations:
- All three models benefit from SMOTE; Random Forest achieves the strongest phishing F1 (0.94) on this tabular dataset.
- XGBoost attains the highest recall (0.98) with solid precision (0.79), yielding F1=0.88.
- Logistic Regression trades precision for high recall; still a decent starting point (F1=0.72).

Caveat: The tabular dataset (≈9.8k rows) and its label balance differ from the graph experiments below. Direct numerical comparison should consider dataset and feature differences.

---

## 1) Downstream models on GCN embeddings

| Model | Binary Precision | Binary Recall | Binary F1 | Accuracy | Macro F1 | PR-AUC | ROC-AUC | Train Time |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| XGBoost | 0.5177 | 0.8155 | 0.6333 | 0.9800 | 0.8115 | 0.7990 | 0.9754 | 3.42 s |
| Random Forest | 0.8480 | 0.4549 | 0.5922 | 0.9867 | 0.7927 | 0.7079 | 0.9676 | 85.81 s |
| SVM | 0.4684 | 0.1588 | 0.2372 | 0.9783 | 0.6131 | 0.2998 | 0.9065 | 1180.93 s |

Observations (GCN embeddings):
- XGBoost achieves the best phishing F1 (0.6333) and PR-AUC (0.7990), with very high ROC-AUC (0.9754) and is also the fastest to train.
- Random Forest trades higher precision for lower recall; overall F1 is competitive (0.5922) with strong accuracy (0.9867).
- SVM underperforms on recall and F1 and is significantly slower to train on these embeddings.

---

## 2) Downstream models on Node2Vec embeddings

| Model | Binary Precision | Binary Recall | Binary F1 | Accuracy | Macro F1 | PR-AUC | ROC-AUC | Train Time |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| XGBoost | 0.5314 | 0.6910 | 0.6007 | 0.9805 | 0.7954 | 0.6725 | 0.9647 | 2.94 s |
| Random Forest | 0.0000 | 0.0000 | 0.0000 | 0.9788 | 0.4946 | 0.4458 | 0.9399 | 125.70 s |
| SVM | 0.4324 | 0.0687 | 0.1185 | 0.9783 | 0.5538 | 0.1817 | 0.8241 | 1030.75 s |

Observations (Node2Vec embeddings):
- XGBoost again leads among models, but F1 (0.6007) and PR-AUC (0.6725) are lower than with GCN embeddings.
- Random Forest collapsed on the positive class (0 recall and 0 precision) despite high overall accuracy—classic symptom of extreme class imbalance sensitivity.
- SVM remains the weakest performer on phishing F1 and PR-AUC and is slow.

---

## 3) Cross-setting comparison (GCN vs Node2Vec)

- XGBoost is consistently the best downstream model. It performs better with GCN embeddings than with Node2Vec on phishing F1 (+0.033) and PR-AUC (+0.1265).
- Random Forest is far more robust with GCN embeddings. With Node2Vec, it predicts no phishing in this run.
- SVM is not competitive in either setting for the phishing class.

Overall takeaway:
- For phishing detection under severe class imbalance, GCN-derived embeddings combined with XGBoost offer the best balance of precision–recall trade-off and training efficiency in these experiments.
- Accuracy and ROC-AUC remain very high across models, but PR-AUC and phishing-class F1 are the more meaningful indicators under heavy imbalance.

---

## 4) Figures

The notebook produced PR and ROC curves for each model in both settings. Refer to `gnn/GNN.ipynb` to view them. The legend boxes in those plots match the PR-AUC and ROC-AUC values reported above.

---

## 5) Reproducibility notes

- Source: `gnn/GNN.ipynb`
- Ensure the dataset from EPTransNet (Ethereum Phishing Transaction Network) is available under `gnn/dataset/` as used in the notebook. See the project README for setup details.
- The metrics listed here were transcribed directly from the notebook’s last cell outputs.
