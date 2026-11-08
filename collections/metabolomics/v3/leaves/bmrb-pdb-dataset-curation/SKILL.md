---
name: bmrb-pdb-dataset-curation
description: Use when building a chemical shift prediction training or benchmark set
  that requires experimentally assigned shifts paired with 3D structures, e.g. harvesting
  matching BMRB/PDB entries and filtering to non-paramagnetic, standard-residue proteins.
when_to_use_negative:
- The protein of interest is paramagnetic or contains non-standard residues — the
  curated training corpus explicitly excluded these, so the trained models are not
  supported by such data.
- You need a ready-made evaluation set and the ShiftX2 61-protein test dataset suffices
  — reuse it directly instead of re-curation.
- Curating RNA chemical shift data — the RNA predictor uses a separate dataset (371
  RNA molecules with BMRB assignments and PDB structures) and FM models trained by
  stochastic gradient descent, a distinct workflow.
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0092
tools:
- name: NMRFx
  role: Consumes the curated BMRB/PDB datasets to train and evaluate its protein chemical
    shift prediction model; benchmark results (RMSD/MAE per atom type) are reported
    in Table S1 of the SI
  repo: https://github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:49bca4fa974342165bb41c5cc1e3f7b216d429955da2be648cd4ddc2a3892473
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  repo_url: https://github.com/nanalysis/nmrfx
  tools:
  - NMRFx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bmrb-pdb-dataset-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles matched protein training/test datasets for chemical shift prediction by pairing experimental chemical shift assignments from BMRB with 3D structures from PDB, filtering to non-paramagnetic proteins with standard residues. This curation underpins training and benchmarking of NMRFx's protein shift prediction model.

## When to use

Use when building a chemical shift prediction training or benchmark set that requires experimentally assigned shifts paired with 3D structures, e.g. harvesting matching BMRB/PDB entries and filtering to non-paramagnetic, standard-residue proteins.

## When NOT to use

- The protein of interest is paramagnetic or contains non-standard residues — the curated training corpus explicitly excluded these, so the trained models are not supported by such data.
- You need a ready-made evaluation set and the ShiftX2 61-protein test dataset suffices — reuse it directly instead of re-curation.
- Curating RNA chemical shift data — the RNA predictor uses a separate dataset (371 RNA molecules with BMRB assignments and PDB structures) and FM models trained by stochastic gradient descent, a distinct workflow.

## Inputs

- BMRB chemical shift assignment entries (experimental assigned shifts for proteins)
- PDB 3D structure entries associated with the BMRB records

## Outputs

- Training dataset of matched BMRB/PDB proteins (1480 non-paramagnetic, standard-residue proteins in the NMRFx work)
- Test dataset of 61 proteins from the ShiftX2 benchmark, disjoint from training data

## How to apply

For each candidate protein, retrieve the experimental chemical shift assignment list from BMRB and the corresponding 3D structure from PDB, keeping only entries where the two records match. Apply the inclusion filters used in NMRFx: the protein must be non-paramagnetic and composed of standard residues; the NMRFx work used 1480 such proteins for training. For testing, reuse an established external benchmark rather than resampling — the paper evaluates on the 61-protein ShiftX2 test dataset ('a dataset of 61 proteins previously used for ShiftX2'). Split the curated data strictly so that training proteins and benchmark test proteins are disjoint, then pass the curated sets to per-amino-acid/per-atom-class regression training (the article documents atom classes per residue, e.g. glutamine attribute tables in Table S6 of the SI).

## Related tools

- **NMRFx** (Consumes the curated BMRB/PDB datasets to train and evaluate its protein chemical shift prediction model; benchmark results (RMSD/MAE per atom type) are reported in Table S1 of the SI) — https://github.com/nanalysis/nmrfx

## Evaluation signals

- Every retained protein has both a BMRB assignment entry and a matching PDB structure entry; unmatched records are dropped.
- Filtered dataset size is consistent with the paper's corpus of 1480 non-paramagnetic, standard-residue proteins (when reproducing the full corpus).
- Training set and the 61-protein ShiftX2 test set are disjoint (no protein appears in both).
- Downstream per-atom-type RMSD/MAE metrics on the test set are consistent in magnitude with Table S1 of the SI.
- Per-residue atom-class coverage is complete, mirroring the attribute classes documented for glutamine in Table S6 of the SI.

## Limitations

- Training corpus excludes paramagnetic proteins and non-standard residues, so models trained this way should not be expected to predict shifts for such systems.
- Exact per-atom-type RMSD/MAE values are deferred to Table S1 of the SI and are not reported in the main text, limiting direct value-matching from the article alone.
- Match quality depends on correct BMRB↔PDB pairing; the article does not describe automated disambiguation of multiple PDB entries per BMRB record.

## Evidence

- [discussion] To train the protein shift prediction model, a dataset of 1480 non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries was used.: "To train the protein shift prediction model, a dataset of 1480 non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries was used."
- [discussion] ShiftX2 test dataset: "For testing, a dataset of 61 proteins previously used for ShiftX2 was used53."
- [discussion] SI evaluation tables: "The RMSD and MAE values of the predicted shift values versus the experimentally assigned values for the 61 protein test dataset are listed in Table S1 of the SI."
- [abstract] NMRFx chemical shift prediction capability: "NMRFx facilitates data processing, peak picking and assignment, chemical shift prediction, molecular structure calculation, and beyond, through a high-speed, feature-rich graphical user interface."
