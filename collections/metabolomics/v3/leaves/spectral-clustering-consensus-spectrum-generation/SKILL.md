---
name: spectral-clustering-consensus-spectrum-generation
description: Use when you have per-sample MGF MS2 files from untargeted LC-MS/MS metabolomics
  (e.g., MSV000080502 Euphorbia dendroides data) and need to collapse redundant MS2
  scans into clean consensus spectra for GNPS/Tremolo library identification.
when_to_use_negative:
- Input is already a feature table or clean consensus spectra — steps 3-5 operate
  on raw per-sample MGF MS2 scans
- Data are non-LC-MS/MS MS1-only acquisitions, since clustering relies on MS2 spectral
  similarity
- You need de novo structure elucidation rather than consensus spectra for spectral
  library search — use structural annotation workflows instead
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Executes steps 3-5 (Clustering, Quantification, and Cleaning) using the NP3-shifted
    cosine function to generate clean consensus spectra
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Mascot Generic Format (MGF)
  role: 'Input format: NP3 preprocessing ''results in an MS2 file in Mascot Generic
    Format (MGF) for each sample'' consumed by the clustering step'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:4689154c6f3c363927602e36d3e2725f72aef5f5f83957c4299b82e12e85c2be
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
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-clustering-consensus-spectrum-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters preprocessed LC-MS/MS MS2 spectra by spectral similarity and merges them into clean consensus spectra using the NP3 MS Workflow's Clustering, Quantification, and Cleaning steps (3-5), with the NP3-shifted cosine function, to produce noise-free, per-component consensus MGF spectra ready for library annotation.

## When to use

Use when you have per-sample MGF MS2 files from untargeted LC-MS/MS metabolomics (e.g., MSV000080502 Euphorbia dendroides data) and need to collapse redundant MS2 scans into clean consensus spectra for GNPS/Tremolo library identification.

## When NOT to use

- Input is already a feature table or clean consensus spectra — steps 3-5 operate on raw per-sample MGF MS2 scans
- Data are non-LC-MS/MS MS1-only acquisitions, since clustering relies on MS2 spectral similarity
- You need de novo structure elucidation rather than consensus spectra for spectral library search — use structural annotation workflows instead

## Inputs

- Per-sample MS2 spectra in Mascot Generic Format (MGF) produced by NP3 pre-processing (step 2)
- NP3 Metadata Table describing LC-MS/MS samples and sample groups
- Parameter file with noise_cutoff (minimum base peak intensity, unnormalized) and ion_mode (positive/negative)

## Outputs

- Clean consensus spectra (one per spectral cluster component) for library identification
- Clean quantification/count table mapping m/z ions to samples
- Diagnosis report and clustering statistics from steps 3-5

## How to apply

Run NP3 MS Workflow steps 3-5 on the preprocessed per-sample MGF files: Step 3 clusters MS2 data relying on spectral similarity; steps 3-5 jointly cluster, quantify, and clean the spectra so that duplicate/noisy scans are merged per component. Similarity comparisons during Cleaning use the NP3-shifted cosine function, a greedy version of the modified cosine, which distinguishes minority compounds from noise. Iterative clustering is applied up to a maximum number of rounds (README notes a fix when 'the maximum number of clustering rounds is reached' in clean step 5); a noise_cutoff applies after Step 3 clustering, keeping only MS2 spectra above the minimum base peak intensity (without normalization). The output clean consensus spectra (one per cluster component) are then handed to Library Identification (Tremolo/GNPS search, step 6) and downstream SSMN construction, as in the m/z 589.31 isomer analysis where all 11 isomers were resolved and the m/z 607.32 consensus spectrum was annotated as a 4-deoxyphorbol ester (CCMSLIB00000840365).

## Related tools

- **NP3 MS Workflow** (Executes steps 3-5 (Clustering, Quantification, and Cleaning) using the NP3-shifted cosine function to generate clean consensus spectra) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Mascot Generic Format (MGF)** (Input format: NP3 preprocessing 'results in an MS2 file in Mascot Generic Format (MGF) for each sample' consumed by the clustering step)

## Evaluation signals

- Iterative clustering terminates correctly: if the maximum number of clustering rounds is reached in clean step 5, the count table is not removed after the last round (regression fixed in v1.5.0)
- noise_cutoff is applied post-Step-3 clustering as an absolute (unnormalized) base peak intensity threshold, removed before clean step 5
- Clean consensus spectra for the m/z 589.31 cluster must resolve exactly 11 distinct isomers with distinct peaks in the 1400-1800 s retention window
- The m/z 607.32 clean consensus spectrum must be annotatable as a 4-deoxyphorbol ester analogue against GNPS entry CCMSLIB00000840365
- Test-case equality checks (added in v1.4.2) compare NP3 results against fixed_results to guarantee reproducibility and correctness of the steps

## Limitations

- If the maximum number of clustering rounds is reached, results may be less consolidated (a count-table access bug after removal was fixed in v1.5.0)
- Since v1.5.1, when the noise cutoff is applied the filtered IVAMN must reset its index to prevent matching errors during join_jobs — mis-set indices can corrupt downstream joins
- Consensus quality depends on preprocessing parameters mirroring the original GNPS processing parameters; suboptimal settings can split or merge isomer spectra

## Evidence

- [methods] Clustering, Quantification, and Cleaning (steps 3-5) cluster preprocessed MS2 data relying on spectral similarity: [evidence span withheld — anchor status: not_found]
