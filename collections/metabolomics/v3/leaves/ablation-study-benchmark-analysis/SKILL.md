---
name: ablation-study-benchmark-analysis
description: 'Use when a multi-component deep learning prediction pipeline must be
  attributed to its parts: train ablation variants (e.g., with/without Gaussian-noise
  augmentation or contrastive loss) and measure which component drives accuracy gains
  on a held-out test split.'
when_to_use_negative:
- The goal is end-to-end benchmarking against external baselines (SIRIUS, BUDDY) rather
  than attributing performance to the pipeline's own components
- Working with Orbitrap data, where no data augmentation is applied (noise addition
  lowered similarity for Orbitrap spectra), so a DA-ablation arm is not meaningful
- The model is a single-component predictor with no post-processing or secondary scoring
  stage to ablate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning pipeline whose components (TCN predictor, contrastive loss,
    candidate generation, rescore model) are ablated and evaluated on Q-TOF spectra
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type defining the data domain (10 ppm precursor tolerance, Gaussian-noise
    augmentation regime) for the ablation experiments
- name: BUDDY
  role: Optional external tool; not part of the ablation itself but installable alongside
    the FIDDLE codebase
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:9797604c77cc832c78bba86a173cbaf827a0d057c9921c8adf15ad725f8aaa08
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - Q-TOF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# ablation-study-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce component-wise ablation of a deep learning pipeline (FIDDLE on MoNA Q-TOF MS/MS) by training model variants with individual components removed (data augmentation, contrastive loss, post-processing) and quantifying each component's contribution via top-1/top-5 accuracy, missed-atom distributions, candidate coverage, and ROC AUC of confidence scores.

## When to use

Use when a multi-component deep learning prediction pipeline must be attributed to its parts: train ablation variants (e.g., with/without Gaussian-noise augmentation or contrastive loss) and measure which component drives accuracy gains on a held-out test split.

## When NOT to use

- The goal is end-to-end benchmarking against external baselines (SIRIUS, BUDDY) rather than attributing performance to the pipeline's own components
- Working with Orbitrap data, where no data augmentation is applied (noise addition lowered similarity for Orbitrap spectra), so a DA-ablation arm is not meaningful
- The model is a single-component predictor with no post-processing or secondary scoring stage to ablate

## Inputs

- MoNA Q-TOF MS/MS spectra converted to MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- Q-TOF config files under ./config/ for the TCN ablation variants
- Pre-trained checkpoints fiddle_tcn_qtof.pt and fiddle_rescore_qtof.pt (reference/full model)

## Outputs

- Missed-atom histograms for top-1 predictions before and after post-processing, per variant
- Top-1 and top-5 accuracy per ablation variant (w/o DA, w/o CL, full Pred)
- Top-5 candidate coverage of the correct formula after post-processing
- ROC curve and ROC AUC of the confidence-score ranking of candidate formulas

## How to apply

Set up the environment (conda env from environment.yml) and fetch pre-trained Q-TOF weights (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) via download_models.sh for reference. Convert MoNA Q-TOF MS/MS spectra to MGF format with title, precursor_mz, precursor_type, and collision_energy fields, applying the standard filters (>=5 peaks, m/z in (0,1500], peaks below 50 m/z removed, 10 ppm precursor tolerance). Train three TCN variants on the Q-TOF config files: 'Pred w/o DA' (no Gaussian noise, mean 0, sd 0.1, applied to intensities only in the full model), 'Pred w/o CL' (no contrastive-learning objective), and the full 'Pred' model, using train_tcn_gpus.py / train_tcn_gpus_cl.py, then train the secondary confidence-scoring model per variant with train_rescore.py. Evaluate each variant on the held-out test split — split by molecular canonical SMILES to prevent compound leakage — running running_scripts/experiments_ablation_study.sh, which uses run_fiddle.py with breadth-first refinement, SENIOR-rule post-processing, and ranking by rescore confidence. Record top-1 accuracy and missed-atom histograms (before and after post-processing), top-5 candidate coverage of the correct formula, and ROC AUC of confidence scores distinguishing correct from incorrect formulas.

## Related tools

- **FIDDLE** (Deep learning pipeline whose components (TCN predictor, contrastive loss, candidate generation, rescore model) are ablated and evaluated on Q-TOF spectra) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type defining the data domain (10 ppm precursor tolerance, Gaussian-noise augmentation regime) for the ablation experiments)
- **BUDDY** (Optional external tool; not part of the ablation itself but installable alongside the FIDDLE codebase) — https://github.com/Philipbear/msbuddy

## Examples

```
conda env create -f environment.yml && bash running_scripts/download_models.sh && bash running_scripts/experiments_ablation_study.sh
```

## Evaluation signals

- Post-processing top-5 candidate coverage of the correct formula exceeds 84% on the MoNA Q-TOF evaluation
- Confidence-score ranking achieves an ROC AUC of ~0.97 on the Q-TOF test split, matching Fig. 4a-d
- Data augmentation raises correctly predicted top-1 formulas from 0% to 9% (all atoms) and 18% to 26% (heavy atoms), with contrastive loss further improving performance
- Missed-atom histograms show the post-processing mass-alignment effect (breadth-first refinement aligns candidate formulas to precursor mass)
- Train/test split by molecular canonical SMILES shows no shared compounds between training and test sets

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data, so ablation results may not transfer to low-data regimes
- No data augmentation was applied for Orbitrap training since noise addition led to lower similarity for Orbitrap spectra; DA-ablation findings are Q-TOF-specific
- FIDDLE performance degrades on chimeric (co-eluting) spectra as spectral mixing increases

## Evidence

- [results] Data augmentation increases correctly predicted top-1 formulas: "we added random noise sampled from a Gaussian distribution with a mean of 0 and a standard deviation of 0.1"
- [discussion] SMILES-based split avoids leakage: "is first split into training and test sets according to their molecular canonical SMILES, which ensures that there are no common compounds between the training set and the test set"
- [discussion] Post-processing refines with SENIOR rules: "ranking them by predicted confidence scores, aim to refine the predicted formulas using the SENIOR rules"
- [discussion] Mass tolerance setting per instrument: "The tolerance for precursor mass discrepancy is set at 10 ppm for Q-TOF and 5 ppm for Orbitrap instruments"
- [readme] Environment setup from README: "conda env create -f environment.yml"
- [discussion] Orbitrap augmentation caveat: "Since the training size for Orbitrap is sufficiently large, we did not apply data augmentation for training a deep learning model for Orbitrap"
