---
name: tandem-ms2-structure-prediction
description: Use when generating or ranking de novo molecular structures from positive-mode,
  low-resolution MS2/MSn spectra (e.g. from a miniaturized linear ion trap mass spectrometer),
  especially for unknown compounds not covered by spectral library search.
when_to_use_negative:
- Spectra recorded in negative ion mode — TeFT was only trained on positive ion mode
  (H+) spectra, restricting its applicability
- 'High-resolution, high-accuracy spectra from large instruments, where models like
  SIRIUS4 (CSI: FingerID), MSNovelist or MassGenie may outperform TeFT'
- Compounds with spectral library matches that can be identified by standard mass
  spectrometry library search
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: TeFT
  role: 'Core model: combines fragmentation tree models and a Transformer module to
    generate structures de novo from MS2 spectra'
  repo: thumingo/TeFT
- name: SIRIUS4
  role: Baseline tool for comparative benchmarking of structure prediction accuracy
    (with Wilcoxon signed-rank tests)
- name: MetFrag
  role: Baseline database/fragment-prediction tool for comparative benchmarking against
    TeFT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:1cda4925cdfcc44fe1c9d707cdab2e8bc7d263d1440e80765f5fb81ac81cb750
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - MetFrag
  - SIRIUS
  - SIRIUS4
  - TeFT
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# tandem-ms2-structure-prediction

## Summary

De novo prediction of complete molecular structures from low-resolution tandem mass spectrometry (MS2/MSn) spectra using TeFT, which combines fragmentation tree models with a Transformer module trained on GNPS, HMDB 5.0 and MoNA. Designed for annotating unknown small molecules on miniaturized mass spectrometers.

## When to use

Use when generating or ranking de novo molecular structures from positive-mode, low-resolution MS2/MSn spectra (e.g. from a miniaturized linear ion trap mass spectrometer), especially for unknown compounds not covered by spectral library search.

## When NOT to use

- Spectra recorded in negative ion mode — TeFT was only trained on positive ion mode (H+) spectra, restricting its applicability
- High-resolution, high-accuracy spectra from large instruments, where models like SIRIUS4 (CSI: FingerID), MSNovelist or MassGenie may outperform TeFT
- Compounds with spectral library matches that can be identified by standard mass spectrometry library search

## Inputs

- Positive-ion-mode (H+) tandem MS2/MSn spectra (peak lists with ion intensities)
- Candidate structure lists (SMILES) for each unknown compound
- Training spectra compiled from open-source databases GNPS, HMDB 5.0 and MoNA

## Outputs

- De novo predicted molecular structures as SMILES strings
- SMILES fragment trees for each predicted molecule
- Ranked candidate substance list based on fragment-tree similarity
- Similarity scores (Dice/Tanimoto) between predicted and ground-truth structures

## How to apply

Run the TeFT model on each positive-mode MS2 spectrum to generate predicted SMILES structures, applying the established constraints: molecular weight <500 Da, only the ten elements C/H/O/N/P/S/Cl/Br/I/F, a maximum of 100 peaks by ion intensity, and the top five to six fragmentation peaks (with five nearby peak points) as experimental data. Deconvolute and average replicate spectra for error correction before prediction, and keep SMILES fragment trees within 1500 nodes. Rank candidate substances by comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from the tandem mass spectra. Validate performance by computing Dice/Tanimoto similarity against ground-truth structures, reporting the fraction ranked fully correct, mostly correct, and >0.8 or >0.9 similarity; benchmark against SIRIUS4 and MetFrag with Wilcoxon signed-rank tests (TeFT reached 12% fully correct on 93 CASMI 2017 spectra, 78% similarity >0.8, with p-values 4.4×10⁻⁶ vs SIRIUS4 and 2.7×10⁻¹¹ vs MetFrag).

## Related tools

- **TeFT** (Core model: combines fragmentation tree models and a Transformer module to generate structures de novo from MS2 spectra) — thumingo/TeFT
- **SIRIUS4** (Baseline tool for comparative benchmarking of structure prediction accuracy (with Wilcoxon signed-rank tests))
- **MetFrag** (Baseline database/fragment-prediction tool for comparative benchmarking against TeFT)

## Evaluation signals

- Percentage of test spectra for which the correct structure is ranked first (TeFT: 12% fully correct, 22% mostly correct on 93 CASMI 2017 spectra)
- Fraction of predictions with Dice similarity >0.8 and >0.9 versus ground-truth structures (TeFT: 78% and 39%)
- Wilcoxon signed-rank p-values on similarity distributions vs SIRIUS4 (4.4×10⁻⁶) and MetFrag (2.7×10⁻¹¹)
- Predicted SMILES fragment trees stay within the 1500-node limit and respect the <500 Da and ten-element constraints
- On the miniaturized-instrument flavonoid test, complete structure prediction should be achieved for roughly half of compounds (8 of 16 flavonoid alcohols)

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting applicability to negative-mode data
- TeFT may not exhibit exceptional performance advantages on high-quality resolution spectra compared to other deep models (e.g. MassGenie, MSNovelist)
- A 1500-node limit is imposed on the total count of SMILES tree nodes; large or unusual molecules may exceed model scope
- Performance depends on fragmentation peaks with clear fragmentation rules; the full rule list is in Supplementary Table S1

## Evidence

- [abstract] TeFT validated on low-resolution miniaturized MS: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [readme] TeFT combines fragmentation trees and Transformer: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
- [discussion] Candidate ranking by SMILES fragment tree similarity: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry"
- [discussion] Molecular weight constraint: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] Peak selection for experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
