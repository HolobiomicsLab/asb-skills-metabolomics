---
name: isomer-resolution-in-spectral-clustering
description: Use when clustering preprocessed LC–MS/MS MS2 spectra (e.g. per-sample
  MGF files from step 2) where co-eluting structural isomers of the same precursor
  m/z must not be merged into a single consensus spectrum.
when_to_use_negative:
- Input is already a set of clean consensus spectra or an MGF library file — re-running
  isomer-resolving clustering would duplicate work rather than de-replicate.
- Goal is only to merge redundant spectra of the same ion with no structural-isomer
  ambiguity (e.g. single-peak features with no co-eluting isomers) — the iterative
  NP3-shifted-cosine rounds are unnecessary.
- You need de novo structural annotation of individual isomers — this skill only separates
  isomeric consensus spectra; structural assignment requires downstream Library Identification
  (e.g. Tremolo against ISDB/GNPS).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Executes Clustering, Quantification, and Cleaning (steps 3–5) with the NP3-shifted
    cosine similarity to resolve isomers into separate clean consensus spectra.
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: File format for per-sample MS2 input spectra and for the exported clean consensus
    spectra representatives.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:1022678b00ab6b41d86d60a34dfa2a8f822dcae8c44302d9aab40a95d2cf1608
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# isomer-resolution-in-spectral-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolves structural isomers during LC–MS/MS consensus spectrum construction by iteratively re-clustering preprocessed MS2 spectra with the NP3-shifted cosine (a greedy version of the modified cosine) inside the NP3 MS Workflow's Clustering/Quantification/Cleaning steps, so that isomeric ions of identical m/z remain separate clean consensus spectra.

## When to use

Use when clustering preprocessed LC–MS/MS MS2 spectra (e.g. per-sample MGF files from step 2) where co-eluting structural isomers of the same precursor m/z must not be merged into a single consensus spectrum.

## When NOT to use

- Input is already a set of clean consensus spectra or an MGF library file — re-running isomer-resolving clustering would duplicate work rather than de-replicate.
- Goal is only to merge redundant spectra of the same ion with no structural-isomer ambiguity (e.g. single-peak features with no co-eluting isomers) — the iterative NP3-shifted-cosine rounds are unnecessary.
- You need de novo structural annotation of individual isomers — this skill only separates isomeric consensus spectra; structural assignment requires downstream Library Identification (e.g. Tremolo against ISDB/GNPS).

## Inputs

- Per-sample MS2 spectra in Mascot Generic Format (MGF) produced by NP3 MS Workflow Preprocessing (step 2)
- Absolute noise_cutoff value (minimum base peak intensity, unnormalized)
- Metadata Table describing LC–MS/MS samples and sample groups (including blanks)

## Outputs

- Clean consensus spectra representatives exported as an MGF file
- Per-feature quantification exported as a clean count table
- Isomer-resolved consensus spectra (e.g. all 11 distinct isomers of m/z 589.31 retained as separate clean consensus spectra)

## How to apply

Run the NP3 MS Workflow Clustering, Quantification, and Cleaning steps (3–5) on the per-sample MGF files produced by the Preprocessing step, which cluster preprocessed MS2 data relying on spectral similarity, peak intensity, peak area, and MS1 information. Apply the absolute noise_cutoff (minimum base peak intensity, no normalization) after Step 3 clustering so MS2 spectra below it are removed before Cleaning. In Cleaning, compare spectra with the NP3-shifted cosine function — a greedy version of the modified cosine — and iterate clustering rounds until convergence or the maximum number of rounds is reached; this merges redundant spectra of the same ion while keeping isomers separated. Finally export the clean consensus spectra representatives as an MGF file and the per-feature quantification as a clean count table. If the maximum number of clustering rounds is reached in step 5, skip the count table removal so the join is not corrupted (README bug-fix note for v1.5.1).

## Related tools

- **NP3 MS Workflow** (Executes Clustering, Quantification, and Cleaning (steps 3–5) with the NP3-shifted cosine similarity to resolve isomers into separate clean consensus spectra.) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (File format for per-sample MS2 input spectra and for the exported clean consensus spectra representatives.)

## Examples

```
nextflow run danielatrivella/NP3_MS_Workflow --input msv000080502_samples.tsv --noise_cutoff <abs_int> --steps clustering,quantification,cleaning --ion_mode positive
```

## Evaluation signals

- Duplicated consensus spectra rate should be near the reported ~1.5% for case study #1.
- All distinct isomers of a shared precursor m/z (e.g. the 11 isomers of m/z 589.31) must appear as separate clean consensus spectra, not collapsed into one.
- MS2 spectra with base peak intensity below the absolute noise_cutoff are absent from the final clean MGF (removed before Cleaning).
- If the maximum number of clustering rounds is reached, no count-table access-after-removal error occurs (v1.5.1 regression test).
- Clean count table row/column counts align with the clean MGF consensus spectra list.

## Limitations

- Iterative clustering rounds may hit the maximum round limit before convergence; the pipeline then stops rather than guaranteeing a converged clustering.
- Isomer separation depends on spectral similarity behavior of the NP3-shifted cosine; isomers with near-identical fragment patterns may still be merged.
- The noise_cutoff is an absolute base-peak intensity threshold (unnormalized), so results are sensitive to raw instrument sensitivity and cannot be tuned as a relative cutoff.

## Evidence
