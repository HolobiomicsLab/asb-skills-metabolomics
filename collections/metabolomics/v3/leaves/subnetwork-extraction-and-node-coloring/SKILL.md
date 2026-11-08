---
name: subnetwork-extraction-and-node-coloring
description: Use when a GNPS molecular networking job is complete and a specific molecular
  family (a connected cluster of MS2 consensus spectra) must be isolated and its nodes
  classified by sample origin, e.g., lung tissue vs. microbial isolates in MSV000083359.
when_to_use_negative:
- The goal is calibrated quantitation of the metabolites — classic molecular networking
  is qualitative because only MS2 scans are considered; follow-up targeted LC-MS is
  required.
- The analysis input is already a feature table (e.g., from MZmine2 or Feature-Based
  Molecular Networking) rather than a finished GNPS classic molecular networking job.
- The question requires differentiating regio- or stereoisomers — MS2 spectra typically
  cannot resolve these, so node coloring cannot distinguish them.
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 molecular networking job whose output tables
    (nodes, edges, library matches at cosine 0.7 with 6 matched MS2 ions) are the
    basis for subnetwork extraction
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: Cytoscape
  role: Visualization environment where the GNPS network is imported, the target subnetwork
    is extracted, and the metadata table is mapped to node attributes for orange/cyan
    origin coloring
- name: GNPS_Documentation
  role: Reference documentation for GNPS workflows and job output formats used when
    downloading and interpreting the network tables
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:b6f80dafb07764e6dabf6f06bf39a2ab4d9c6332145757f7fe2916ea3f15c85b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - Cytoscape
  - GNPS
  - GNPS_Documentation
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Subnetwork extraction and node coloring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts a target molecular family (e.g., the Pseudomonas quinolone subnetwork) from a GNPS METABOLOMICS-SNETS-V2 molecular network in Cytoscape and colors its nodes by sample origin using the standardized sample metadata table, yielding a publishable classified subnetwork node table.

## When to use

Use when a GNPS molecular networking job is complete and a specific molecular family (a connected cluster of MS2 consensus spectra) must be isolated and its nodes classified by sample origin, e.g., lung tissue vs. microbial isolates in MSV000083359.

## When NOT to use

- The goal is calibrated quantitation of the metabolites — classic molecular networking is qualitative because only MS2 scans are considered; follow-up targeted LC-MS is required.
- The analysis input is already a feature table (e.g., from MZmine2 or Feature-Based Molecular Networking) rather than a finished GNPS classic molecular networking job.
- The question requires differentiating regio- or stereoisomers — MS2 spectra typically cannot resolve these, so node coloring cannot distinguish them.

## Inputs

- GNPS METABOLOMICS-SNETS-V2 job output tables (network edges, consensus spectra/cluster results, library search results)
- Standardized sample information (metadata) table for the dataset, e.g., MSV000083359 lung-tissue extracts and cultured Pseudomonas isolates
- Library-annotated target nodes identifying the molecular family of interest (e.g., quinolones: HHQ, NHQ, NHQ-C9:1 db, NQNO)

## Outputs

- Extracted subnetwork (Cytoscape session/export) containing only the target molecular family
- Classified subnetwork node table (CSV) with per-node m/z, library annotation, and sample-origin class (e.g., orange = lung tissue + isolates, cyan = isolates only)
- Colored network graphic suitable for publication (e.g., reproducing Fig. 8)

## How to apply

After the METABOLOMICS-SNETS-V2 job finishes (consensus MS2 spectra clustered into nodes, edges formed by cosine spectral similarity, and library search applied with the default cutoff of cosine 0.7 with 6 matched MS2 ions), download the GNPS job output tables and import the network into Cytoscape. In Cytoscape, locate the molecular family of interest by its annotated member nodes and extract the connected subnetwork containing them. Map the sample information (metadata) table onto the network as node attributes so each node's occurrence across sample classes (diseased lung tissue extracts versus cultured Pseudomonas isolates) is explicit. Color nodes by origin — e.g., orange for nodes detected in both lung tissue and isolates, cyan for isolates-only nodes — and export the classified subnetwork node table (e.g., quinolone_subnetwork_node_table.csv) with node m/z, annotation, and origin class.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 molecular networking job whose output tables (nodes, edges, library matches at cosine 0.7 with 6 matched MS2 ions) are the basis for subnetwork extraction) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **Cytoscape** (Visualization environment where the GNPS network is imported, the target subnetwork is extracted, and the metadata table is mapped to node attributes for orange/cyan origin coloring)
- **GNPS_Documentation** (Reference documentation for GNPS workflows and job output formats used when downloading and interpreting the network tables) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Evaluation signals

- The exported quinolone subnetwork node table contains the four expected quinolone nodes HHQ, NHQ, NHQ-C9:1 db, and NQNO at m/z 244.1696, 270.1852, 272.2009, and 288.1958.
- Each node carries the correct orange/cyan origin classification: orange for nodes detected in both lung tissue and cultured isolates, cyan for isolates-only nodes, matching the published tutorial result.
- Library annotations within the subnetwork meet the default GNPS cutoff of cosine 0.7 with at least 6 matched MS2 ions.
- Node attributes in Cytoscape trace back to the standardized sample metadata table rows (one attribute column per sample class), so origin labels are auditable.
- The extracted subnetwork is a connected component: every node in the exported table is linked by cosine spectral-similarity edges to the quinolone family, with no unrelated clusters included.

## Limitations

- Classic molecular networking provides only qualitative information (MS2 scans only); no calibrated intensities are available for node coloring by abundance.
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated, so most subnetwork nodes will remain unannotated 'dark matter' and can only be colored by origin, not by identity.
- MS2 spectra typically cannot differentiate regio- or stereoisomers, so library-annotated node names may be ambiguous within the subnetwork.
- The tutorial example's data analysis was not performed blindly because knowledge of sample grouping and information is required to color nodes by origin, and no statistics were performed for the tutorial example.

## Evidence

- [intro] An empirical cutoff for cosine scoring of 0.7 with 6 MS2 ions matching is the default setting in GNPS: [evidence span withheld — anchor status: not_found]
