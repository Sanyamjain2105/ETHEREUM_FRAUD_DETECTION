# Ethereum Fraud Detection — Baselines → Embeddings → Comparison

This repository explores phishing detection on the Ethereum transaction network using graph-derived embeddings (GCN and Node2Vec) and downstream classical models (XGBoost, Random Forest, SVM). The focus is on meaningful metrics under heavy class imbalance.

## Repository layout

- `simple models/ETHEREUM FRAUD DETECTION.ipynb` — baseline tabular pipeline with preprocessing, SMOTE, and classical models (Logistic Regression, Random Forest, XGBoost).
- `gnn/GNN.ipynb` — embeddings pipeline that builds GCN/Node2Vec embeddings and evaluates the same downstream models. Final cells print the metrics compiled in `results.md`.
- `gnn/dataset/` — place the EPTransNet dataset here (see Dataset below).
- `results.md` — consolidated results and observations copied from the notebook outputs.

## Dataset

We use the EPTransNet (Ethereum Phishing Transaction Network) dataset by InPlusLab (Sun Yat-sen University).
- Type: `networkx.classes.multidigraph.MultiDiGraph`
- Scale (from dataset docs): ~2.97M nodes, ~13.55M edges
- Labels: node attribute `isp` (1 = phishing, 0 = normal)

Place the dataset as:
```
./gnn/dataset/Ethereum Phishing Transaction Network/MulDiGraph.pkl
```
Refer to the dataset README for details (`gnn/dataset/Ethereum Phishing Transaction Network/README.txt`).

## How to run (notebook)

1) Open `gnn/GNN.ipynb` in VS Code or Jupyter.
2) Ensure the dataset path in the notebook matches your local path (`MulDiGraph.pkl`).
3) Run all cells; the last cells print metrics and display PR/ROC curves.

Optional environment suggestions (Windows, PowerShell):
- Python ≥ 3.10
- Packages: `networkx`, `numpy`, `pandas`, `scikit-learn`, `xgboost`, `matplotlib`, `seaborn`, `tqdm` (and optionally `torch`, `torch-geometric` for GCN feature extraction as configured in your notebook).

## What’s inside the experiments

- Baselines (tabular features + SMOTE):
  - Logistic Regression
  - Random Forest
  - XGBoost

- Embeddings:
  - GCN embeddings (graph convolutional features)
  - Node2Vec embeddings

- Downstream models on embeddings:
  - XGBoost
  - Random Forest
  - SVM
- Metrics emphasized:
  - Binary precision/recall/F1 for the phishing class
  - Accuracy and macro averages
  - PR-AUC and ROC-AUC
  - Training time

Why PR-AUC and phishing-class F1?
- Under severe class imbalance, accuracy and even ROC-AUC can be misleadingly high. PR-AUC and phishing-class F1 directly capture utility for the minority class.

## Results

See `results.md` for full tables and observations.

High-level summary from current runs:
- Baseline (tabular + SMOTE): Random Forest reached phishing F1≈0.94, XGBoost F1≈0.88, Logistic Regression F1≈0.72 on the ~9.8k-row tabular dataset.
- Embeddings + models (graph sample ~54.9k nodes): GCN embeddings + XGBoost achieved phishing F1≈0.63 and the best PR-AUC in that setting; Node2Vec was weaker in recall/F1 for the phishing class.
- Interpreting “Did embeddings help?”
  - On this setup, the tabular baseline (with heavy preprocessing + SMOTE) produced higher phishing-class F1 than the first-pass embedding pipelines on a different sample/label balance.
  - Embeddings remain valuable for incorporating graph structure at scale; further gains typically require careful sampling, feature normalization, threshold tuning, and class-aware training.

## Notes

- The results in `results.md` are transcribed from the final executed cells of the notebooks.
- If you re-run the notebook with different random seeds, sampling strategies, or thresholds, the numbers will vary slightly.
- For production use, consider threshold tuning based on PR curves to better match your precision/recall requirements.
