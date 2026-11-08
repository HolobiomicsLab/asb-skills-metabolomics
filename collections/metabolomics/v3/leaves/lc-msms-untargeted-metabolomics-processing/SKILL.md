---
name: lc-msms-untargeted-metabolomics-processing
description: Use when raw LC–MS/MS untargeted metabolomics files (e.g. mzML from a
  Bruker Impac II qQToF or similar) plus a Metadata Table describing samples/sample
  groups must be converted into quantified, cleaned consensus spectra, library annotations,
  and a filtered SSMN.
when_to_use_negative:
- Input is already a feature/quantification table or clean consensus spectra — the
  skill targets raw LC–MS/MS data processing from step 2 onward
- The goal is a purely targeted MS/MS experiment (e.g. MRM/SRM) rather than untargeted
  feature detection, clustering, and molecular networking
- A plain GNPS FBMN comparison is wanted without NP3-specific steps such as NP3-shifted
  cosine cleaning, IVAMN [M+H]+ deconvolution, or K-most-similar-neighbor damped pruning
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NP3 MS Workflow
  role: 'Main pipeline: executes steps 1–10 (preprocess, cluster, quantify, clean,
    identify, IVAMN, SSMN) via the run/join_jobs CLI commands'
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Tremolo
  role: 'Step 6 Library Identification: searches clean consensus spectra against ISDB'
- name: Cytoscape
  role: Visualization of the resulting SSMN graph; v1.6.0 adds automated SSMN mounting
    in Cytoscape via post-processing
- name: Mascot Generic Format (MGF)
  role: 'Intermediate file format: one MS2 MGF file per sample produced by Preprocessing'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:cc2a23d180c2bcb017b470dfd7f8ee33d42893d625e933b0db2a00d0d6248f39
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lc-msms-untargeted-metabolomics-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end processing of LC–MS/MS untargeted metabolomics data with the NP3 MS Workflow, a Nextflow-based open-source pipeline (steps 1–10) that detects MS1 features, clusters and cleans MS2 spectra with the NP3-shifted cosine, identifies compounds via Tremolo against ISDB/GNPS2, and builds [M+H]+ IVAMNs and filtered SSMNs for natural-product-based drug discovery.

## When to use

Use when raw LC–MS/MS untargeted metabolomics files (e.g. mzML from a Bruker Impac II qQToF or similar) plus a Metadata Table describing samples/sample groups must be converted into quantified, cleaned consensus spectra, library annotations, and a filtered SSMN.

## When NOT to use

- Input is already a feature/quantification table or clean consensus spectra — the skill targets raw LC–MS/MS data processing from step 2 onward
- The goal is a purely targeted MS/MS experiment (e.g. MRM/SRM) rather than untargeted feature detection, clustering, and molecular networking
- A plain GNPS FBMN comparison is wanted without NP3-specific steps such as NP3-shifted cosine cleaning, IVAMN [M+H]+ deconvolution, or K-most-similar-neighbor damped pruning

## Inputs

- Raw LC–MS/MS untargeted metabolomics data files per sample (converted to MGF per sample by Preprocessing)
- Metadata Table describing LC–MS/MS samples and sample groups (step 1)
- Molecular networking / library search parameters (ion mode, minimum number of matched peaks, noise_cutoff, K neighbors)
- Reference compound libraries: ISDB and locally retrieved ALL_GNPS_NO_PROPOGATED.mgf (split positive/negative)

## Outputs

- Clean consensus spectra with curated library annotations (curated_lib_annotation_* columns)
- Relative precursor-ion quantification (clean count table) for bioactivity correlation scoring
- IVAMN [M+H]+ representative-ion selections
- Filtered SSMN (edge list) loadable in Cytoscape
- Final reports with quantification, chemical, and molecular networking statistics; PCA chemical-space plot vs reference dataset

## How to apply

Provide a Metadata Table describing the LC–MS/MS samples and sample groups as step 1, then run NP3 MS Workflow steps 2–10: Preprocessing detects LC–MS features (MS1 peaks) and extracts MS2 per sample into Mascot Generic Format (MGF); steps 3–5 cluster preprocessed MS2 by spectral similarity, quantify, and clean using the NP3-shifted cosine function (a greedy version of the modified cosine) with the noise_cutoff applied as an absolute minimum base peak intensity (v1.5.0+); step 6 uses Tremolo to search clean consensus spectra against the In Silico MS2 DataBase (ISDB) and, optionally (v1.4.0+), the local GNPS2 ALL_GNPS_NO_PROPOGATED library split by ion mode; [M+H]+ representative ions are selected via an IVAMN using the PageRank algorithm; finally the SSMN is filtered by removing links below the minimum number of matched peaks, keeping only the K-most similar neighbors of each node with damped pruning. Final reports include quantification, chemical statistics, and SSMN statistics, and the SSMN can be opened in Cytoscape.

## Related tools

- **NP3 MS Workflow** (Main pipeline: executes steps 1–10 (preprocess, cluster, quantify, clean, identify, IVAMN, SSMN) via the run/join_jobs CLI commands) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Tremolo** (Step 6 Library Identification: searches clean consensus spectra against ISDB)
- **Cytoscape** (Visualization of the resulting SSMN graph; v1.6.0 adds automated SSMN mounting in Cytoscape via post-processing)
- **Mascot Generic Format (MGF)** (Intermediate file format: one MS2 MGF file per sample produced by Preprocessing)

## Examples

```
np3_ms_workflow run  # executes NP3 Steps 2-10 (pre_process, cluster, quantify, clean, Tremolo/ISDB identify, IVAMN, filtered SSMN) from the Metadata Table
```

## Evaluation signals

- Each sample yields an MS2 file in Mascot Generic Format (MGF) from Preprocessing
- Final reports are produced containing quantification, chemical, and molecular networking statistics, and the SSMN opens correctly in Cytoscape
- Library annotation outputs carry the v1.6.0 'curated_lib_annotation_*' column prefixes, with ion_mode-aware GNPS2 annotations
- Pipeline test-case equality tests pass: results match the fixed_results folder, confirming reproducibility of each step
- In the paper's case study, NP3's filtered SSMN retained 1413 nodes vs 765 for GNPS FBMN (1.9x) with single-node rate 35.32% vs 44.71% and library annotation rate 5.1% vs 5.4%

## Limitations

- Version upgrades change required setup: v1.4.0 (GNPS2 library integration) and v1.6.0 (ALL_GNPS MGF split by ion mode) both require re-running the setup command before processing
- noise_cutoff semantics changed in v1.5.0 to an absolute minimum base peak intensity (unnormalized); older parameter values calibrated as relative cutoffs will not transfer
- Joining jobs (join_jobs) re-executes steps 3–10 with adaptations for incremental clustering; a v1.5.1 bug fixed index-reset errors in Step 7 when the noise cutoff filtered the IVAMN, so older joins may contain matching errors

## Evidence
