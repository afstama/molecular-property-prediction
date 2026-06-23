# Molecular Property Prediction

Master's thesis project — **"Prediction of molecular properties through the combination of NLP features, molecular descriptors, and fingerprints"** (University of Zagreb, Faculty of Electrical Engineering and Computing).

Awarded the **GDi Excellence Award** and presented at the **10th Symposium of Chemistry Students**.

---

## Overview

This project explores how combining different molecular representations affects the quality of molecular property prediction. The core idea is that fusing descriptors derived from different modalities — physicochemical descriptors, topological fingerprints, and NLP-based latent embeddings — into a joint representation improves prediction performance compared to any single representation alone.

## Notebooks

| Notebook | Description |
|---|---|
| [`molecular_property_prediction_dopamine.ipynb`](molecular_property_prediction_dopamine.ipynb) | **Main thesis notebook.** Dopamine receptor dataset (479 molecules, pIC50 values). Full pipeline from SMILES retrieval to model comparison. |
| [`molecular_property_prediction_lipophilicity.ipynb`](molecular_property_prediction_lipophilicity.ipynb) | Same pipeline applied to the **MolNet Lipophilicity dataset (~4,200 molecules)**. Larger dataset for more robust evaluation. |
| [`molecular_property_prediction_wip.ipynb`](molecular_property_prediction_wip.ipynb) | ⚠️ Work in progress — experimental extension exploring additional modelling strategies. |

## Pipeline

1. **Data preprocessing** — SMILES retrieval from ChEMBL, canonicalization, pIC50 scaling
2. **Molecular descriptors** — Mordred (1826 → ~23 after selection via RFECV)
3. **Molecular fingerprints** — MACCS, Avalon, ECFP, ErG (reduced to 128 dims via SVD)
4. **Latent embeddings** — Mol2Vec and ChemBERTa (fine-tuned, reduced to 128 dims via autoencoder)
5. **Model comparison** — RF, SVM, HGBR trained on individual and combined representations
6. **Stacking ensemble** — Ridge meta-model over per-representation base models

## Key Results

Combined representations achieve R² of **0.93–0.94**, an improvement of **8–15%** over the best individual representation. A stacking ensemble over per-representation models consistently outperforms direct concatenation.

## Stack

Python · PyTorch · scikit-learn · RDKit · Mordred · Mol2Vec · HuggingFace Transformers (ChemBERTa) · DeepChem
