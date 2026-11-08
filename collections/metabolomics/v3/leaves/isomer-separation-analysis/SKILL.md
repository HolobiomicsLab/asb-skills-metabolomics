---
name: isomer-separation-analysis
description: Use when an LC–MS/MS dataset shows a single precursor m/z (e.g. m/z 589.31)
  with multiple chromatographic peaks, and the goal is to count distinct isomers,
  assign them to spectral clusters, and annotate them via GNPS library spectra.
when_to_use_negative:
- When the analysis targets are bioactivity-ranked ions (BCC scoring against blank/culture
  media) rather than isomer resolution within one precursor m/z
- When input is already a feature/quantification table rather than raw LC–MS/MS data
  requiring NP3 preprocessing
- When the precursor m/z has only a single chromatographic peak and no spectral-cluster
  ambiguity to resolve
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: NP3 MS Workflow
  role: Performs preprocessing, NP3-shifted-cosine spectral clustering/quantification/cleaning,
    IVAMN with PageRank [M+H]+ selection, and SSMN construction on the Euphorbia dendroides
    LC–MS/MS data
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Spectral library database used to annotate the clean consensus spectra of
    the resolved isomers and the m/z 607.32 4-deoxyphorbol ester analogue
- name: Cytoscape
  role: Graph visualization of the filtered SSMN for visual confirmation of spectral-cluster
    membership
- name: Tremolo
  role: Library Identification (step 6) tool that searches clean consensus spectra
    against MS2 databases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:2883b9a10a27e107416188f3840ee6a871386aa6e08cbc10f6df7a850fea28fd
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
  - GNPS (Global Natural Products Social Molecular Networking)
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  - Tremolo
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# isomer-separation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolves how many distinct isomeric species underlie a single precursor m/z in LC–MS/MS data by combining NP3 MS Workflow's spectral clustering (NP3-shifted cosine) with extracted ion chromatogram peak counting, then annotating each resolved isomer against the GNPS spectral database. Demonstrated on the Euphorbia dendroides MSV000080502 dataset, where 11 distinct m/z 589.31 isomers were detected and the associated m/z 607.32 ion was annotated as a 4-deoxyphorbol ester analogue.

## When to use

Use when an LC–MS/MS dataset shows a single precursor m/z (e.g. m/z 589.31) with multiple chromatographic peaks, and the goal is to count distinct isomers, assign them to spectral clusters, and annotate them via GNPS library spectra.

## When NOT to use

- When the analysis targets are bioactivity-ranked ions (BCC scoring against blank/culture media) rather than isomer resolution within one precursor m/z
- When input is already a feature/quantification table rather than raw LC–MS/MS data requiring NP3 preprocessing
- When the precursor m/z has only a single chromatographic peak and no spectral-cluster ambiguity to resolve

## Inputs

- LC–MS/MS dataset MSV000080502 (Euphorbia dendroides extract, 16 samples) from MassIVE/GNPS
- NP3 Metadata Table describing LC–MS/MS samples and sample groups
- GNPS spectral database references CCMSLIB00000840551, CCMSLIB00000840552, and CCMSLIB00000840365

## Outputs

- Per-sample MS2 files in Mascot Generic Format (MGF) and a preprocessing diagnosis report
- mz58931_isomer_analysis.csv listing the isomers, spectral-cluster membership, and exclusive-feature percentage
- Filtered SSMN (spectral similarity molecular network) for visualization in Cytoscape
- GNPS annotation of the m/z 607.32 clean consensus spectrum as a 4-deoxyphorbol ester analogue (CCMSLIB00000840365)

## How to apply

Preprocess the LC–MS/MS data (MSV000080502 from the Euphorbia dendroides extract, 16 samples) with the NP3 MS Workflow: build the NP3 Metadata Table describing samples and sample groups, run Preprocessing (step 2) to produce per-sample MGF files, then execute Clustering, Quantification, and Cleaning (steps 3–5), which cluster preprocessed MS2 data relying on spectral similarity using the NP3-shifted cosine function, a greedy version of the modified cosine. Run Library Identification (step 6, via the Tremolo tool) or annotate clean consensus spectra against the GNPS spectral database using deposited 4-deoxyphorbol ester references CCMSLIB00000840551 and CCMSLIB00000840552. Extract the m/z 589.31 ion chromatogram over a retention window of 1400–1800 s and count the distinct chromatographic peaks; verify that distinct peaks map to distinct spectral clusters (e.g. isomers IX and XI fall in separate clusters corresponding to the 2 Da-shifted spectra type E). Compute the percentage of NP3-exclusive features within the m/z 589.31 spectral cluster, confirm the m/z 607.32 clean consensus spectrum is annotated as CCMSLIB00000840365, and compile results into mz58931_isomer_analysis.csv for Cytoscape inspection of the filtered SSMN.

## Related tools

- **NP3 MS Workflow** (Performs preprocessing, NP3-shifted-cosine spectral clustering/quantification/cleaning, IVAMN with PageRank [M+H]+ selection, and SSMN construction on the Euphorbia dendroides LC–MS/MS data) — https://github.com/danielatrivella/NP3_MS_Workflow
- **GNPS (Global Natural Products Social Molecular Networking)** (Spectral library database used to annotate the clean consensus spectra of the resolved isomers and the m/z 607.32 4-deoxyphorbol ester analogue)
- **Cytoscape** (Graph visualization of the filtered SSMN for visual confirmation of spectral-cluster membership)
- **Tremolo** (Library Identification (step 6) tool that searches clean consensus spectra against MS2 databases)

## Evaluation signals

- The results table contains exactly 11 distinct m/z 589.31 isomers
- The m/z 607.32 entry is annotated to GNPS entry CCMSLIB00000840365
- The NP3-exclusive-feature percentage within the m/z 589.31 spectral cluster is consistent with the reported 39%
- Isomers IX and XI fall in distinct spectral clusters corresponding to the 2 Da-shifted spectra type E
- Extracted ion chromatogram (retention window 1400–1800 s) shows one chromatographic peak per distinct isomer

## Limitations

- The m/z 589.31 result depends on preprocessing parameters mirroring the original GNPS processing; different parameters may merge or split clusters
- Isomer distinction ultimately relies on chromatographic separation and spectral similarity (NP3-shifted cosine); co-eluting isomers with near-identical MS2 spectra may not resolve
- The noise_cutoff and [M+H]+ network construction behavior changed across NP3 MS Workflow versions (e.g. v1.5.0 changes), so results may vary by software version

## Evidence
