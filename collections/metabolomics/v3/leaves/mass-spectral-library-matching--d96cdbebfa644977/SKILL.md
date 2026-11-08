---
name: mass-spectral-library-matching
description: Use when a structurally characterized Mass2Motif (or other fragment/loss
  set) needs putative annotation or validation against curated MS/MS spectral libraries
  before transferring annotations to new datasets.
when_to_use_negative:
- The motif set contains dominant neutral losses that are central to the annotation
  — these must be dropped for matching, so the pseudo-spectrum loses key diagnostic
  information.
- The dataset is not an information-rich, complex mixture with ramped or stepped collision
  energy fragmentation — MS2LDA decomposition quality drops and the motif pseudo-spectra
  may be unreliable.
- You need fully confirmed structural IDs rather than putative substructure annotations
  — matching only supports putative de novo annotation, not definitive structure assignment.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: Generates and structurally characterizes the Mass2Motifs whose pseudo-spectra
    are matched against libraries; also decomposes external datasets with beer-characterized
    motifs kept fixed during LDA inference
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: GNPS
  role: Reference MS/MS spectral library queried by spectral matching to validate
    motif-derived substructure annotations
- name: Mass Bank
  role: Reference spectral library providing independent validation of motif annotations
- name: NIST MS/MS database
  role: Additional small-molecule MS/MS reference library used for spectral matching
    in the workflow
- name: MS2LDAvis
  role: Visualization of molecular networks produced alongside motif matching results
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:b2f61e020f6d34fea1c132dbe0ec6df17116c1025767425fda7cf35c708584b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - GNPS
  - MS2LDA
  - MS2LDAvis
  - Mass Bank
  - MassIVE
  - NIST MS/MS database
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- spectral-library-matching
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mass-spectral-library-matching

## Summary

Validate Mass2Motif substructure annotations by representing each motif as a pseudo-spectrum (its fragments and losses as peaks) and matching it against external MS/MS reference libraries such as GNPS, Mass Bank, and NIST at a 90% normalized score threshold.

## When to use

Use when a structurally characterized Mass2Motif (or other fragment/loss set) needs putative annotation or validation against curated MS/MS spectral libraries before transferring annotations to new datasets.

## When NOT to use

- The motif set contains dominant neutral losses that are central to the annotation — these must be dropped for matching, so the pseudo-spectrum loses key diagnostic information.
- The dataset is not an information-rich, complex mixture with ramped or stepped collision energy fragmentation — MS2LDA decomposition quality drops and the motif pseudo-spectra may be unreliable.
- You need fully confirmed structural IDs rather than putative substructure annotations — matching only supports putative de novo annotation, not definitive structure assignment.

## Inputs

- Structurally characterized Mass2Motifs (fragments and neutral losses with associated probabilities)
- External reference MS/MS spectra from GNPS, Mass Bank, and NIST MS/MS database
- MS/MS spectra from the dataset under analysis (e.g., Beer3 positive-ionization mode file, human urine sample)

## Outputs

- Per-motif validation table (CSV) with matched library, hit score, and matched reference for each Mass2Motif
- Proportion of validated substructure annotations at a chosen probability threshold (e.g., 81.5% Mass Bank / 63.3% GNPS at 0.1)
- Automatically assigned substructure annotations for spectra in the analyzed dataset

## How to apply

Represent each structurally characterized Mass2Motif as a pseudo-spectrum in which its co-occurring molecular fragments (retained at a probability threshold of 0.1) act as peaks; neutral losses must be omitted because current spectral matching tools do not support them. Load the reference spectra — in this study GNPS and Mass Bank spectra used in the CSI:FingerID paper, plus the NIST MS/MS database for small molecules — and perform spectral matching of each motif pseudo-spectrum against the libraries. Apply a 90% normalized score threshold to accept hits, the same cutoff the paper uses for spectral library matching of regular spectra. Record per-motif match status (matched library, hit score, matched reference) in a validation table. Judge success by the validation benchmark: at a probability threshold of 0.1, 81.5% of Mass Bank and 63.3% of GNPS substructure annotations were validated, with 694 and 613 spectra respectively carrying at least one validated substructure annotation.

## Related tools

- **MS2LDA** (Generates and structurally characterizes the Mass2Motifs whose pseudo-spectra are matched against libraries; also decomposes external datasets with beer-characterized motifs kept fixed during LDA inference) — https://github.com/vdhooftcompmet/MS2LDA
- **GNPS** (Reference MS/MS spectral library queried by spectral matching to validate motif-derived substructure annotations)
- **Mass Bank** (Reference spectral library providing independent validation of motif annotations)
- **NIST MS/MS database** (Additional small-molecule MS/MS reference library used for spectral matching in the workflow)
- **MS2LDAvis** (Visualization of molecular networks produced alongside motif matching results) — https://github.com/vdhooftcompmet/MS2LDA

## Examples

```
pip install ms2lda
```

## Evaluation signals

- At probability threshold 0.1, ≥80% of motif substructure annotations validated against Mass Bank and ≥60% against GNPS (paper benchmarks: 81.5% and 63.3%).
- 694 Mass Bank and 613 GNPS reference spectra carry at least one validated substructure annotation.
- Every accepted hit must exceed the 90% normalized score threshold.
- Inferred novel motifs' annotations are absent from MotifDB, confirming the validation targets are the beer-characterized fixed motifs.
- Validation table records a matched library, hit score, and matched reference for each characterized positive-mode beer Mass2Motif.

## Limitations

- Neutral losses cannot be included in spectral matching because losses are not supported by current spectral matching tools, so motif pseudo-spectra are fragment-only.
- Validation was only performed on the 19 correctly annotated positive-mode beer Mass2Motifs; other ionization modes or sample matrices were not benchmarked.
- The matching workflow is computationally more costly than simpler tools, though not prohibitively so.
- Only spectra matched at ≥90% normalized score are annotated, leaving the majority of spectra (e.g., only 25% and 6% of spectra obtain hits for some datasets) without library annotation.
- MS2LDA works best for complex mixtures with information-rich MS2 spectra (ramped or stepped collision energy); sparse spectra degrade motif quality.

## Evidence

- [discussion] works best for complex mixtures where a large number of metabolites are fragmented and information-rich MS2 spec-tra are available (e.g., generated by ramped or stepped collision energy): [evidence span withheld — anchor status: not_found]
