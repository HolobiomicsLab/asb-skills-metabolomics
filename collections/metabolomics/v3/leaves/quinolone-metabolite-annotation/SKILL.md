---
name: quinolone-metabolite-annotation
description: Use when an untargeted LC-MS/MS dataset (e.g., GNPS-MassIVE MSV000083359)
  contains MS2 spectra from biological tissue plus microbial isolates and the goal
  is to recover a specific molecular family, such as Pseudomonas quinolones, and map
  its nodes to sample metadata.
when_to_use_negative:
- The input is already a feature table (feature abundances per sample) rather than
  MS2 spectra — classic molecular networking considers only MS2 scans and cannot provide
  quantitative information; targeted follow-up LC-MS would be required for calibrated
  quantitation.
- The goal requires differentiating regio- or stereoisomers — MS2 spectra typically
  cannot resolve them.
- Comprehensive metabolite annotation of all nodes is expected — on average only ~5–6%
  of MS2 spectra in untargeted metabolomics are annotated, so most nodes will remain
  unannotated 'dark matter'.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow that clusters
    MS2 spectra into nodes, connects them by cosine similarity, and searches nodes
    against the GNPS spectral library at cosine 0.7 with 6 matched ions
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: Cytoscape
  role: Visualization of the downloaded GNPS molecular network, used to extract and
    color-code the quinolone subnetwork by sample origin
- name: GNPS Documentation
  role: Documentation for submitting and parameterizing the GNPS molecular networking
    job and downloading its output tables
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:5de34d472a03eae0038f4a647fff5441ed3b8967b1efe4321e71942a2df7bb69
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - Cytoscape
  - GNPS
  - GNPS Documentation
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# quinolone-metabolite-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recovers and annotates a quinolone molecular family (HHQ, NHQ, NHQ-C9:1, NQNO) from GNPS molecular networking of untargeted LC-MS/MS data, classifying nodes by sample origin (lung tissue vs. Pseudomonas isolates) in the MSV000083359 case study.

## When to use

Use when an untargeted LC-MS/MS dataset (e.g., GNPS-MassIVE MSV000083359) contains MS2 spectra from biological tissue plus microbial isolates and the goal is to recover a specific molecular family, such as Pseudomonas quinolones, and map its nodes to sample metadata.

## When NOT to use

- The input is already a feature table (feature abundances per sample) rather than MS2 spectra — classic molecular networking considers only MS2 scans and cannot provide quantitative information; targeted follow-up LC-MS would be required for calibrated quantitation.
- The goal requires differentiating regio- or stereoisomers — MS2 spectra typically cannot resolve them.
- Comprehensive metabolite annotation of all nodes is expected — on average only ~5–6% of MS2 spectra in untargeted metabolomics are annotated, so most nodes will remain unannotated 'dark matter'.

## Inputs

- Public LC-MS/MS raw/converted data files for MSV000083359 from the GNPS-MassIVE repository
- Standardized sample information (metadata) table mapping filenames to sample origin (lung tissue vs. cultured Pseudomonas isolates)
- GNPS METABOLOMICS-SNETS-V2 job output tables (clustered consensus spectra, library-search annotations, edge pairs)

## Outputs

- quinolone_subnetwork_node_table.csv containing the four quinolone nodes HHQ (m/z 244.1696), NHQ (m/z 270.1852), NHQ-C9:1 db (m/z 272.2009), and NQNO (m/z 288.1958) with orange/cyan origin classification
- Quinolone subnetwork visualization in Cytoscape with nodes colored by sample origin

## How to apply

Retrieve the public LC-MS/MS files and the standardized sample metadata table for MSV000083359 from the GNPS-MassIVE repository, then submit a Molecular Networking (METABOLOMICS-SNETS-V2) job on GNPS with default spectral filters: the precursor ion window filter deleting all peaks within ±17 Da of the precursor mass and the 50 Da window filter keeping only the top six most intense peaks per ±50 Da window. Consensus spectra (nodes) are clustered and connected by cosine spectral similarity, and each node is searched against the GNPS spectral library using the default empirical cutoff of cosine 0.7 with 6 matched MS2 ions. Download the job output tables, import the network into Cytoscape, extract the quinolone subnetwork, and annotate each node with its sample origin from the metadata table. Classify nodes as detected in lung tissue plus isolates (orange) versus isolates only (cyan), and save the classified node table as quinolone_subnetwork_node_table.csv. Validation relies on the protocol's published tutorial result, which the co-first authors verified independently by rerunning the analysis.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 molecular networking workflow that clusters MS2 spectra into nodes, connects them by cosine similarity, and searches nodes against the GNPS spectral library at cosine 0.7 with 6 matched ions) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **Cytoscape** (Visualization of the downloaded GNPS molecular network, used to extract and color-code the quinolone subnetwork by sample origin)
- **GNPS Documentation** (Documentation for submitting and parameterizing the GNPS molecular networking job and downloading its output tables) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22,%22library_on_server%22:%22d.speclibs;%22%7D
```

## Evaluation signals

- The output table contains exactly the four quinolone nodes HHQ, NHQ, NHQ-C9:1 db, and NQNO at m/z 244.1696, 270.1852, 272.2009, and 288.1958, matching the protocol's published tutorial result.
- Each node's origin classification is correct: HHQ, NHQ, NHQ-C9:1 db, and NQNO detected in lung tissue (orange); nodes present only in cultured isolates are cyan.
- Library annotations were assigned with the default empirical cutoffs: cosine ≥ 0.7 with ≥ 6 matched MS2 ions against the GNPS spectral library.
- The submitted job used the default METABOLOMICS-SNETS-V2 spectral filters (±17 Da precursor ion window deletion; top-six peaks per ±50 Da window retained).
- The reproducibility check mirrors the protocol's practice: an independent rerun of the same MSV000083359 job should reproduce the same subnetwork membership.

## Limitations

- Classic molecular networking is qualitative only (MS2 scans considered); calibrated quantitation requires follow-up targeted LC-MS experiments.
- Only a small fraction of spectra are annotated via library matching (~5–6% on average), so most nodes in the network stay unannotated; in silico annotation tools are beyond the scope of this networking protocol.
- For the tutorial example, statistics were not performed and the data analysis was not performed blindly, because knowledge of sample grouping and information is required.
- The classification depends on accurate standardized sample metadata; mislabeled or missing metadata rows will corrupt the orange/cyan origin annotation.

## Evidence

- [intro] An empirical cutoff for cosine scoring of 0.7 with 6 MS2 ions matching is the default setting in GNPS: [evidence span withheld — anchor status: not_found]
