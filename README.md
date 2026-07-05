# Molecular Property Prediction

Master's thesis project: **"Prediction of molecular properties through the combination of NLP features, molecular descriptors, and fingerprints"** (Faculty of Electrical Engineering and Computing, University of Zagreb).

Awarded the **GDi Excellence Award** and presented at the **10th Symposium of Chemistry Students**.

---

## Overview

This project explores how combining different molecular representations affects the quality of molecular property prediction. The core idea is that fusing descriptors derived from different modalities (in this case physicochemical descriptors, topological fingerprints, and NLP-based latent embeddings) into a joint representation improves prediction performance compared to any single representation alone.

## Notebooks

| Notebook | Description |
|---|---|
| [`molecular_property_prediction_dopamine.ipynb`](molecular_property_prediction_dopamine.ipynb) | **Original thesis notebook - _dopamine_ dataset.** Dopamine receptor dataset (479 molecules, pIC50 values). Full pipeline from SMILES retrieval to model comparison, including the stacking-ensemble fusion. |
| [`molecular_property_prediction_dopamine.ipynb`](molecular_property_prediction_lipophilicity.ipynb) | **Original thesis notebook - _lipophilicity_ dataset.** Lipophilicity receptor dataset (~4,200 molecules). Original thesis pipeline on a larger dataset. |
| [`molecular_property_prediction_lipophilicity.ipynb`](molecular_property_prediction_lipophilicity_wip.ipynb) | **Reworked pipeline on the MolNet Lipophilicity dataset (~4,200 molecules).** Adds Bemis-Murcko scaffold splitting + GroupKFold CV (to prevent leakage between structurally similar molecules), removes 3D Mordred descriptors, adds MLM-fine-tuned ChemBERTa, and re-adds the fused/combined representation (the thesis's core contribution) on top of this leakage-safe methodology. Work in progress. |

## Pipeline

TODO: add pipeline steps with explanation

## Key Results

TODO: add thesis notebooks results

## Stack

Python · PyTorch · scikit-learn · RDKit · Mordred · Mol2Vec · HuggingFace Transformers (ChemBERTa) · DeepChem
