---
name: mass-spectrometry-metadata-integration
description: 'Use when a GNPS molecular networking job must be tied back to sample
  context: uploading a standardized metadata table with the LC-MS/MS files and classifying
  network nodes by sample origin (e.g., lung tissue vs. cultured isolates).'
when_to_use_negative:
- The input is already a feature/quantification table with metadata merged — only
  network node tables from GNPS output need this join
- The goal is in silico structural annotation of unannotated 'dark matter' spectra,
  which the article places beyond the scope of this networking protocol
- Statistical testing or quantitation is required — classic molecular networking provides
  only qualitative information (MS2 scans only) and the tutorial example performed
  no statistics
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Platform for submitting the METABOLOMICS-SNETS-V2 molecular networking job
    whose output node tables are merged with the sample metadata; also hosts the standardized
    metadata upload alongside public datasets
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: Cytoscape
  role: Visualizes the imported GNPS network and is used to map metadata attributes
    onto nodes to color/classify the quinolone subnetwork by sample origin
- name: GNPS-MassIVE repository
  role: Public repository where the LC-MS/MS files and the standardized sample metadata
    table for MSV000083359 are retrieved and deposited
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: WinSCP
  role: FTP client for uploading the data files and metadata table to MassIVE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:1e180aa99b5ac90beee1a861eeb9dd020921aa6b54e9f8c5092d97856e6aaba2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPSDocumentation
  tools:
  - Cytoscape
  - GNPS
  - GNPS-MassIVE repository
  - WinSCP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-metadata-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collate a standardized sample-information (metadata) table for an LC-MS/MS dataset, upload it alongside the raw files to GNPS-MassIVE, and map sample-origin attributes onto molecular network nodes in Cytoscape so that subnetworks (e.g., the Pseudomonas quinolone family in MSV000083359) can be classified by where each node was detected.

## When to use

Use when a GNPS molecular networking job must be tied back to sample context: uploading a standardized metadata table with the LC-MS/MS files and classifying network nodes by sample origin (e.g., lung tissue vs. cultured isolates).

## When NOT to use

- The input is already a feature/quantification table with metadata merged — only network node tables from GNPS output need this join
- The goal is in silico structural annotation of unannotated 'dark matter' spectra, which the article places beyond the scope of this networking protocol
- Statistical testing or quantitation is required — classic molecular networking provides only qualitative information (MS2 scans only) and the tutorial example performed no statistics

## Inputs

- LC-MS/MS data files in open formats (.mzXML, .mzML, .mgf) retrieved from a GNPS-MassIVE accession (e.g., MSV000083359)
- Standardized sample information (metadata) table mapping filenames to sample origin (tissue vs. isolate)
- GNPS molecular networking job output tables (clusterinfo summary and quantification table for Cytoscape import)

## Outputs

- Metadata-annotated network node table with sample-origin classification (e.g., orange = lung tissue + isolates, cyan = isolates only)
- Quinolone subnetwork node table exported as CSV (quinolone_subnetwork_node_table.csv)

## How to apply

Collate a metadata table whose sample names/filenames exactly match the LC-MS/MS files (converted to .mzXML/.mzML/.mgf) and upload both to the GNPS-MassIVE dataset, since the paper stresses reproducibly creating publishable networks with standardized sample information. After running a METABOLOMICS-SNETS-V2 job (library search at the default cosine 0.7 with 6 matched MS2 ions), download the GNPS job output tables (clusterinfo/quantification) and import the network into Cytoscape. Merge the metadata onto the network node attribute table (join on filename/sample identifier) and use node attributes to color/classify nodes — e.g., quinolone nodes detected in lung tissue plus isolates (orange) versus isolates only (cyan). Export the classified subnetwork node table (e.g., quinolone_subnetwork_node_table.csv). Correct application is judged by whether library-annotated nodes carry the right origin labels and reproduce the published tutorial result (HHQ, NHQ, NHQ-C9:1, NQNO at m/z 244.1696, 270.1852, 272.2009, 288.1958).

## Related tools

- **GNPS** (Platform for submitting the METABOLOMICS-SNETS-V2 molecular networking job whose output node tables are merged with the sample metadata; also hosts the standardized metadata upload alongside public datasets) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **Cytoscape** (Visualizes the imported GNPS network and is used to map metadata attributes onto nodes to color/classify the quinolone subnetwork by sample origin)
- **GNPS-MassIVE repository** (Public repository where the LC-MS/MS files and the standardized sample metadata table for MSV000083359 are retrieved and deposited) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **WinSCP** (FTP client for uploading the data files and metadata table to MassIVE)

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22,%22library_on_server%22:%22d.speclibs;%22%7D  # then import DB_result_network.graphml + quantification table into Cytoscape and merge the metadata CSV on filename
```

## Evaluation signals

- The exported quinolone node table contains HHQ, NHQ, NHQ-C9:1 db and NQNO at m/z 244.1696, 270.1852, 272.2009, and 288.1958
- Each node's orange/cyan origin classification matches the metadata: orange for nodes detected in lung tissue plus isolates, cyan for isolates only
- Every filename in the metadata table matches exactly one uploaded LC-MS/MS file in the MassIVE dataset (no orphan rows)
- Library-annotated nodes were found at the GNPS default empirical cutoff of cosine 0.7 with 6 matched MS2 ions before metadata merging
- The final node table reproduces the protocol's published tutorial result for the MSV000083359 quinolone subnetwork

## Limitations

- Classic molecular networking is qualitative only (MS2 scans only); calibrated quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated, so most nodes cannot be given compound-level metadata beyond origin
- MS2 spectra typically cannot differentiate regio- or stereoisomers, and the tutorial analysis was not performed blindly and included no statistics

## Evidence

- [full_text] Metadata mapping of node origin: [evidence span withheld — anchor status: not_found]
- [intro] Library-search cutoffs before metadata merge: [evidence span withheld — anchor status: not_found]
- [supplementary] Data and metadata source accessibility: [evidence span withheld — anchor status: not_found]
