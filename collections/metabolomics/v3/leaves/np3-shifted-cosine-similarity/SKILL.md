---
name: np3-shifted-cosine-similarity
description: Use when clustering or de-replicating MS2 consensus spectra from LC–MS/MS
  untargeted metabolomics (e.g. per-sample MGFs from NP3 preprocessing) so that redundant
  spectra of the same ion merge while isomers stay separate.
when_to_use_negative:
- The input is already a finalized clean consensus MGF — applying the cosine merge
  again would over-merge spectra
- The goal is library search or spectral matching against GNPS2/ISDB libraries (that
  is step 6, which uses Tremolo, not the Cleaning cosine merge)
- Spectra come from a different pipeline without NP3's noise_cutoff and MGF layout,
  since the similarity comparison assumes NP3 preprocessed MS2 features
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Runs steps 3–5 (Clustering, Quantification, Cleaning) where the NP3-shifted
    cosine drives the spectral-similarity clustering
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:b87ba904f70b7da41d42c1113bda51dd3682fb7d38a6e8c5e5e80305b75c8653
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# np3-shifted-cosine-similarity

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A greedy version of the modified cosine used inside the NP3 MS Workflow's Cleaning step to compare MS2 spectra, merge redundant spectra of the same ion, and keep distinct isomers separated during clustering.

## When to use

Use when clustering or de-replicating MS2 consensus spectra from LC–MS/MS untargeted metabolomics (e.g. per-sample MGFs from NP3 preprocessing) so that redundant spectra of the same ion merge while isomers stay separate.

## When NOT to use

- The input is already a finalized clean consensus MGF — applying the cosine merge again would over-merge spectra
- The goal is library search or spectral matching against GNPS2/ISDB libraries (that is step 6, which uses Tremolo, not the Cleaning cosine merge)
- Spectra come from a different pipeline without NP3's noise_cutoff and MGF layout, since the similarity comparison assumes NP3 preprocessed MS2 features

## Inputs

- Preprocessed MS2 spectra in MGF format (output of NP3 preprocessing on LC–MS/MS raw data such as MSV000080502)
- Absolute noise_cutoff value (minimum base peak intensity, without normalization)
- Metadata Table describing LC–MS/MS samples, sample groups, and blank samples

## Outputs

- Clean consensus spectrum representatives exported as an MGF file
- Per-feature quantification as the clean count table
- Rate of duplicated consensus spectra for validation

## How to apply

After Step 3 clustering and the absolute noise_cutoff filter (MS2 spectra whose minimum base peak intensity is below the cutoff are removed), iterate clustering rounds where each round compares spectra pairwise with the NP3-shifted cosine — a greedy formulation of the modified cosine — to merge redundant spectra of the same ion. Continue merging rounds until the maximum number of rounds is reached or clustering converges; stop the count-table removal only if this is the last round. Feed the resulting clean consensus spectra representatives as an MGF export and the per-feature quantification as the clean count table. Validate on the case-study data that the duplicated consensus spectra rate stays near the reported 1.5% and that all 11 distinct isomers of m/z 589.31 remain separate clean consensus spectra.

## Related tools

- **NP3 MS Workflow** (Runs steps 3–5 (Clustering, Quantification, Cleaning) where the NP3-shifted cosine drives the spectral-similarity clustering) — https://github.com/danielatrivella/NP3_MS_Workflow

## Examples

```
nextflow run danielatrivella/NP3_MS_Workflow --data MSV000080502 --noise_cutoff 1000
```

## Evaluation signals

- Duplicated consensus spectra rate should be near the reported 1.5% for case study #1 (MSV000080502)
- All 11 distinct isomers of m/z 589.31 must remain separate clean consensus spectra after clustering
- Each MS2 retained after Step 3 must have a minimum base peak intensity at or above the absolute noise_cutoff
- Clean count table row counts must match the number of exported MGF consensus spectra representatives
- Maximum-cluster-round termination must be honored: if the last clustering round is reached, no post-removal count-table access should occur

## Limitations

- The greedy formulation of the modified cosine may not capture the same similarity as the exact modified cosine; merges depend on the chosen iteration budget
- Blank sample ions must be excluded downstream; if the M+H]+ networks are created without blank exclusion, m/z appearing in bed samples remain and can distort clustering
- Behavior changed in v1.5.0 where the noise_cutoff became an absolute value (minimum base peak intensity without normalization) — older releases used different semantics
- If a prior pre-processed result is reused, MS1_no_MS2 table bookkeeping bugs in earlier versions (<1.4.1) could propagate into the clustering input

## Evidence
