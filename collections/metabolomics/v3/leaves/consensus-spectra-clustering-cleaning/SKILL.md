---
name: consensus-spectra-clustering-cleaning
description: Use when raw LC–MS/MS per-sample MGF files contain redundant, noisy,
  or duplicate MS2 spectra (e.g., multiple scans of the same precursor, isotopes,
  or media/blank signals) that must be collapsed into one cleaned consensus spectrum
  per compound before annotation or BCC ranking.
when_to_use_negative:
- The input is already a feature table or clean consensus spectra — rerunning clustering
  would recluster already-collapsed spectra
- The analysis is targeted MS/MS or spectral library matching of a single spectrum
  rather than untargeted LC–MS/MS mixture processing
- Data lack sample-group metadata (blanks, culture media, bioactivity fractions),
  since cleaning and BCC filtering depend on grouping spectra by sample groups
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Executes the Clustering, Quantification, and Cleaning pipeline (steps 3–5)
    via its 'run' command, producing clean consensus spectra
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: Per-sample MS2 file format input that clustering and cleaning operate on
- name: Tremolo
  role: 'Downstream consumer: searches the clean consensus spectra against the ISDB
    for library identification (step 6)'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:89d20bd9100fd1390c7294df71902975a276214f5e4c0bc940ff69a879e84b2b
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
  - Tremolo
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Consensus Spectra Clustering and Cleaning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters preprocessed MS2 spectra from an LC–MS/MS untargeted metabolomics run into consensus spectra using the NP3-shifted cosine, then cleans and deconvolves them (including automatic [M+H]+ ion selection) to produce clean consensus spectra suitable for library identification and bioactivity correlation scoring.

## When to use

Use when raw LC–MS/MS per-sample MGF files contain redundant, noisy, or duplicate MS2 spectra (e.g., multiple scans of the same precursor, isotopes, or media/blank signals) that must be collapsed into one cleaned consensus spectrum per compound before annotation or BCC ranking.

## When NOT to use

- The input is already a feature table or clean consensus spectra — rerunning clustering would recluster already-collapsed spectra
- The analysis is targeted MS/MS or spectral library matching of a single spectrum rather than untargeted LC–MS/MS mixture processing
- Data lack sample-group metadata (blanks, culture media, bioactivity fractions), since cleaning and BCC filtering depend on grouping spectra by sample groups

## Inputs

- Per-sample MS2 spectra in Mascot Generic Format (MGF) files produced by Preprocessing (step 2)
- Metadata Table describing LC–MS/MS samples and sample groups (extract, fractions, blanks, culture media)
- noise_cutoff parameter (absolute minimum base peak intensity, unnormalized)
- Parameters per Supporting Information Note S13 for the BRA-346 case (MSV00091453)

## Outputs

- Clean consensus spectra (one representative MS2 spectrum per cluster) with precursor m/z and retention time
- Cluster/quantification count tables of precursor ion peak areas per sample
- Ionization variant annotation molecular networks (IVAMNs) with [M+H]+ representative ions selected by PageRank
- Cleaned spectra eligible for Tremolo library identification and BCC bioactivity ranking

## How to apply

After Preprocessing (step 2) has detected MS1 features and dumped per-sample MGF files, run Clustering, Quantification, and Cleaning (steps 3–5) of the NP3 MS Workflow: step 3 clusters preprocessed MS2 data by spectral similarity, applying an absolute noise_cutoff on the minimum base peak intensity (without normalization) to discard noise before the clean step; step 5 refines clusters using the NP3-shifted cosine, a greedy version of the modified cosine, iterating clustering rounds until convergence or a maximum number of rounds. During cleaning, automatic [M+H]+ ion deconvolution selects representative protonated ions using fragmentation rules, an ionization variant annotation molecular network (IVAMN), and PageRank to pick the [M+H]+ representative ion; m/z appearing in culture media/blank samples are removed from protonated networks by default. The resulting clean consensus spectra feed Library Identification (step 6, Tremolo against ISDB/GNPS2 libraries) and relative quantification for bioactivity correlation (BCC) scoring. In the BRA-346 case (MSV00091453), this step produced the clean consensus spectrum of m/z 343.187 (rt 4.24 min) that ranked highest with BCC 0.80.

## Related tools

- **NP3 MS Workflow** (Executes the Clustering, Quantification, and Cleaning pipeline (steps 3–5) via its 'run' command, producing clean consensus spectra) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (Per-sample MS2 file format input that clustering and cleaning operate on)
- **Tremolo** (Downstream consumer: searches the clean consensus spectra against the ISDB for library identification (step 6))

## Examples

```
nextflow run danielatrivella/NP3_MS_Workflow --cmd run --input_file Metadata.csv --ion_mode positive --noise_cutoff 1000
```

## Evaluation signals

- Number of clustering rounds terminates (convergence or max rounds reached) without the count-table access bug seen before v1.5.0
- m/z ions detected in blank or culture media samples are absent from the cleaned/protonated network tables (default bed-sample removal since v1.5.0)
- Each clean consensus spectrum has a single representative [M+H]+ precursor selected via PageRank over the IVAMN
- Reproducibility: IVAMN component ordering and edge sorting are deterministic (sorted by source node ID, largest components first) — re-running should yield byte-identical outputs per the fixed_results test cases
- Downstream sanity: in the BRA-346 case, the clean consensus spectrum m/z 343.187 (rt 4.24 min) must emerge and rank top with BCC 0.80, reproducible against Table 1

## Limitations

- The noise_cutoff must be set as an absolute base-peak intensity value (changed in v1.5.0); values carried over from earlier relative semantics will misfilter
- If the maximum number of clustering rounds is reached, cleaning stops early and some clusters may remain unrefined
- Full re-execution of the 'setup' command is required after version upgrades (e.g., v1.6.0 split the ALL_GNPS MGF library by ion mode), otherwise library-facing downstream steps may use stale libraries
- [M+H]+ networks are created without culture media by default; media-derived m/z removal is baked in and must be considered when interpreting ion coverage

## Evidence

- [methods] Clustering, Quantification, and Cleaning (steps 3–5) cluster preprocessed MS2 data relying on spectral similarity: "Clustering, Quantification, and Cleaning (steps 3–5) cluster preprocessed MS2 data relying on spectral similarity"
- [methods] NP3-shifted cosine: "Similarity comparisons in Cleaning use the NP3-shifted cosine function, a greedy version of the modified cosine"
