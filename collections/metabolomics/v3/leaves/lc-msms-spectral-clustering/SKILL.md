---
name: lc-msms-spectral-clustering
description: Use when you have per-sample MGF MS2 spectra from LC–MS/MS feature detection
  (e.g. MSV000080502) and need to group them into clean consensus spectra, merging
  duplicates of the same ion yet keeping distinct isomers of the same m/z (e.g. 11
  isomers of m/z 589.31) separated.
when_to_use_negative:
- Input is already a clean consensus spectra MGF or a curated feature/quantification
  table rather than raw per-sample MS2 spectra needing clustering
- The data set was not preprocessed by NP³ (no per-sample MGF, no MS1–MS2 correspondence
  table), so the clustering input format is absent
- The analysis goal is library search/identification of already-clean consensus spectra
  (Step 6, Tremolo) rather than grouping/merging raw MS2 spectra
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Implements the Clustering, Quantification, and Cleaning steps (3–5) using
    the NP3-shifted cosine and absolute noise_cutoff to produce clean consensus spectra
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: File format for the per-sample MS2 spectra input and the clean consensus spectra
    output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:48e4ad04e71e537d6727aed3ecaa937b6117100b10c8857ed3231326add0380e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# LC–MS/MS Spectral Clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters preprocessed MS2 spectra from LC–MS/MS untargeted metabolomics into consensus spectra using spectral similarity, peak intensity, peak area, and MS1 information, then merges redundant spectra of the same ion (NP3-shifted cosine) while retaining isomers as separate clusters.

## When to use

Use when you have per-sample MGF MS2 spectra from LC–MS/MS feature detection (e.g. MSV000080502) and need to group them into clean consensus spectra, merging duplicates of the same ion yet keeping distinct isomers of the same m/z (e.g. 11 isomers of m/z 589.31) separated.

## When NOT to use

- Input is already a clean consensus spectra MGF or a curated feature/quantification table rather than raw per-sample MS2 spectra needing clustering
- The data set was not preprocessed by NP³ (no per-sample MGF, no MS1–MS2 correspondence table), so the clustering input format is absent
- The analysis goal is library search/identification of already-clean consensus spectra (Step 6, Tremolo) rather than grouping/merging raw MS2 spectra

## Inputs

- Per-sample MGF (Mascot Generic Format) MS2 spectra files from NP³ pre-processing (step 2)
- Metadata Table describing the LC–MS/MS samples and sample groups, including blank and culture-media samples
- MS1 information (detected LC–MS features / MS1 peaks per sample)

## Outputs

- Clean consensus spectrum representatives exported as an MGF file
- Per-feature quantification as the clean count table
- IVAMN (ionization variant annotation molecular network) components derived from the clean consensus spectra

## How to apply

Run Steps 3–5 of the NP³ MS Workflow (Clustering, Quantification, Cleaning) on the per-sample MGF files produced by pre-processing. Cluster preprocessed MS2 data relying on spectral similarity plus peak intensity, peak area, and MS1 information; apply the absolute noise_cutoff after Step 3 clustering so that MS2 spectra whose minimum base peak int (without normalization) falls below the threshold are removed before the clean step. In Cleaning, compare spectra with the NP3-shifted cosine function (a greedy version of the modified cosine) and iterate clustering rounds until the maximum number of rounds or convergence, merging redundant spectra of the same ion while keeping isomers separated. Exclude m/z ions detected in blank or culture-media samples from the result. Export the clean consensus spectrum representatives as an MGF file and the per-feature quantification as the clean count table.

## Related tools

- **NP3 MS Workflow** (Implements the Clustering, Quantification, and Cleaning steps (3–5) using the NP3-shifted cosine and absolute noise_cutoff to produce clean consensus spectra) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (File format for the per-sample MS2 spectra input and the clean consensus spectra output)

## Examples

```
nextflow run danielatrivella/NP3_MS_Workflow --input MSV000080502.mgf --metadata metadata.tsv --noise_cutoff 1000
```

## Evaluation signals

- Duplicated consensus spectra rate should be near the reported 1.5% for case study #1
- All 11 distinct isomers of m/z 589.31 must be retained as separate clean consensus spectra
- MS2 spectra below the absolute noise_cutoff (minimum base peak int without normalization) must be removed before the clean step
- m/z ions present in blank or culture-media samples must be excluded from the clean count table
- Clustering iterations must stop either at the maximum number of rounds or at convergence

## Limitations

- Clustering relies on NP3-shifted cosine similarity; spectra of the same ion may still be merged or split incorrectly if similarity thresholds are not tuned
- Maximum number of clustering rounds caps the merging process — if convergence is not reached, some redundant spectra remain unmerged
- Requires blank/culture-media metadata to exclude background m/z; without it, contaminant ions persist in the clean count table

## Evidence

- [methods] Merging while keeping isomers separated: [evidence span withheld — anchor status: not_found]
