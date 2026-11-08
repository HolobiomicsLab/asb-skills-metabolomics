---
name: gnps-molecular-networking-job-submission
description: Use when raw MS2 data (.mzML/.mgf) has been uploaded to MassIVE and the
  goal is to cluster spectra into molecular families, annotate known metabolites via
  library matching, and propagate annotations along edge mass shifts.
when_to_use_negative:
- Input is already a feature table or quantified matrix — classic molecular networking
  considers only MS2 scans and provides only qualitative information; calibrated quantitation
  requires follow-up targeted LC-MS
- The goal is quantitative statistics or blinded analysis — statistics were not performed
  for the tutorial protocol example, and knowledge of sample grouping is required
  for network inspection
- The task is in silico structure annotation (e.g., Sirius/MS2LDA) — the protocol
  states the application of such tools is beyond the immediate scope of the networking
  protocol
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Web platform hosting the METABOLOMICS-SNETS-V2 molecular networking workflow,
    spectral library search, and job result downloads
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Convert instrument-specific raw files to open formats (.mzXML, .mzML, .mgf)
    before upload and job submission
- name: MassIVE
  role: Public repository where LC-MS data files and metadata are hosted and referenced
    by the GNPS job
- name: Cytoscape
  role: Visualization of the downloaded .graphml/.cys molecular network and its subnetworks/families
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:c75ef1a32b98735e04e0948c9f94b02bb7ad00fbbd5718c4047e49867eae8337
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
  - MSConvert (ProteoWizard)
  - MassIVE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gnps-molecular-networking-job-submission

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Submit an LC-MS/MS dataset to GNPS as a Molecular Networking (METABOLOMICS-SNETS-V2) job so consensus MS2 spectra are clustered by pairwise spectral similarity and matched against the GNPS spectral library, producing a .graphml/.cys network and bucket table.

## When to use

Use when raw MS2 data (.mzML/.mgf) has been uploaded to MassIVE and the goal is to cluster spectra into molecular families, annotate known metabolites via library matching, and propagate annotations along edge mass shifts.

## When NOT to use

- Input is already a feature table or quantified matrix — classic molecular networking considers only MS2 scans and provides only qualitative information; calibrated quantitation requires follow-up targeted LC-MS
- The goal is quantitative statistics or blinded analysis — statistics were not performed for the tutorial protocol example, and knowledge of sample grouping is required for network inspection
- The task is in silico structure annotation (e.g., Sirius/MS2LDA) — the protocol states the application of such tools is beyond the immediate scope of the networking protocol

## Inputs

- LC-MS/MS data files in open formats (.mzML, .mzXML, or .mgf) uploaded to the GNPS-MassIVE repository (e.g., MSV000083437, MSV000083381)
- Sample information (metadata) table collated and attached to the job
- GNPS account credentials for job submission

## Outputs

- Molecular network as .graphml/.cys file (nodes = consensus spectra, edges = paired cosine similarity)
- Bucket table (consensus spectra/quantification table) from the job results page
- Library spectral matching annotations per node (cosine >= 0.7, >= 6 matched ions)

## How to apply

Create a GNPS/MassIVE account, then submit a METABOLOMICS-SNETS-V2 job pointing at the MassIVE dataset files with a sample metadata table attached. Configure parameters per the protocol's example: precursor and fragment ion mass tolerances of ±0.02 Da for high-resolution data (or 2.0/0.5 for low-resolution), minimum paired cosine 0.7, minimum matched fragment ions 4, TopK edges per node 10, and minimum consensus cluster size 4 (sweep 1–4 when probing sensitivity). The job internally applies spectral filters (deleting peaks within ±17 Da of the precursor and keeping only the top six most intense peaks per ±50 Da window), builds consensus spectra, and runs library search with a cosine cutoff of 0.7 and 6 matched MS2 ions. Download the job outputs (network .graphml/.cys and bucket/consensus table) and inspect subnetworks in the GNPS web viewer or Cytoscape, validating that library-annotated nodes anchor families whose edge mass shifts annotate analog nodes.

## Related tools

- **GNPS** (Web platform hosting the METABOLOMICS-SNETS-V2 molecular networking workflow, spectral library search, and job result downloads) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Convert instrument-specific raw files to open formats (.mzXML, .mzML, .mgf) before upload and job submission)
- **MassIVE** (Public repository where LC-MS data files and metadata are hosted and referenced by the GNPS job)
- **Cytoscape** (Visualization of the downloaded .graphml/.cys molecular network and its subnetworks/families)

## Evaluation signals

- Job outputs include both a .graphml/.cys network file and a bucket (consensus spectra) table
- Library-annotated node appears at expected m/z with default library-search cutoffs (cosine 0.7, 6 matched ions); e.g., glycocholic acid at m/z 466.316 in the MSV000083437 network
- The conjugated bile-acid family reproduces the expected node set (m/z 466.316, 556.362, 572.358, 546.309, 563.334, 482.329) as in Figs 3 and 6 of the protocol
- Parameter-sensitivity counts reproduce reported values: Minimum Consensus Cluster Size 1–4 giving stenothricin subnetwork sizes of 52/169, 29/144, 12/89, 7/73 nodes/spectra in MSV000083381
- Applied spectral filters match defaults: peaks within ±17 Da of the precursor removed and only top six most intense peaks per ±50 Da window retained

## Limitations

- Classic molecular networking provides only qualitative information because only MS2 scans are considered; calibrated quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated; most spectra remain unannotated 'dark matter'
- MS2 spectra typically cannot differentiate regio- or stereoisomers
- High-resolution settings can lose analog annotations: at PIMT/FIMT 0.03, stenothricin annotations that appear at 2.0/0.5 are lost (2 stenothricin nodes of 7,642 at MCC 1; none at MCC 2)

## Evidence

- [intro] An empirical cutoff for cosine scoring of 0.7 with 6 MS2 ions matching is the default setting in GNPS: [evidence span withheld — anchor status: not_found]
- [intro] All peaks in a ±17 Da range around the precursor ion mass are deleted: "All peaks in a ±17 Da range around the precursor ion mass are deleted"
- [supplementary] All LC–MS data used in this paper are publicly available at the GNPS-MassIVE repository: [evidence span withheld — anchor status: not_found]
- [intro] classic molecular networking can provide only qualitative information: [evidence span withheld — anchor status: not_found]
