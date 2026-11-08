---
name: lc-msms-feature-quantification
description: Use when raw LC–MS/MS runs (e.g. mzML/MGF per sample) must be turned
  into per-feature relative abundances that support bioactivity correlation scoring
  or fraction-based prioritization of bioactive natural products.
when_to_use_negative:
- Input is already a finalized, curated feature-abundance table (e.g. exported count
  table ready for BCC computation) — the quantification/clustering stage would be
  redundant.
- The goal is pure spectral library matching without relative quantification; Library
  Identification (step 6) via Tremolo/GNPS2 alone suffices.
- Targeted metabolomics data already processed outside NP3 (e.g. with vendor peak
  tables) where the NP3 clustering/noise_cutoff behavior is not desired.
edam_operation: http://edamontology.org/operation_3615
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: NP3 MS Workflow
  role: Executes the clustering/quantification/cleaning steps (3–5) that turn preprocessed
    LC–MS/MS features into a clean count table for bioactivity correlation scoring.
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Visualization of the SSMN derived from the quantified/cleaned consensus spectra.
- name: Mascot Generic Format (MGF)
  role: Output format for per-sample MS2 files produced by preprocessing and consumed
    by clustering/quantification.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:98ca71789037487a7379bd21e4f04c586a4c837f0875c19b3e2815ea01f10354
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
  - Cytoscape
  - GNPS
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  edam_operation: http://edamontology.org/operation_3615
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# lc-msms-feature-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Relative quantification of LC–MS/MS precursor-ion features inside the NP3 MS Workflow, where preprocessed MS1 peaks and their MS2 spectra are clustered into clean consensus spectra and converted into a clean count table whose peak-area abundances are linked to sample groups for bioactivity correlation scoring.

## When to use

Use when raw LC–MS/MS runs (e.g. mzML/MGF per sample) must be turned into per-feature relative abundances that support bioactivity correlation scoring or fraction-based prioritization of bioactive natural products.

## When NOT to use

- Input is already a finalized, curated feature-abundance table (e.g. exported count table ready for BCC computation) — the quantification/clustering stage would be redundant.
- The goal is pure spectral library matching without relative quantification; Library Identification (step 6) via Tremolo/GNPS2 alone suffices.
- Targeted metabolomics data already processed outside NP3 (e.g. with vendor peak tables) where the NP3 clustering/noise_cutoff behavior is not desired.

## Inputs

- LC–MS/MS sample files (per-sample MGF/MS2 plus MS1 peak detection input, e.g. from Waters H-Class UPLC / Bruker Impac II qQToF runs)
- Metadata Table describing LC–MS/MS samples, sample groups and bioactivity values (e.g. IC50 for proteasome inhibition)
- Optional blank and culture media samples for downstream filtering of the count table

## Outputs

- Clean consensus spectra with associated precursor ions (clean count table of relative peak-area abundances per sample)
- Clustered MS2 spectra per sample in Mascot Generic Format (MGF)
- Quantification and chemistry statistics/plots in final reports (including superclass grouping and IVAMN-based molecular networking output openable in Cytoscape)

## How to apply

Provide a Metadata Table (step 1) describing samples, sample groups and any bioactivity measurements (e.g. proteasome-inhibition IC50 values), then run the NP3 MS Workflow 'run' command so that Preprocessing (step 2) detects LC–MS features (MS1 peaks) per sample and extracts their MS2 spectra; Clustering, Quantification and Cleaning (steps 3–5) then rely on spectral similarity — compared with the NP3-shifted cosine function, a greedy version of the modified cosine — to group MS2 spectra and quantify precursor ions. Quantification is tuned for drug discovery on natural products, ensuring high sensitivity for minority compounds while distinguishing them from noise (noise_cutoff = absolute minimum base peak intensity, without normalization, that an MS2 must have to be kept after clustering). The resulting clean consensus spectra are collapsed into a clean count table of precursor-ion abundances; ionization variants (e.g. [M+H]+ networks) are handled via IVAMN with PageRank selection of representative [M+H]+ ions, and the table is later used to compute bioactivity correlation coefficients (BCC) against sample groups. Validate quantification by checking that clean consensus spectra reproduce expected group-specific patterns, e.g. m/z 343.187 detected only in bioactive fractions of peak 1, or m/z 679.272 showing highest intensities in inhibitory fractions.

## Related tools

- **NP3 MS Workflow** (Executes the clustering/quantification/cleaning steps (3–5) that turn preprocessed LC–MS/MS features into a clean count table for bioactivity correlation scoring.) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Visualization of the SSMN derived from the quantified/cleaned consensus spectra.)
- **Mascot Generic Format (MGF)** (Output format for per-sample MS2 files produced by preprocessing and consumed by clustering/quantification.)

## Examples

```
nextflow run NP3_MS_Workflow --run command --input LCMS/ --metadata Metadata.tsv --noise_cutoff 1000
```

## Evaluation signals

- Clustering/quantification must reproduce expected group-specific detections, e.g. m/z 343.187 (rt 4.24 min) only in bioactive fractions of peak 1, and m/z 679.272 (rt 5.65 min) with highest intensities in inhibitory fractions.
- Clean count table columns must match the Metadata Table sample groups, with blanks/beds excluded from the [M+H]+ networks (removed by default since v1.5.0).
- Reproducibility check: added test cases compare NP³ results to fixed_results, with equality tests performed at the end of each saved test case to guarantee consistency.
- noise_cutoff is applied as an absolute minimum base peak intensity (unnormalized); MS2 below cutoff must be removed after clustering (step 3) before cleaning (step 5).
- Final reports should include quantification and molecular networking statistics, and the IVAMN components must be ordered reproducibly (largest components first, edges sorted by source node ID).

## Limitations

- Quantification is relative (peak-area based) and intended for untargeted drug-discovery prioritization, not absolute quantification.
- Results depend on the Metadata Table grouping and the bioactivity values entered (e.g. inverse BCC when IC50 values are used, so lower BCC = stronger bioactivity association).
- Noise cutoff is absolute (unnormalized base peak intensity), so sensitivity for minority compounds is a trade-off against noise retention.
- Partial pre-processing reuse had a bug fixed in v1.4.1; older results may contain MS1_no_MS2 artifacts.

## Evidence
