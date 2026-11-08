---
name: subnetwork-node-count-tabulation
description: Use when a GNPS molecular networking job has completed and you need to
  quantify how a parameter sweep (Minimum Consensus Cluster Size, PIMT/FIMT tolerance,
  filter settings) changes the node/spectrum composition of a specific subnetwork.
when_to_use_negative:
- The input is raw instrument data or an unsubmitted dataset — conversion (mzXML/mzML/mgf),
  upload, and job submission must happen first; this skill only tabulates results
  of completed networking jobs.
- The goal is calibrated quantitation — classic molecular networking considers only
  MS2 scans and 'can provide only qualitative information about the experiment'; targeted
  follow-up LC-MS is required instead.
- The analysis targets in silico structural annotation rather than subnetwork size/yield
  tabulation — 'the application of such tools is beyond the immediate scope of the
  networking protocol presented here'.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: GNPS
  role: Platform on which METABOLOMICS-SNETS-V2 molecular networking jobs are launched
    and from which per-job node/subnetwork results are tabulated
  repo: https://ccms-ucsd.github.io/GNPSDocumentation
- name: GNPS_Workflows
  role: Repository containing the METABOLOMICS-SNETS-V2 workflow definition used for
    each parameter-sweep job
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:2225fe82cc9d87c3ae41f26cd2e58321883ac5d695258182e76f3d112658cae7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - GNPS_Workflows
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Subnetwork node count tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates node and consensus-spectrum counts for individual subnetworks (e.g., the stenothricin-containing subnetwork) across GNPS METABOLOMICS-SNETS-V2 jobs run under different parameter settings, producing a parameter-sensitivity table showing how Minimum Consensus Cluster Size, ion mass tolerances, and spectrum filters reshape network structure.

## When to use

Use when a GNPS molecular networking job has completed and you need to quantify how a parameter sweep (Minimum Consensus Cluster Size, PIMT/FIMT tolerance, filter settings) changes the node/spectrum composition of a specific subnetwork.

## When NOT to use

- The input is raw instrument data or an unsubmitted dataset — conversion (mzXML/mzML/mgf), upload, and job submission must happen first; this skill only tabulates results of completed networking jobs.
- The goal is calibrated quantitation — classic molecular networking considers only MS2 scans and 'can provide only qualitative information about the experiment'; targeted follow-up LC-MS is required instead.
- The analysis targets in silico structural annotation rather than subnetwork size/yield tabulation — 'the application of such tools is beyond the immediate scope of the networking protocol presented here'.

## Inputs

- Completed GNPS METABOLOMICS-SNETS-V2 molecular networking jobs on MassIVE dataset MSV000083381 (one job per parameter setting)
- GNPS networking job parameter records (Minimum Consensus Cluster Size, Precursor/Fragment Ion Mass Tolerance, filter settings)
- GNPS networking results (node/cluster annotations, including spectral library matches to stenothricin analogs)

## Outputs

- fig7_sensitivity_table.csv with per-setting node and consensus-spectrum counts for the target subnetwork and total network
- Subnetwork node/spectrum counts per Minimum Consensus Cluster Size setting (MCC 1–4)
- Filtering comparison counts (filters on vs. off) and high-resolution tolerance comparison counts (PIMT/FIMT 0.03 vs. 2.0/0.5)

## How to apply

Run one GNPS METABOLOMICS-SNETS-V2 job per parameter setting on the same LC-MS/MS input files (e.g., MassIVE dataset MSV000083381): sweep Minimum Consensus Cluster Size 1–4 while holding Precursor Ion Mass Tolerance at 2.0 and Fragment Ion Mass Tolerance at 0.5 with spectrum filtering disabled; run an advanced-filtering job under otherwise equivalent parameters; run a high-resolution job with PIMT/FIMT at 0.03. For each completed job, count the nodes in the subnetwork of interest (identified via spectral library annotations such as stenothricin analogs) and the total consensus spectra in the network, then assemble all per-setting counts into a single sensitivity table (e.g., fig7_sensitivity_table.csv). The rationale is that consensus clustering (MCC) and mass tolerance settings directly control which MS2 spectra merge into nodes and whether library-matched analogs are retained, so per-subnetwork counts expose these effects quantitatively.

## Related tools

- **GNPS** (Platform on which METABOLOMICS-SNETS-V2 molecular networking jobs are launched and from which per-job node/subnetwork results are tabulated) — https://ccms-ucsd.github.io/GNPSDocumentation
- **GNPS_Workflows** (Repository containing the METABOLOMICS-SNETS-V2 workflow definition used for each parameter-sweep job) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
open 'https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22'  # launch one METABOLOMICS-SNETS-V2 job per MCC/filter/tolerance setting on MSV000083381 files
```

## Evaluation signals

- MCC sweep must reproduce 52/169, 29/144, 12/89, 7/73 nodes/spectra for the stenothricin subnetwork at MCC 1–4 (PIMT/FIMT 2.0/0.5, filtering off)
- Advanced filtering comparison must give 9 nodes with filters enabled vs. 26 nodes without
- PIMT/FIMT 0.03 at MCC 1 must give only 2 stenothricin nodes of 7,642 total spectra; at MCC 2 no stenothricin subnetwork among 192 nodes
- Each tabulated row must be traceable to one completed GNPS job with its exact parameter settings recorded
- Node counts in the table must match per-job counts obtained by inspecting the subnetwork in the GNPS viewer or Cytoscape

## Limitations

- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated, so subnetwork identification depends on sparse library matches; many nodes remain 'dark matter'
- Stringent mass tolerances (PIMT/FIMT 0.03) can lose entire stenothricin analog subnetworks that are recovered at low-resolution settings (2.0/0.5)
- Classic molecular networking provides only qualitative information (MS2 scans only); no calibrated quantitation
- Statistics were not performed for the tutorial protocol example, and MS2 spectra typically cannot differentiate regio- or stereoisomers

## Evidence

- [supplementary] Data availability: [evidence span withheld — anchor status: not_found]
- [intro] Qualitative-only caveat: [evidence span withheld — anchor status: not_found]
