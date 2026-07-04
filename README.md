# Molecular Property Prediction

Master's thesis project: **"Prediction of molecular properties through the combination of NLP features, molecular descriptors, and fingerprints"** (Faculty of Electrical Engineering and Computing, University of Zagreb).

Awarded the **GDi Excellence Award** and presented at the **10th Symposium of Chemistry Students**.

---

## Overview

This project explores how combining different molecular representations affects the quality of molecular property prediction. The core idea is that fusing descriptors derived from different modalities (in this case physicochemical descriptors, topological fingerprints, and NLP-based latent embeddings) into a joint representation improves prediction performance compared to any single representation alone.

## Notebooks

| Notebook | Description |
|---|---|
| [`molecular_property_prediction_dopamine.ipynb`](molecular_property_prediction_dopamine.ipynb) | **Original thesis notebook, unchanged.** Dopamine receptor dataset (479 molecules, pIC50 values). Full pipeline from SMILES retrieval to model comparison, including the stacking-ensemble fusion. |
| [`molecular_property_prediction_lipophilicity.ipynb`](molecular_property_prediction_lipophilicity.ipynb) | **Reworked pipeline on the MolNet Lipophilicity dataset (~4,200 molecules).** Adds Bemis-Murcko scaffold splitting + GroupKFold CV (to prevent leakage between structurally similar molecules), 2D+3D Mordred descriptors, and MLM-fine-tuned ChemBERTa — then re-adds the fused/combined representation (the thesis's core contribution) on top of this leakage-safe methodology. |
| [`molecular_property_prediction_wip.ipynb`](molecular_property_prediction_wip.ipynb) | Currently identical to the lipophilicity notebook above (historical name from when it was the working/consolidation copy). |

## Pipeline (lipophilicity notebook)

1. **Data preprocessing** — MolNet Lipophilicity via DeepChem, Bemis-Murcko scaffold split, SMILES canonicalization
2. **Molecular descriptors** — Mordred 2D and 3D, cleaned (train-only column/row rules) and reduced via variance/correlation filtering + RFECV
3. **Molecular fingerprints** — MACCS, ECFP/Morgan, ErG pharmacophore (SVD-reduced)
4. **Latent embeddings** — Mol2Vec (pretrained) and ChemBERTa (both frozen-pretrained and MLM-fine-tuned on the training split)
5. **Per-representation evaluation** — each representation evaluated independently via 5-fold scaffold CV (GroupKFold on Murcko scaffolds), so structurally similar molecules can't leak across folds
6. **Feature fusion** — all representations concatenated (joined on SMILES, since Mordred cleaning can drop different molecules in 2D vs 3D) into one feature matrix, then evaluated under the *same* per-fold selection + scaffold CV protocol as the individual representations, so the fused-vs-single comparison is apples-to-apples
7. **Fine-tuned ChemBERTa** is evaluated on the official train/val/test split only (not full-dataset CV), since its MLM fine-tuning corpus was the training split's own SMILES — including it in a CV that reassigns that data to validation folds would leak information. A supplementary fused+fine-tuned-ChemBERTa variant is included, evaluated on the held-out test split only.

## Key Results

**Dopamine notebook (original thesis, unchanged):** combining molecular representations via early fusion (concatenation) and a Ridge stacking ensemble consistently improved test performance over any individual representation. Best test R² was **0.46** (SVM + combined representations) on this 479-molecule dataset, though a dataset this small means all models overfit substantially — treat this result as indicative rather than conclusive.

**Lipophilicity notebook:** the per-representation numbers below come from leakage-safe 5-fold scaffold CV; the fused-representation section is new code (see Pipeline step 6 above) and needs to be run once to populate real numbers here — no results are reported until it has actually executed, to avoid claiming figures that weren't produced by the code as committed.

## Stack

Python · PyTorch · scikit-learn · RDKit · Mordred · Mol2Vec · HuggingFace Transformers (ChemBERTa) · DeepChem
