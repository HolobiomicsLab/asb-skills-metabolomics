---
name: consensus-spectrum-dereplication
description: Use when clustered MS2 spectra from LC–MS/MS samples must be dereplicated
  into non-redundant consensus spectra while keeping isomers separated, e.g. NP3 MS
  Workflow Cleaning steps 3–5 on per-sample MGF input.
when_to_use_negative:
- Raw vendor LC–MS/MS files that have not passed NP3 preprocessing (step 2) — per-sample
  MGF MS2 spectra and MS1 feature data are required first.
- Targeted MS/MS library search where the spectra are already individual standards
  rather than redundant replicate clusters needing dereplication.
- Datasets where isomer separation is not needed and a full spectral-library-style
  merge of all similar spectra would be acceptable.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Executes Clustering, Quantification and Cleaning (steps 3–5) that dereplicate
    MS2 spectra into clean consensus spectra via the NP3-shifted cosine
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: File format used for per-sample MS2 input and clean consensus spectrum output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:5a319c74134b1952e88424e96a13e48a9328a4c2b93f02e7ede175502d12903c
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
yes_rate: 1.0
n_claims: 10
---

# consensus-spectrum-dereplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Iteratively merge redundant MS2 spectra from clustered LC–MS/MS features into clean consensus spectra using the NP3-shifted cosine (greedy modified cosine), then export one MGF per ion plus a clean count table for downstream library identification and bioactivity correlation.

## When to use

Use when clustered MS2 spectra from LC–MS/MS samples must be dereplicated into non-redundant consensus spectra while keeping isomers separated, e.g. NP3 MS Workflow Cleaning steps 3–5 on per-sample MGF input.

## When NOT to use

- Raw vendor LC–MS/MS files that have not passed NP3 preprocessing (step 2) — per-sample MGF MS2 spectra and MS1 feature data are required first.
- Targeted MS/MS library search where the spectra are already individual standards rather than redundant replicate clusters needing dereplication.
- Datasets where isomer separation is not needed and a full spectral-library-style merge of all similar spectra would be acceptable.

## Inputs

- Per-sample MGF MS2 files produced by NP3 preprocessing (step 2)
- Metadata Table describing LC–MS/MS samples and sample groups, including blanks
- Preprocessed MS1/MS2 feature table with peak intensity, peak area and MS1 information

## Outputs

- Clean consensus spectra representatives as an MGF file
- Per-feature quantification as a clean count table
- Dereplicated set of isomer-distinct consensus spectra (e.g. 11 isomers of m/z 589.31 retained separately)

## How to apply

Run NP3 MS Workflow Clustering, Quantification, and Cleaning (steps 3–5): after step 3 clustering apply the absolute noise_cutoff — MS2 spectra whose base peak intensity falls below the minimum are removed before Cleaning. In Cleaning, compare spectra with the NP3-shifted cosine function, a greedy version of the modified cosine, to merge redundant spectra of the same ion; the greedy merging keeps isomers (e.g. 11 distinct isomers of m/z 589.31) separate while collapsing duplicate spectra of the same precursor. Iterate clustering rounds until convergence or the maximum number of rounds; if the maximum is reached and the count table is removed, skip the removal on the last round (fixed in v1.5.1). Exclude m/z ions detected in blank or culture-media samples, then export clean consensus spectrum representatives as MGF and per-feature quantification as a clean count table.

## Related tools

- **NP3 MS Workflow** (Executes Clustering, Quantification and Cleaning (steps 3–5) that dereplicate MS2 spectra into clean consensus spectra via the NP3-shifted cosine) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (File format used for per-sample MS2 input and clean consensus spectrum output)

## Examples

```
np3-ms-workflow run --input MSV000080502 --metadata metadata.csv --noise_cutoff <abs_bpi> --ion_mode positive
```

## Evaluation signals

- Duplicated consensus spectra rate should be near the reported 1.5% for case study #1
- All 11 distinct isomers of m/z 589.31 must be retained as separate clean consensus spectra
- No consensus spectrum with base peak intensity below the absolute noise_cutoff survives Cleaning
- m/z ions detected in blank or culture-media samples must be absent from the clean count table
- Convergence or the maximum number of clustering rounds is reached before final export

## Limitations

- Blank/culture-media removal depends on blank samples being present and correctly annotated in the Metadata Table; without blanks m/z from culture media persist in the [M+H]+ networks.
- The noise_cutoff is an absolute base peak intensity (no normalization), so threshold choice depends on instrument sensitivity and can drop minority compounds if set too high.
- If the maximum number of clustering rounds is reached without convergence, spectra may remain redundant; versions before 1.5.1 had a bug that removed the count table before the last round's access.

## Evidence
