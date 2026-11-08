---
name: contrastive-learning-ablation
description: Use when reproducing or adapting the FIDDLE ablation study to measure
  how spectrum noise augmentation, contrastive loss, and post-processing (candidate
  generation + rescore ranking) each affect top-1/top-5 formula accuracy, missed-atom
  distributions, and confidence-score ROC AUC.
when_to_use_negative:
- 'Working with Orbitrap spectra: the paper notes no data augmentation was applied
  for Orbitrap training because noise addition lowered spectral similarity, so the
  DA ablation arm does not transfer'
- Goal is end-to-end formula prediction rather than attribution of performance to
  training objectives or post-processing components
- Input spectra are chimeric (co-eluting) or fall outside the training distribution,
  where FIDDLE's accuracy degrades and ablation deltas would be confounded
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning framework whose TCN, contrastive-learning, and rescore components
    are ablated and evaluated
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type of the MoNA evaluation spectra used for the ablation study
- name: BUDDY
  role: Optional external formula-annotation tool installable alongside FIDDLE, not
    part of the ablation arms
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:74f52dc42c5835b228fe5f85bc2f067fd84032ec2358770ddbad756fd5ed0ebf
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# contrastive-learning-ablation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ablation methodology for quantifying the individual contributions of data augmentation (Gaussian noise, mean 0, sd 0.1), the contrastive-learning objective, and the post-processing steps (candidate formula generation plus confidence-score ranking) to FIDDLE's formula identification performance on the MoNA Q-TOF dataset.

## When to use

Use when reproducing or adapting the FIDDLE ablation study to measure how spectrum noise augmentation, contrastive loss, and post-processing (candidate generation + rescore ranking) each affect top-1/top-5 formula accuracy, missed-atom distributions, and confidence-score ROC AUC.

## When NOT to use

- Working with Orbitrap spectra: the paper notes no data augmentation was applied for Orbitrap training because noise addition lowered spectral similarity, so the DA ablation arm does not transfer
- Goal is end-to-end formula prediction rather than attribution of performance to training objectives or post-processing components
- Input spectra are chimeric (co-eluting) or fall outside the training distribution, where FIDDLE's accuracy degrades and ablation deltas would be confounded

## Inputs

- MoNA Q-TOF MS/MS spectra converted to MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- Q-TOF config files from ./config/ for TCN and contrastive-learning training
- Pre-trained weights fiddle_tcn_qtof.pt and fiddle_rescore_qtof.pt
- Held-out Q-TOF test split partitioned by molecular canonical SMILES

## Outputs

- Missed-atom histograms for top-1 predictions before and after post-processing, per ablation variant
- Top-1 and top-5 accuracy per variant ('Pred w/o DA', 'Pred w/o CL', full 'Pred')
- ROC curve and AUC of the rescore model's confidence scores distinguishing correct from incorrect formulas
- Top-5 candidate coverage of the correct formula after post-processing

## How to apply

Create the conda environment from environment.yml and fetch the Q-TOF weights (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) via running_scripts/download_models.sh. Convert MoNA Q-TOF MS/MS spectra to MGF (title, precursor_mz, precursor_type, collision_energy fields), applying the standard filters (>=5 peaks, m/z in (0, 1500], peaks below 50 m/z removed, 10 ppm precursor mass tolerance). Train three TCN variants with train_tcn_gpus.py and train_tcn_gpus_cl.py using the ./config/ Q-TOF configs: 'Pred w/o DA' (no Gaussian noise on intensities), 'Pred w/o CL' (no contrastive objective), and the full 'Pred' model; then train a rescore model per variant with train_rescore.py. Evaluate each variant on the held-out Q-TOF test split (split by molecular canonical SMILES to avoid compound leakage) with running_scripts/experiments_ablation_study.sh, using run_fiddle.py with breadth-first refinement, SENIOR-rule post-processing, and ranking by the rescore model's confidence scores. The rationale is that data augmentation alone lifted correctly predicted top-1 formulas from 0% to 9% (all atoms) and 18% to 26% (heavy atoms), contrastive loss further improved performance, and confidence-score ranking reaches ROC AUC ~0.97.

## Related tools

- **FIDDLE** (Deep learning framework whose TCN, contrastive-learning, and rescore components are ablated and evaluated) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type of the MoNA evaluation spectra used for the ablation study)
- **BUDDY** (Optional external formula-annotation tool installable alongside FIDDLE, not part of the ablation arms) — https://github.com/Philipbear/msbuddy

## Examples

```
bash running_scripts/experiments_ablation_study.sh
```

## Evaluation signals

- Post-processing top-5 candidate coverage of the correct formula exceeds 84% on the MoNA Q-TOF evaluation (Fig. 4a-d)
- Confidence-score ROC AUC reaches ~0.97 for distinguishing correct from incorrect formulas
- Removing data augmentation ('Pred w/o DA') drops correctly predicted top-1 formulas from 9% to 0% (all atoms) and from 26% to 18% (heavy atoms)
- Missed-atom histograms show the post-processing mass-alignment effect relative to pre-post-processing predictions
- Test set contains no compounds shared with training, verified by canonical-SMILES splitting

## Limitations

- Experimental-condition effects are not fully eliminated; accuracy varies across conditions, especially those with less training data
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- Data augmentation is Q-TOF specific: no augmentation was used for Orbitrap since noise addition led to lower similarity

## Evidence
