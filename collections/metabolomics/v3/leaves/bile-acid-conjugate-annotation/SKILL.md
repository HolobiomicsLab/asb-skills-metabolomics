---
name: bile-acid-conjugate-annotation
description: Use when a GNPS molecular network contains a library-annotated seed node
  (e.g. glycocholic acid, m/z 466.316) and structurally related analogs must be discovered
  by reading edge mass shifts (e.g. +79.993 Da for sulfation) to neighboring unannotated
  nodes.
when_to_use_negative:
- Quantitative comparison between GF and SPF samples is required — classic molecular
  networking considers only MS2 scans and provides only qualitative information, so
  calibrated quantitation needs follow-up targeted LC-MS
- The input is already a processed feature table — feature-based molecular networking
  (FBMN) rather than this classic MS2-spectrum propagation is the appropriate route
- Definitive regio- or stereochemical assignment of the conjugates is required — MS2
  spectra typically cannot differentiate regio- or stereoisomers, and in silico annotation
  tools are beyond the scope of this networking protocol
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Hosts the METABOLOMICS-SNETS-V2 molecular networking job with library spectral
    matching that produces the annotated network nodes and edges
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: Cytoscape
  role: Visualization of the downloaded .graphml/.cys network to locate the glycocholic
    acid seed node and inspect edge mass shifts to family neighbors
- name: MassIVE
  role: Public repository from which the MSV000083437 GF/SPF mouse duodenum LC-MS
    files and metadata are retrieved
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:4d22a31200dabbc91e8de3f5ee4be4ecfd29058b6ef49c99a285ab7d587651c2
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
  - MassIVE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- dipeptide-conjugated-bile-acid-annotation
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bile-acid-conjugate-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Propagates a GNPS spectral-library annotation (e.g. glycocholic acid at m/z 466.316) across edge mass shifts within a molecular network family to discover conjugated bile-acid analogs (phenylalanine-, tyrosine- and sulfate-conjugated cholic acid) in the MSV000083437 GF/SPF mouse duodenum dataset.

## When to use

Use when a GNPS molecular network contains a library-annotated seed node (e.g. glycocholic acid, m/z 466.316) and structurally related analogs must be discovered by reading edge mass shifts (e.g. +79.993 Da for sulfation) to neighboring unannotated nodes.

## When NOT to use

- Quantitative comparison between GF and SPF samples is required — classic molecular networking considers only MS2 scans and provides only qualitative information, so calibrated quantitation needs follow-up targeted LC-MS
- The input is already a processed feature table — feature-based molecular networking (FBMN) rather than this classic MS2-spectrum propagation is the appropriate route
- Definitive regio- or stereochemical assignment of the conjugates is required — MS2 spectra typically cannot differentiate regio- or stereoisomers, and in silico annotation tools are beyond the scope of this networking protocol

## Inputs

- LC-MS/MS dataset MSV000083437 (GF and SPF mouse duodenum) as .mzML or .mgf files in the GNPS-MassIVE repository
- Sample information (metadata) table attached to the GNPS job
- GNPS account for submitting the METABOLOMICS-SNETS-V2 job

## Outputs

- Molecular network exported as .graphml/.cys for Cytoscape
- Bucket table (consensus spectra table) from the GNPS results page
- Annotated conjugated bile-acid molecular family: nodes m/z 466.316 (glycocholic acid library hit), 556.362 (Phe-conjugated cholic acid), 572.358 (Tyr-conjugated cholic acid), 546.309 (sulfated cholic acid), 563.334 and 482.329

## How to apply

Retrieve the LC-MS data for MSV000083437 (germ-free vs specific-pathogen-free mouse duodenum) from the GNPS-MassIVE repository and confirm the .mzML/.mgf input files and sample metadata table are attached. Submit a METABOLOMICS-SNETS-V2 Molecular Networking job on GNPS with precursor and fragment ion mass tolerances of ±0.02 Da, minimum paired cosine score 0.7, minimum 4 matched fragment ions, TopK = 10 edges per node, and minimum cluster size 4; the job applies the standard spectral filters (deletion of peaks within ±17 Da of the precursor and keeping only the top six most intense peaks per ±50 Da window) and library search at cosine 0.7 with 6 matched ions. Download the .graphml/.cys network and bucket (consensus spectra) table, open the network in Cytoscape or the GNPS web viewer, and locate the glycocholic-acid library-annotated node at m/z 466.316. Then read edge mass shifts from that seed node to its family neighbors, propagating annotations: m/z 556.362 is annotated as phenylalanine-conjugated cholic acid, m/z 572.358 as tyrosine-conjugated cholic acid, and m/z 546.309 as sulfated cholic acid via the diagnostic 79.993 Da sulfate shift, alongside nodes at m/z 563.334 and 482.329 as shown in Figs 3 and 6 of the protocol. The rationale is that structurally related molecules share fragment ion patterns under MS2, so pairwise spectral similarity edges group analogs whose mass differences reveal the conjugating moiety.

## Related tools

- **GNPS** (Hosts the METABOLOMICS-SNETS-V2 molecular networking job with library spectral matching that produces the annotated network nodes and edges) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **Cytoscape** (Visualization of the downloaded .graphml/.cys network to locate the glycocholic acid seed node and inspect edge mass shifts to family neighbors)
- **MassIVE** (Public repository from which the MSV000083437 GF/SPF mouse duodenum LC-MS files and metadata are retrieved)

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22,%22library_on_server%22:%22d.speclibs;%22%7D
```

## Evaluation signals

- The exported .graphml/.cys network and bucket table both exist on the GNPS results page
- The bile-acid family reproduces the expected node set m/z 466.316, 556.362, 572.358, 546.309, 563.334 and 482.329, matching Figs 3 and 6 of the protocol
- The seed node matches the glycocholic acid spectral library entry at the default cutoffs of cosine 0.7 with 6 matched MS2 ions
- The m/z 546.309 neighbor displays the diagnostic 79.993 Da edge shift consistent with sulfation
- Job parameters match the protocol example: ±0.02 Da precursor/fragment tolerance, min paired cosine 0.7, min 4 matched fragment ions, TopK 10, minimum cluster size 4

## Limitations

- Classic molecular networking provides only qualitative information (MS2 scans only); calibrated quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated, so most family nodes remain unannotated 'dark matter'
- MS2 spectra typically cannot differentiate regio- or stereoisomers, so conjugate identities are putative; reproducibility of the tutorial example was verified by the co-first authors, but statistics were not performed and the analysis was not blinded

## Evidence

- [intro] library propagation rationale: "Each consensus spectrum (node) is then queried against spectral library databases to assign it to a putative known molecule within a network"
- [intro] structural similarity basis: [evidence span withheld — anchor status: not_found]
- [intro] library match cutoffs: [evidence span withheld — anchor status: not_found]
- [readme] workflow entry point: "| Molecular Networking  | [Workflow](https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22"
- [intro] standard spectral filter: "Removes peaks that are not one of the top six most intense within a ±50 Da window"
