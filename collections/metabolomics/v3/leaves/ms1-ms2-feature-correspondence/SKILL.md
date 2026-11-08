---
name: ms1-ms2-feature-correspondence
description: Use when processing raw LC–MS/MS runs where per-sample MS1 peaks must
  be linked to their acquired MS2 spectra before any clustering, spectral merging,
  or bioactivity correlation — the article's Preprocessing step extracts MS2 spectra
  and determines MS1–MS2 correspondence per sample.
when_to_use_negative:
- Data has already been preprocessed and per-sample MGF files exist — skip to Clustering/Quantification/Cleaning
  (steps 3–5) instead
- You only need post-processing visualization or drug discovery reports (post_dd_analysis),
  which do not require re-determining MS1–MS2 correspondence
- The goal is joining prior job results (join_jobs), which runs Steps 3–10 and does
  not redo step 2
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Performs the Preprocessing step that detects MS1 peaks, extracts MS2 spectra,
    and determines MS1–MS2 correspondence before writing per-sample MGF files
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: Per-sample output format carrying extracted MS2 spectra linked to detected
    MS1 features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:07128e1b1dd957afba8086a32e630dfb268931098fe453fd971031bcf6ebce5f
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ms1-ms2-feature-correspondence

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines which detected MS1 LC–MS features correspond to which MS2 spectra during preprocessing of LC–MS/MS untargeted metabolomics data in the NP3 MS Workflow, so downstream clustering, quantification, and bioactivity scoring operate on properly paired features.

## When to use

Use when processing raw LC–MS/MS runs where per-sample MS1 peaks must be linked to their acquired MS2 spectra before any clustering, spectral merging, or bioactivity correlation — the article's Preprocessing step extracts MS2 spectra and determines MS1–MS2 correspondence per sample.

## When NOT to use

- Data has already been preprocessed and per-sample MGF files exist — skip to Clustering/Quantification/Cleaning (steps 3–5) instead
- You only need post-processing visualization or drug discovery reports (post_dd_analysis), which do not require re-determining MS1–MS2 correspondence
- The goal is joining prior job results (join_jobs), which runs Steps 3–10 and does not redo step 2

## Inputs

- Raw LC–MS/MS instrument data (e.g. from a H-Class UPLC coupled to an Impac II qQToF, positive mode)
- Metadata Table describing LC–MS/MS samples, sample groups, and blank samples
- Optional prior partial pre-processing results (MS1_no_MS2 table to be incremented and cleaned)

## Outputs

- Per-sample MS2 Mascot Generic Format (MGF) files with MS1–MS2 feature correspondence resolved
- MS1 peak/feature table per sample for use in Clustering, Quantification, and Cleaning
- Pre-process statistics report

## How to apply

Run NP3 MS Workflow's Preprocessing (step 2) on raw LC–MS/MS data after supplying a Metadata Table (step 1) describing samples and sample groups, including blanks. The step detects LC–MS features (MS1 peaks) for each sample, extracts their MS2 spectra, and determines MS1–MS2 correspondence, then writes per-sample MGF files carrying the feature assignments. Retain the MS1_no_MS2 bookkeeping when reusing prior preprocessing results (a partial pre-processing bug was fixed in v1.4.1, so this table must be correctly incremented and cleaned across runs). Rationale: downstream Clustering, Quantification, and Cleaning (steps 3–5) rely on spectral similarity plus peak intensity, peak area, and MS1 information, so misassigned MS1–MS2 pairs propagate errors into consensus spectra, the clean count table, and blank-filtered BCC rankings.

## Related tools

- **NP3 MS Workflow** (Performs the Preprocessing step that detects MS1 peaks, extracts MS2 spectra, and determines MS1–MS2 correspondence before writing per-sample MGF files) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (Per-sample output format carrying extracted MS2 spectra linked to detected MS1 features)

## Examples

```
nextflow run main.nf --input MSV000080502/ --metadata metadata.csv --pre_process
```

## Evaluation signals

- Each detected MS1 feature has exactly one corresponding MS2 spectrum (or is recorded in the MS1_no_MS2 table) in the per-sample MGF
- Re-running preprocessing on the same raw data reproduces identical MS1–MS2 assignments (workflow includes equality tests to guarantee reproducibility)
- MS1_no_MS2 table is correctly incremented and cleaned when reusing partial pre-processing results, not duplicated or lost (bug fixed in v1.4.1)
- Downstream Step 3 clustering and Step 5 cleaning produce no spectra whose MS1 feature is absent from the input feature table
- Count table precursor-ion intensities and areas trace back to the resolved MS1–MS2 pairs, not to unmatched MS2 orphan spectra

## Limitations

- Only the first workflow step (Metadata Table) requires user intervention; MS1–MS2 correspondence is automated and its thresholds are not exposed for manual tuning in the README
- In partial pre-processing reuse, the MS1_no_MS2 table was historically mishandled (fixed in v1.4.1) — earlier versions may produce incorrect correspondences
- Cannot rescue MS2 spectra for MS1 features with no acquired MS2; these are tracked separately as MS1_no_MS2

## Evidence

- [methods] Preprocessing detects MS1 peaks and determines MS1–MS2 correspondence: [evidence span withheld — anchor status: not_found]
