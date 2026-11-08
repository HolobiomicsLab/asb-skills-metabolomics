---
name: ms2-spectral-clustering-and-cleaning
description: Use when processing raw LC–MS/MS untargeted metabolomics data (e.g. mzML/MGF
  per-sample MS2 files from a feature-detection step) and you need to consolidate
  redundant MS2 spectra into clean consensus spectra for downstream molecular networking,
  library identification, and bioactivity correlation.
when_to_use_negative:
- Input is already a clean consensus spectra table or a feature quantification table
  (e.g. GNPS FBMN output) — clustering/cleaning has already been performed
- The analysis targets raw MS1-only quantification without MS2 spectra, since this
  skill depends on MS2 spectral similarity
- Data were acquired as targeted MS/MS of a short inclusion list with few redundant
  spectra per feature, where spectral clustering adds little consolidation value
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Executes Clustering, Quantification, and Cleaning (steps 3–5) with the NP3-shifted
    cosine function within the automated `run` command
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: Per-sample MS2 file format produced by Preprocessing that is the direct input
    to clustering and cleaning
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:fdea8265bf04d15b7b1f9850bd58f66644dc7fc5fa4f1c905a7ae6ba4f5395e2
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
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  - Tremolo
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ms2-spectral-clustering-and-cleaning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster preprocessed MS2 spectra by spectral similarity, quantify the resulting clusters, and consolidate each cluster into a single 'clean consensus spectrum' using the NP3-shifted cosine function, as implemented in steps 3–5 of the NP3 MS Workflow for LC–MS/MS untargeted metabolomics.

## When to use

Use when processing raw LC–MS/MS untargeted metabolomics data (e.g. mzML/MGF per-sample MS2 files from a feature-detection step) and you need to consolidate redundant MS2 spectra into clean consensus spectra for downstream molecular networking, library identification, and bioactivity correlation.

## When NOT to use

- Input is already a clean consensus spectra table or a feature quantification table (e.g. GNPS FBMN output) — clustering/cleaning has already been performed
- The analysis targets raw MS1-only quantification without MS2 spectra, since this skill depends on MS2 spectral similarity
- Data were acquired as targeted MS/MS of a short inclusion list with few redundant spectra per feature, where spectral clustering adds little consolidation value

## Inputs

- Preprocessed LC–MS/MS feature tables and per-sample MS2 files in Mascot Generic Format (MGF) from step 2 Preprocessing
- Metadata Table describing LC–MS/MS samples and sample groups (including blanks and culture media for BCC filtering)
- noise_cutoff parameter: absolute minimum base peak intensity (unnormalized) for retaining MS2 after clustering

## Outputs

- Clean consensus spectra (one per MS2 cluster) with m/z, retention time, and per-sample precursor-ion quantification
- Clean count table of m/z ion occurrences per sample used for bioactivity correlation and exclusive-feature counts
- Molecular networking statistics including duplicated consensus spectra rate and probable exclusive features for SSMN/FBMN comparison

## How to apply

After Preprocessing (step 2) detects LC–MS features and exports one MGF per sample, run Clustering, Quantification, and Cleaning (steps 3–5): cluster preprocessed MS2 data relying on spectral similarity, quantify precursor-ion abundance per sample, and consolidate each cluster into a clean consensus spectrum. Similarity comparisons in Cleaning use the NP3-shifted cosine function, a greedy version of the modified cosine, which improves grouping of duplicated spectra while preserving minority compounds. Apply an absolute noise_cutoff — the minimum base peak intensity (without normalization) an MS2 must have after step 3 clustering, otherwise it is removed before clean step 5; iterate clustering rounds up to a maximum, removing count-table entries for merged clusters. The output clean consensus spectra, with their m/z, retention time, and per-sample peak areas, feed Library Identification (Tremolo/ISDB), IVAMN [M+H]+ deconvolution via PageRank, and filtered SSMN construction, and are the basis for computing duplicated consensus spectra rates and probable exclusive features.

## Related tools

- **NP3 MS Workflow** (Executes Clustering, Quantification, and Cleaning (steps 3–5) with the NP3-shifted cosine function within the automated `run` command) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (Per-sample MS2 file format produced by Preprocessing that is the direct input to clustering and cleaning)

## Examples

```
np3 run --project_dir <dir>   # executes Preprocessing (step 2) then Clustering, Quantification, and Cleaning (steps 3–5) through the final reports
```

## Evaluation signals

- Each MS2 cluster yields exactly one clean consensus spectrum; the duplicated consensus spectra rate (same clean spectrum assigned to multiple features) is low — the paper reports 1.5% for NP3 vs 0.4% for GNPS FBMN on MSV000080502
- Spectra below the absolute noise_cutoff base peak intensity are removed before clean step 5; count-table access/removal errors must not occur when the maximum number of clustering rounds is reached (bug fixed in v1.5.0)
- The clean count table supports computing probable exclusive features within m/z and retention-time tolerances, excluding isotopes and blank/culture-media m/z — e.g. 51.2% of features exclusive to NP3 vs FBMN on MSV000080502
- Downstream Library Identification (Tremolo) finds matches for clean consensus spectra against ISDB, and IVAMN PageRank selects [M+H]+ representative ions from the cleaned clusters
- Consolidated node counts in the filtered SSMN (e.g. 1413 nodes for NP3 vs 765 for GNPS FBMN on MSV000080502) reflect the effect of the clustering/cleaning parameters applied

## Limitations

- Iterative clustering uses a maximum number of clustering rounds; when reached, remaining clusters are not further merged, which can leave residual duplicated spectra
- The noise_cutoff is an absolute (unnormalized) base peak intensity, so results depend on instrument response and are not normalized across acquisitions
- Version history records sensitivity to index-reset bugs after noise-cutoff filtering during IVAMN joining (fixed in 1.5.1) and count-table removal bugs at max clustering rounds (fixed in 1.5.0), so older versions may produce inconsistent clean tables

## Evidence
