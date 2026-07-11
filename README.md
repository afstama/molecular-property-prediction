# Molecular Property Prediction

Master's thesis project: **"Prediction of molecular properties through the combination of NLP features, molecular descriptors, and fingerprints"** (Faculty of Electrical Engineering and Computing, University of Zagreb).

Awarded the **GDi Excellence Award** and presented at the **10th Symposium of Chemistry Students**.

---

## Overview

This project explores how combining different molecular representations affects the quality of molecular property prediction. The core idea is that fusing descriptors derived from different modalities (in this case physicochemical descriptors, topological fingerprints, and NLP-based latent embeddings) into a joint representation improves prediction performance compared to any single representation alone.

## Notebooks

| Notebook | Description |
|---|---|
| [`molecular_property_prediction_dopamine.ipynb`](molecular_property_prediction_dopamine.ipynb) | **Original thesis notebook – _dopamine_ dataset.** Dopamine receptor dataset (479 molecules, pIC50 values). Full pipeline from SMILES retrieval to model comparison, including the stacking-ensemble fusion. |
| [`molecular_property_prediction_lipophilicity.ipynb`](molecular_property_prediction_lipophilicity.ipynb) | **Original thesis notebook – _lipophilicity_ dataset.** MolNet Lipophilicity dataset (~4,200 molecules). Same pipeline as above, run on a larger dataset. Notebook text is in Croatian, the thesis's original language. |
| [`molecular_property_prediction_lipophilicity_wip.ipynb`](molecular_property_prediction_lipophilicity_wip.ipynb) | **Reworked pipeline on the MolNet Lipophilicity dataset (~4,200 molecules).** Adds Bemis-Murcko scaffold splitting + GroupKFold CV (to prevent leakage between structurally similar molecules), drops 3D Mordred descriptors (shown not to help), adds MLM-fine-tuned ChemBERTa, and re-tests the fused/combined representation (the thesis's core contribution) under this leakage-safe methodology. Work in progress. |

## Pipeline

Both original thesis notebooks (dopamine, lipophilicity) follow the same sequence:

1. **Data loading & cleaning** — SMILES are retrieved (via the ChEMBL API for the dopamine target, via DeepChem's MolNet loader for lipophilicity — loaded with a scaffold splitter but then randomly re-split 80:20, since the leakage-safe split was only introduced later in the `_wip` notebook), validated, and canonicalized. No missing values in either dataset. The target property is scaled to [0, 1] with min-max scaling fit on the training set.
2. **Molecular representations** are computed per molecule:
   - *Descriptors*: ~1,826 Mordred descriptors, cleaned (invalid/near-constant columns dropped, variance threshold 0.16), then reduced via autocorrelation removal, Random Forest importance, Pearson-correlation pruning (>0.9), and RFECV — down to 23 features (dopamine) / 28 features (lipophilicity).
   - *Fingerprints*: ECFP/Morgan (512 bits) and ErG (Gobbi Pharm2D, ~40,000 bits) are each reduced independently (near-constant removal + TruncatedSVD) to 64 dimensions and evaluated as two separate representations. (MACCS and Avalon fingerprints are mentioned as generated in the notebook text but aren't actually used by the downstream models.)
   - *Mol2Vec*: pretrained 300-dim embeddings, reduced to 64 dimensions with an autoencoder.
   - *ChemBERTa*: `ChemBERTa-zinc-base-v1`, additionally fine-tuned on the training set's target values (20 epochs, Adam), then its embeddings are reduced to 64 dimensions the same way as Mol2Vec.
3. **Modeling** — features are standardized; Random Forest, SVM, and HistGradientBoostingRegressor are each hyperparameter-tuned (Grid Search for RF/SVM, Randomized Search for HGBR) with 3-fold CV on the training set, then evaluated on: each of the five representations above individually, a direct concatenation of all five ("combined"), and a stacking ensemble (Ridge meta-model over the five base models' predictions). Metrics: MSE, MAE, R² on train and test.

The reworked lipophilicity notebook (`_wip`) regenerates representations with a different fingerprint/modeling approach (MACCS, ECFP, and ErG each evaluated individually via SGDRegressor) and replaces the modeling methodology with scaffold-aware splitting/CV throughout, closing a data-leakage gap in the original pipeline (structurally similar molecules could otherwise land in both train and test).

## Key Results

### Dopamine (479 molecules, 120 held out for testing)

Test-set R² by representation and model (from the notebook's own final comparison, § 4. Conclusion):

| Representation | RF | SVM | HGBR |
|---|---|---|---|
| Descriptors | 0.336 | 0.388 | 0.347 |
| ECFP | 0.365 | 0.435 | 0.425 |
| ErG | 0.447 | 0.457 | **0.487** |
| Mol2Vec | 0.274 | 0.369 | 0.290 |
| ChemBERTa | 0.275 | 0.341 | 0.272 |
| Combined (concatenation) | 0.364 | 0.433 | 0.416 |
| Stacking ensemble | 0.431 | **0.487** | 0.471 |

The best result (R²=0.487) is a tie between the SVM stacking ensemble and the ErG fingerprint alone under HGBR — fusion (specifically stacking, not naive concatenation) matches the best individual representation rather than clearly beating it. The ErG pharmacophore fingerprint is unexpectedly the strongest single representation, outperforming Mordred descriptors across all three models. The latent/NLP representations (Mol2Vec, ChemBERTa) are consistently the weakest individually. The large training/test R² gap (0.93–0.99 vs. 0.27–0.49) is attributed in the notebook to the small test set (120 molecules) rather than a fundamental model failure — these results are indicative rather than conclusive at this dataset size.

### Lipophilicity — original thesis pipeline (~4,200 molecules)

Test-set R² by representation and model (from the notebook's own final comparison):

| Representation | RF | SVM | HGBR |
|---|---|---|---|
| Descriptors | 0.579 | 0.650 | 0.589 |
| ECFP | 0.294 | 0.521 | 0.363 |
| ErG | 0.417 | 0.547 | 0.451 |
| Mol2Vec | 0.331 | 0.506 | 0.341 |
| ChemBERTa | 0.270 | 0.253 | 0.236 |
| Combined (concatenation) | 0.550 | **0.710** | 0.604 |
| Stacking ensemble | 0.610 | 0.692 | 0.622 |

At this larger scale, fusion wins clearly for SVM and HGBR — the best result overall is SVM with direct concatenation (R²=0.710), just ahead of the SVM stacking ensemble (R²=0.692). Random Forest is the one partial exception: plain descriptors (R²=0.579) beat direct concatenation (R²=0.550), though the RF stacking ensemble (R²=0.610) still edges out descriptors alone — so fusion still helps for RF, just only via stacking, not naive concatenation. ChemBERTa is a notably weaker individual representation here than for dopamine (R²=0.24–0.27 across all three models). (The original notebook's "4. Zaključak" conclusion section was left as a header with no written text; the summary above is derived directly from its results tables.)

### Reworked lipophilicity pipeline (leakage-safe, `_wip` notebook)

Under scaffold-aware splitting and cross-validation — which prevents structurally similar molecules from leaking between train/val/test — the picture changes: fusing all representations (2D Mordred descriptors, fingerprints, Mol2Vec, and pretrained ChemBERTa) gives essentially no improvement over 2D Mordred descriptors alone (5-fold scaffold CV R²: 0.634 fused vs. 0.631 descriptors-only). 3D Mordred descriptors and MLM-fine-tuned ChemBERTa were also tested and found not to help. This suggests the original thesis's "fusion helps" result may partly reflect leakage from random (non-scaffold) splitting rather than a robust effect — see the notebook for the full per-representation breakdown.

## Stack

Python · PyTorch · scikit-learn · RDKit · Mordred · Mol2Vec · HuggingFace Transformers (ChemBERTa) · DeepChem
