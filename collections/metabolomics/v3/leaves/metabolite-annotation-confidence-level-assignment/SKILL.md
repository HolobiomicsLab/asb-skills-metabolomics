---
name: metabolite-annotation-confidence-level-assignment
description: Use when an unassigned LC-HRMS signal (e.g. a recurrent m/z feature observed
  on multiple platforms such as Orbitrap Fusion and Q-ToF Impact II) has been characterized
  by HRMS/MS and needs a metabolite annotation with an explicit MSI confidence level.
when_to_use_negative:
- The compound has been confirmed with an authentic standard analyzed under identical
  conditions — this would be MSI level 1, and the level-2 assignment logic does not
  apply
- Only raw unprocessed vendor files are available and no cleaned, SOP-conformant spectral
  metadata exist — PeakForest requires users to supply completely-cleaned data and
  manually entered metadata
- The task is storage or reprocessing of raw spectral data rather than annotation
  reporting — PeakForest is designed for metabolite annotation report data rather
  than storage of raw spectral data
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PeakForest
  role: Open-source Java Web application hosting experimental MS and NMR spectra;
    its REST protocol-based Web services (OpenAPI v3.0 compliant) are queried to pull
    spectrum and compound records and to cross-check annotations and MSI confidence
    levels
  repo: https://github.com/peakforest/peakforest-webapp
- name: Galaxy
  role: Downstream analysis environment in which PeakForest databases can be used
    in conjunction with bespoke data analysis pipelines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:781ac76945f4e8314ba7bada77308fc51a702cf52264de7037300b6bb5091a96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Galaxy
  - PeakForest
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolite-annotation-confidence-level-assignment

## Summary

Assign Metabolomics Standards Initiative (MSI) confidence levels to LC-HRMS/MS metabolite annotations by cross-checking retrieved spectral and compound records (e.g. via PeakForest REST Web services) against published results, as done for the recurrent NIST plasma m/z 247.1441 signal annotated as hypaphorine at MSI level 2.

## When to use

Use when an unassigned LC-HRMS signal (e.g. a recurrent m/z feature observed on multiple platforms such as Orbitrap Fusion and Q-ToF Impact II) has been characterized by HRMS/MS and needs a metabolite annotation with an explicit MSI confidence level.

## When NOT to use

- The compound has been confirmed with an authentic standard analyzed under identical conditions — this would be MSI level 1, and the level-2 assignment logic does not apply
- Only raw unprocessed vendor files are available and no cleaned, SOP-conformant spectral metadata exist — PeakForest requires users to supply completely-cleaned data and manually entered metadata
- The task is storage or reprocessing of raw spectral data rather than annotation reporting — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data

## Inputs

- LC-HRMS/MS feature list with observed m/z values (e.g. m/z 247.1441, positive ion mode)
- PeakForest spectrum and compound records retrieved via REST Web services
- Published article text reporting annotations (e.g. DOI 10.1007/s11306-022-01899-3 results on NIST plasma)
- Instrument/platform provenance metadata (Orbitrap Fusion, Q-ToF Impact II)

## Outputs

- Structured annotation table (CSV, one row per platform × feature) with columns for sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, and MSI confidence level
- MSI confidence level assignments (e.g. hypaphorine, MSI level 2)

## How to apply

First, pull the spectrum and compound records for the target signal from the deposited spectral database using PeakForest's REST protocol-based Web services (OpenAPI v3.0 compliant), which support query filters such as chromatography column characteristics and basic logic gates ('AND', 'OR'). Second, cross-check each retrieved record against the published article text, verifying that the observed m/z (e.g. 247.1441 in positive ion mode) and the HRMS/MS evidence match the annotated metabolite (e.g. hypaphorine) on every instrument platform it was detected on. Third, assign the MSI confidence level based on the evidence tier: level 2 applies when a putatively annotated compound is supported by diagnostic MS/MS spectra but not by an authentic standard. Finally, compile one row per platform × feature (sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, MSI confidence level) into a structured table such as nist_plasma_lchrms_annotations.csv.

## Related tools

- **PeakForest** (Open-source Java Web application hosting experimental MS and NMR spectra; its REST protocol-based Web services (OpenAPI v3.0 compliant) are queried to pull spectrum and compound records and to cross-check annotations and MSI confidence levels) — https://github.com/peakforest/peakforest-webapp
- **Galaxy** (Downstream analysis environment in which PeakForest databases can be used in conjunction with bespoke data analysis pipelines)

## Evaluation signals

- The compiled CSV parses cleanly and contains one row for each LC-HRMS platform on which the signal was observed (Orbitrap Fusion and Q-ToF Impact II)
- The recorded values match the reported results exactly: m/z 247.1441 in positive ion mode, annotation hypaphorine, MSI confidence level 2
- Each annotation row is traceable to a PeakForest record retrieved via the REST Web services and to the corresponding statement in the article text
- The MSI level assignment is consistent with the evidence tier: level 2 indicates putative annotation supported by HRMS/MS but no authentic-standard confirmation

## Limitations

- MS–MS matching capabilities of the infrastructure still require enhancement — the authors state that 'additional developments are needed to enhance the MS–MS and NMR-2D matching capabilities'
- Metadata must be manually entered and users must supply completely-cleaned data following SOPs; PeakForest does not include data clean-up or extraction tools
- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional, and installation requires personnel with computing or bioinformatics skills

## Evidence

- [abstract] PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations.: "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [intro] metabolite annotation remains a major bottleneck in untargeted mass spectrometry (MS) and nuclear magnetic resonance (NMR) metabolomics: "metabolite annotation remains a major bottleneck in untargeted mass spectrometry (MS) and nuclear magnetic resonance (NMR) metabolomics"
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [results] A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type: "A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type"
- [discussion] additional developments are needed to enhance the MS–MS and NMR-2D matching capabilities: "additional developments are needed to enhance the MS–MS and NMR-2D matching capabilities"
