---
name: tandem-mass-spectrometry-spectrum-processing
description: Use when raw tandem MS (MS2/MSn) spectra—especially low-resolution spectra
  from a miniaturized mass spectrometer—must be converted into candidate molecular
  structures and scored by fingerprint Tanimoto similarity against known SMILES.
when_to_use_negative:
- Spectra were recorded in negative ion mode—TeFT was trained only on positive ion
  mode (H+) spectra, so applicability is restricted
- High-resolution, high-mass-accuracy spectra from large instruments—models like MassGenie
  or SIRIUS4 are better suited for high-quality resolution spectra
- Compounds outside the <500 Da / ten-element (C,H,O,N,P,S,Cl,Br,I,F) chemical constraint
  space
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Transformer enabled Fragment Tree (TeFT)
  role: Core model combining fragmentation tree simulation with a Transformer module
    to de novo generate the complete candidate structure from MS2 spectra
  repo: thumingo/TeFT
- name: SIRIUS4
  role: Baseline database-search/CSI:FingerID comparator benchmarked against TeFT
    on exact and >0.9 Tanimoto match rates
- name: Transformer
  role: Deep learning module that generates the complete SMILES structure of the target
    substance from the MS2 spectrum
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:8b03ca2c8d99c1a4347e882534d86ef4933c7d6b3b0fb9d8f2e915b5bed33d9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - SIRIUS
  - SIRIUS4
  - Transformer
  - Transformer enabled Fragment Tree (TeFT)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# tandem-mass-spectrometry-spectrum-processing

## Summary

Process low-resolution MS2 spectra from miniaturized mass spectrometers and generate de novo candidate molecular structures using the TeFT (Transformer enabled Fragment Tree) pipeline, scoring predictions against reference SMILES with MACCS/Morgan fingerprint Tanimoto similarity.

## When to use

Use when raw tandem MS (MS2/MSn) spectra—especially low-resolution spectra from a miniaturized mass spectrometer—must be converted into candidate molecular structures and scored by fingerprint Tanimoto similarity against known SMILES.

## When NOT to use

- Spectra were recorded in negative ion mode—TeFT was trained only on positive ion mode (H+) spectra, so applicability is restricted
- High-resolution, high-mass-accuracy spectra from large instruments—models like MassGenie or SIRIUS4 are better suited for high-quality resolution spectra
- Compounds outside the <500 Da / ten-element (C,H,O,N,P,S,Cl,Br,I,F) chemical constraint space

## Inputs

- Low-resolution MS2/MSn tandem mass spectra (positive ion mode, H+ precursor)
- Peak lists limited to ≤100 peaks ordered by ion intensity
- Reference SMILES structures for database compounds (GNPS, HMDB 5.0, MoNA)

## Outputs

- Best-candidate molecular structure (SMILES) per input spectrum
- MACCS-key and Morgan fingerprint Tanimoto similarity scores per spectrum
- Aggregate results table (results_table.csv) with counts at Tanimoto 1, >0.9, >0.8

## How to apply

Compile or load MS2 spectra (e.g., 660 non-repetitive test spectra drawn from GNPS, HMDB 5.0, and MoNA), standardize SMILES representations, and remove duplicate entries. Preprocess each spectrum by selecting the top five to six fragmentation peaks (with their five nearby peak points) and truncating the peak list to a maximum of 100 peaks ranked by ion intensity; apply error correction via multiple-measurement averaging and deconvolution integration when spectra are noisy. Constrain the search space by restricting molecular weight to <500 Da, permitting only the ten elements C, H, O, N, P, S, Cl, Br, I, F, limiting the SMILES tree to 1500 nodes, and narrowing molecular formulas by element kinds and counts. Run the TeFT model (fragmentation tree simulation combined with a Transformer structure-generation module) to produce the best candidate structure per spectrum, then compute MACCS-key and Morgan fingerprint Tanimoto similarities between predicted and true structures and report counts at Tanimoto = 1, >0.9, and >0.8 thresholds.

## Related tools

- **Transformer enabled Fragment Tree (TeFT)** (Core model combining fragmentation tree simulation with a Transformer module to de novo generate the complete candidate structure from MS2 spectra) — thumingo/TeFT
- **SIRIUS4** (Baseline database-search/CSI:FingerID comparator benchmarked against TeFT on exact and >0.9 Tanimoto match rates)
- **Transformer** (Deep learning module that generates the complete SMILES structure of the target substance from the MS2 spectrum)

## Evaluation signals

- On the 660-spectrum GNPS/HMDB/MoNA test set, reproduce 195 exact matches (Tanimoto = 1, 30%), 311 candidates above 0.9 similarity (47%), and 439 above 0.8 similarity (67%)
- TeFT should exceed SIRIUS4 baselines of 27.6% exact and 39% >0.9 Tanimoto matches on comparable test spectra
- Every predicted and reference structure converts to valid, standardized SMILES before fingerprint comparison
- Candidate rankings are consistent with fragmentation-tree similarity ordering between predicted SMILES fragment trees and spectra-derived fragment trees

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting its applicability to negative-ion data
- The model may not outperform competing deep models on high-resolution, high-quality spectra
- Applicable mainly to compounds with clear fragmentation rules; validated for complete structure prediction on only 8 of 16 flavonoid alcohols on a miniaturized instrument
- Large-scale Transformer training (à la MassGenie) is impractical for on-site miniaturized mass spectrometry platforms

## Evidence

- [abstract] the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [discussion] The training set utilized for the Transformer model was created by compiling open-source spectrometry databases GNPS, HMDB (5.0), and MoNA: "The training set utilized for the Transformer model was created by compiling open-source spectrometry databases GNPS, HMDB (5.0), and MoNA"
- [discussion] we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module. It is aimed to generate the specific structure of molecules de novo solely from"
