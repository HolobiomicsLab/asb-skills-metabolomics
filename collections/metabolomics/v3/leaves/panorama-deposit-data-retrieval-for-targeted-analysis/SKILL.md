---
name: panorama-deposit-data-retrieval-for-targeted-analysis
description: Use when a publication states its Skyline documents and raw LC-MS data
  were shared via Panorama and you need the underlying SRM/HRMS vendor files to reproduce,
  validate, or extend the targeted metabolomics analysis.
when_to_use_negative:
- The data of interest are non-targeted feature tables already processed by XCMS/MZmine/MS-DIAL
  rather than vendor raw files or Skyline documents
- Only the fragment-ion spectra of the library entries are needed and no quantitative
  re-analysis of raw data is intended
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: 'Consumes the downloaded Panorama deposit: opens the .sky document and imports
    the vendor raw files for targeted small-molecule processing'
  repo: github.com/ProteoWizard/pwiz
- name: Panorama Public
  role: Web-based targeted method and data repository from which the raw files and
    Skyline documents are retrieved
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:5b7b94e28f66bf812a245b37f6e105c6806d53cc209d88101f35c2bcbebea884
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HMDB
  - METLIN
  - MassBank
  - Panorama Public
  - Skyline
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# panorama-deposit-data-retrieval-for-targeted-analysis

## Summary

Retrieve vendor raw files and associated Skyline documents from a Panorama Public deposit (e.g. the SRM and HRMS small-molecule data in this study) so that targeted quantitative small-molecule analysis can be reproduced or extended in Skyline.

## When to use

Use when a publication states its Skyline documents and raw LC-MS data were shared via Panorama and you need the underlying SRM/HRMS vendor files to reproduce, validate, or extend the targeted metabolomics analysis.

## When NOT to use

- The data of interest are non-targeted feature tables already processed by XCMS/MZmine/MS-DIAL rather than vendor raw files or Skyline documents
- Only the fragment-ion spectra of the library entries are needed and no quantitative re-analysis of raw data is intended

## Inputs

- Panorama Public deposit URL (e.g. https://panoramaweb.org/) for the study
- Vendor raw LC-MS files (SRM and HRMS acquisitions from multiple instrument vendors)
- Associated Skyline .sky/.sky.zip documents containing transition lists and calibration settings

## Outputs

- Locally stored vendor raw files ready for import into Skyline
- Skyline document with imported raw data, transition list, and retention-time-scheduled method settings
- Reproducible targeted quantification results (peak areas, concentrations) for small-molecule targets

## How to apply

Locate the Panorama Public accession or https://panoramaweb.org/ URL cited in the article (this study deposits raw files alongside processed Skyline documents). Download the vendor raw LC-MS files and the .sky/.sky.zip documents; note that Skyline's automated data import workflow, extended from Panorama-enabled peptide workflows to small molecules, can batch-import these raw files into the document for processing. Configure the downloaded Skyline document's transition settings for small molecules (SRM or HRMS extraction as appropriate) and proceed with peak integration and calibration as described in the paper's step-by-step instructions. Rationale: Panorama is a web-based targeted method and data repository designed to keep raw data, methods, and Skyline results linked, enabling vendor-independent analysis of data acquired on six major instrument vendors supported by Skyline.

## Related tools

- **Skyline** (Consumes the downloaded Panorama deposit: opens the .sky document and imports the vendor raw files for targeted small-molecule processing) — github.com/ProteoWizard/pwiz
- **Panorama Public** (Web-based targeted method and data repository from which the raw files and Skyline documents are retrieved) — https://panoramaweb.org/

## Evaluation signals

- The downloaded deposit contains both raw files and a Skyline document; opening the .sky file in Skyline lists the expected small-molecule targets from the study (e.g. serum amino acids from the ADNI/Biocrates p180 dataset)
- Number of imported chromatograms equals the number of raw files in the deposit after Skyline's automated data import
- Retention-time-scheduled method regenerated from the document matches the 1-minute scheduling window described in the paper
- Peak integration can be manually inspected in Skyline for a subset of targets and agrees with published results

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows
- All Panorama-enabled workflows for peptides, such as automated data import and AutoQC, have only recently been extended to small molecules and may not cover every scenario

## Evidence
