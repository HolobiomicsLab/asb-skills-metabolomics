---
name: lc-msms-spectral-clustering-parameter-tuning
description: Use when an LC-MS/MS dataset (mzML/mzXML/mgf uploaded to MassIVE) must
  be molecular-networked and the practitioner needs to know how clustering and tolerance
  settings affect node counts and annotation yield for a compound class of interest.
when_to_use_negative:
- When calibrated quantitation is needed — classic molecular networking is qualitative
  because only MS2 scans are considered
- When regio- or stereoisomers must be differentiated — MS2 spectra typically cannot
  resolve these
- When the goal is in-silico structure annotation of unannotated spectra, which is
  beyond the scope of this networking protocol
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GNPS (METABOLOMICS-SNETS-V2)
  role: Runs the classic molecular networking jobs under each parameter sweep (MCC
    1-4, PIMT/FIMT 2.0/0.5 vs 0.03, filtering on/off)
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw LC-MS/MS files to open formats (.mzXML, .mzML,
    .mgf) before upload to MassIVE
  repo: http://proteowizard.sourceforge.net/downloads.shtml
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:717e1461793bd06cfbd1bccd2b45593ff0c30f0303fd0f08f287db5e4cecf9fa
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
  - GNPS (METABOLOMICS-SNETS-V2)
  - MSConvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-msms-spectral-clustering-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic tuning of GNPS classic molecular networking (METABOLOMICS-SNETS-V2) parameters — Minimum Consensus Cluster Size, precursor/fragment ion mass tolerances, and spectrum filtering — to understand how clustering stringency changes subnetwork size and library-annotation yield in untargeted LC-MS/MS data.

## When to use

Use when an LC-MS/MS dataset (mzML/mzXML/mgf uploaded to MassIVE) must be molecular-networked and the practitioner needs to know how clustering and tolerance settings affect node counts and annotation yield for a compound class of interest.

## When NOT to use

- When calibrated quantitation is needed — classic molecular networking is qualitative because only MS2 scans are considered
- When regio- or stereoisomers must be differentiated — MS2 spectra typically cannot resolve these
- When the goal is in-silico structure annotation of unannotated spectra, which is beyond the scope of this networking protocol

## Inputs

- LC-MS/MS data files (raw converted to .mzXML/.mzML/.mgf) from a MassIVE accession such as MSV000083381
- Sample information (metadata) table collated for the dataset
- GNPS networking job parameters (Minimum Consensus Cluster Size, PIMT/FIMT, filtering toggles)

## Outputs

- Molecular network per parameter setting (viewable in the GNPS web browser or Cytoscape)
- Per-subnetwork node and total-consensus-spectra counts for each parameter combination
- Sensitivity table (fig7_sensitivity_table.csv) comparing settings

## How to apply

Retrieve the LC-MS/MS files from the GNPS-MassIVE repository (e.g., MSV000083381) and launch one GNPS METABOLOMICS-SNETS-V2 job per parameter setting. Sweep Minimum Consensus Cluster Size from 1 to 4 while holding Precursor Ion Mass Tolerance at 2.0 and Fragment Ion Mass Tolerance at 0.5 with spectrum filtering disabled; then run comparison jobs with advanced filtering enabled and with stringent PIMT/FIMT of 0.03. For each completed job, count nodes and total consensus spectra in the subnetwork of interest (e.g., the stenothricin-containing subnetwork) and assemble the counts into a sensitivity table. Rationale: relaxed tolerances (2.0/0.5) and low cluster sizes preserve rare analog annotations that stringent settings (0.03) lose, while default library matching still applies cosine 0.7 with 6 matched MS2 ions and spectrum filtering (±17 Da precursor window deletion, top-6 peaks per 50 Da window).

## Related tools

- **GNPS (METABOLOMICS-SNETS-V2)** (Runs the classic molecular networking jobs under each parameter sweep (MCC 1-4, PIMT/FIMT 2.0/0.5 vs 0.03, filtering on/off)) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw LC-MS/MS files to open formats (.mzXML, .mzML, .mgf) before upload to MassIVE) — http://proteowizard.sourceforge.net/downloads.shtml

## Examples

```
https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22METABOLOMICS-SNETS-V2%22
```

## Evaluation signals

- Stenothricin subnetwork node/spectra counts must reproduce MCC 1-4 values of 52/169, 29/144, 12/89, 7/73
- Advanced filtering comparison must yield 9 nodes vs 26 without filtering
- PIMT/FIMT 0.03 at MCC 1 must give only 2 stenothricin nodes among 7,642 total spectra, and no stenothricin subnetwork among 192 nodes at MCC 2
- Library-search defaults unchanged: cosine score cutoff 0.7 with 6 matching MS2 ions
- Job outputs should be clonable/shareable GNPS tasks whose spectra resolve via metabolomics USI URLs

## Limitations

- Only ~5-6% of MS2 spectra in untargeted metabolomics are annotated; most remain unannotated 'dark matter'
- Stringent mass tolerances (0.03) can lose stenothricin analog annotations entirely at higher cluster sizes
- High Minimum Consensus Cluster Size values collapse subnetworks (7 nodes at MCC 4 vs 52 at MCC 1), hiding low-abundance analogs

## Evidence

- [intro] library match thresholds: [evidence span withheld — anchor status: not_found]
- [supplementary] dataset availability: [evidence span withheld — anchor status: not_found]
- [intro] qualitative-only scope: [evidence span withheld — anchor status: not_found]
