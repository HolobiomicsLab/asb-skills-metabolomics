---
name: teft-structure-prediction-inference
description: Use when low-resolution MS2 spectra from a miniaturized ion trap instrument
  must be annotated with de novo molecular structures (SMILES) without library matching,
  especially for unknown natural products or drugs on-site.
when_to_use_negative:
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode
  (H+) spectra, restricting its applicability
- 'High-resolution, high-mass-accuracy spectra from full-size instruments, where methods
  like SIRIUS4 (CSI: FingerID), MSNovelist or MassGenie are applicable'
- 'Substances outside the constraints: molecular weight ≥500 Da, elements beyond C,
  H, O, N, P, S, Cl, Br, I, F, or SMILES trees exceeding 1500 nodes'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Transformer enabled Fragment Tree (TeFT)
  role: Core model combining fragmentation tree models with a Transformer module to
    generate de novo SMILES from MS2 spectra
  repo: https://github.com/thumingo/TeFT
- name: Transformer
  role: Generates the complete target-substance structure from MS2 spectra, predicting
    and extending SMILES fragment strings
  repo: https://github.com/thumingo/TeFT
- name: GNPS
  role: Open-source spectrometry database used to compile the Transformer model training
    set
- name: HMDB (5.0)
  role: Open-source spectrometry database used to compile the training set
- name: MoNA
  role: Open-source spectrometry database used to compile the training set
- name: miniaturized mass spectrometer
  role: Custom linear ion trap instrument providing the low-resolution MS2 spectra
    input for on-site prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:2ad35954283577979193d9d281b7353725e7deb5131be7a32cf1f729df87989c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - GNPS
  - HMDB (5.0)
  - MoNA
  - Transformer
  - Transformer enabled Fragment Tree (TeFT)
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# teft-structure-prediction-inference

## Summary

Run TeFT (Transformer enabled Fragment Tree) to generate de novo SMILES structures from low-resolution MS2 spectra acquired on a miniaturized mass spectrometer, then rank candidates by SMILES fragment tree similarity.

## When to use

Use when low-resolution MS2 spectra from a miniaturized ion trap instrument must be annotated with de novo molecular structures (SMILES) without library matching, especially for unknown natural products or drugs on-site.

## When NOT to use

- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode (H+) spectra, restricting its applicability
- High-resolution, high-mass-accuracy spectra from full-size instruments, where methods like SIRIUS4 (CSI: FingerID), MSNovelist or MassGenie are applicable
- Substances outside the constraints: molecular weight ≥500 Da, elements beyond C, H, O, N, P, S, Cl, Br, I, F, or SMILES trees exceeding 1500 nodes

## Inputs

- Low-resolution MS2 spectra (MSn) from a miniaturized linear ion trap mass spectrometer
- Peak lists (top 5-6 fragmentation peaks plus five nearby peak points, max 100 peaks by ion intensity)
- Training spectra from GNPS, HMDB 5.0 and MoNA (positive ion mode H+) for model fitting

## Outputs

- De novo SMILES candidate structures per substance
- SMILES fragment trees from simulated molecular fragmentation
- Per-substance fragment-tree similarity scores and ranked candidates
- Per-class prediction outcomes (e.g. table2_results.csv with substance class, prediction outcome and similarity score)

## How to apply

Preprocess the MS2 spectrogram by selecting the top five to six fragmentation peaks with their five nearby peak points, capping the peak list at 100 peaks by ion intensity, and applying multiple-measurement averaging plus deconvolution integration for error correction. Feed the spectra into the TeFT model — a Transformer module combined with fragmentation tree models trained on GNPS, HMDB (5.0) and MoNA in positive ion mode (H+) — under candidate constraints of molecular weight below 500 Da, only the ten elements C, H, O, N, P, S, Cl, Br, I, F, and a 1500-node limit on SMILES tree nodes. TeFT generates candidate SMILES structures; the model predicts and extends SMILES fragment strings while preserving fragment-structure integrity. Simulate molecular fragmentation of each predicted molecule and compute similarity between its SMILES fragment tree and the fragment tree generated from the tandem mass spectrum, ranking candidate substances by this similarity. Classify each substance as fully predicted or not by the similarity score and aggregate counts by compound class (e.g. the 23-substance test yielded 8/16 flavonoid alcohols, 1/4 stilbenes, 1/2 flavones fully predicted with minimum similarity 0.93).

## Related tools

- **Transformer enabled Fragment Tree (TeFT)** (Core model combining fragmentation tree models with a Transformer module to generate de novo SMILES from MS2 spectra) — https://github.com/thumingo/TeFT
- **Transformer** (Generates the complete target-substance structure from MS2 spectra, predicting and extending SMILES fragment strings) — https://github.com/thumingo/TeFT
- **GNPS** (Open-source spectrometry database used to compile the Transformer model training set)
- **HMDB (5.0)** (Open-source spectrometry database used to compile the training set)
- **MoNA** (Open-source spectrometry database used to compile the training set)
- **miniaturized mass spectrometer** (Custom linear ion trap instrument providing the low-resolution MS2 spectra input for on-site prediction)

## Evaluation signals

- Per-substance fragment-tree similarity scores are computed and candidates ranked; fully predicted substances should reach high similarity (worst fully-predicted similarity was 0.93 in the 23-substance test)
- Class-level counts reproduce Table 2: 8 of 16 flavonoid alcohols, 1 of 4 stilbenes and 1 of 2 flavones fully predicted
- Generated SMILES are valid and standardized, consistent with training-set standardization of SMILES representations with duplicate removal
- Predicted molecules pass the constraints: MW <500 Da, only C/H/O/N/P/S/Cl/Br/I/F present, SMILES tree ≤1500 nodes
- Peak lists respect preprocessing limits: max 100 peaks by ion intensity, top 5-6 fragmentation peaks with five nearby peak points

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting applicability to negative-mode data
- The model may not exhibit exceptional performance advantages over other deep models when predicting high-quality resolution spectra
- Fragmentation and rearrangement rules used are only listed in Supplementary Table S1 without versioning or public availability details
- Only a minimal README exists for thumingo/TeFT; no code availability statement or changelog is provided in the article

## Evidence

- [abstract] we propose the Transformer enabled Fragment Tree (TeFT) method, which combines various types of fragmentation tree models and a deep learning Transformer module: "we propose the Transformer enabled Fragment Tree (TeFT) method, which combines various types of fragmentation tree models and a deep learning Transformer module"
- [discussion] simulating molecular fragmentation based on deep learning predictions and scoring the similarity with fragmentation trees: "simulating molecular fragmentation based on deep learning predictions and scoring the similarity with fragmentation trees"
- [discussion] the training set standardized the SMILES representations for all molecules and removed duplicate entries: "the training set standardized the SMILES representations for all molecules and removed duplicate entries"
- [readme] It is aimed to generate the specific structure of molecules de novo solely from mass spectrometry spectra: "It is aimed to generate the specific structure of molecules de novo solely from mass spectrometry spectra"
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
