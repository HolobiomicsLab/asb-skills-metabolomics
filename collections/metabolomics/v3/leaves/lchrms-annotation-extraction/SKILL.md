---
name: lchrms-annotation-extraction
description: Use when a recurring unassigned LC-HRMS peak (e.g. a specific m/z in
  positive mode, such as m/z 247.1441 in NIST plasma) must be resolved to a metabolite
  annotation and its MSI confidence level across instruments.
when_to_use_negative:
- Input is already a curated feature-annotation table with confirmed metabolite identities
  and MSI levels rather than raw or partially annotated spectral records
- The task requires raw spectral data storage or data clean-up/extraction from vendor
  files — PeakForest stores metadata related to raw data rather than raw data itself
  and 'does not include data clean-up or extraction tools per se'
- The spectra are NMR rather than MS, or require 2D NMR matching or MS-MS matching
  capabilities beyond current PeakForest matching features
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakForest
  role: Hosts the MS/compound records and exposes REST protocol-based Web services
    (OpenAPI v3.0) used to pull spectrum and compound records for the target m/z and
    metabolite in positive ion mode; chromatography column characteristics can be
    used as filters
  repo: https://github.com/peakforest
- name: Galaxy
  role: Downstream environment in which PeakForest databases can be used with bespoke
    data analysis pipelines after annotation extraction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:f54a4d1e17b7fdbb6506f4a09294d924e114ec3e49447cea62bea95d2243e366
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - PeakForest
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lchrms-annotation-extraction

## Summary

Extracts and cross-checks LC-HRMS/MS metabolite annotations (observed m/z, ionization mode, metabolite identity, MSI confidence level) for a recurrent unassigned signal across multiple instruments, using the PeakForest Web services and the source article as ground truth.

## When to use

Use when a recurring unassigned LC-HRMS peak (e.g. a specific m/z in positive mode, such as m/z 247.1441 in NIST plasma) must be resolved to a metabolite annotation and its MSI confidence level across instruments.

## When NOT to use

- Input is already a curated feature-annotation table with confirmed metabolite identities and MSI levels rather than raw or partially annotated spectral records
- The task requires raw spectral data storage or data clean-up/extraction from vendor files — PeakForest stores metadata related to raw data rather than raw data itself and 'does not include data clean-up or extraction tools per se'
- The spectra are NMR rather than MS, or require 2D NMR matching or MS-MS matching capabilities beyond current PeakForest matching features

## Inputs

- Positive-mode LC-HRMS and HRMS/MS annotation records for the target signal (e.g. m/z 247.1441 in NIST plasma)
- PeakForest database records: spectrum and compound entries queried via REST/OpenAPI v3.0 Web services
- Source article text reporting the annotation and its Metabolomics Standards Initiative confidence level

## Outputs

- Structured CSV table (e.g. nist_plasma_lchrms_annotations.csv) with one row per platform x feature and columns: sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, MSI confidence level
- Cross-checked annotation record pairing the observed m/z with the metabolite identity and MSI level (e.g. hypaphorine, MSI level 2)

## How to apply

Retrieve the article and locate the LC-HRMS(+MS/MS) positive-mode annotation results for the matrix of interest (e.g. NIST plasma acquired on the Orbitrap Fusion and the Q-ToF Impact II). Open the deposited demonstrator/supplementary data or the public PeakForest demonstrator, and use the PeakForest REST protocol-based Web services (OpenAPI v3.0 compliant) to programmatically pull spectrum and compound records for the target m/z signal and candidate metabolite (e.g. hypaphorine) in positive ion mode; chromatographic characteristics such as column brand, type, name, length and diameter can be used as filters, and the advanced search supports 'AND'/'OR' logic gates. Cross-check each retrieved record against the article text before accepting it. Compile one row per platform x feature into a structured table (columns: sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, MSI confidence level) and validate that the CSV parses cleanly, contains a row per instrument, and matches the reported values.

## Related tools

- **PeakForest** (Hosts the MS/compound records and exposes REST protocol-based Web services (OpenAPI v3.0) used to pull spectrum and compound records for the target m/z and metabolite in positive ion mode; chromatography column characteristics can be used as filters) — https://github.com/peakforest
- **Galaxy** (Downstream environment in which PeakForest databases can be used with bespoke data analysis pipelines after annotation extraction)

## Examples

```
curl -X GET "https://peakforest.org/rest/spectra/ms/search?mode=positive& mz=247.1441"
```

## Evaluation signals

- The output CSV parses cleanly with no malformed rows
- The CSV contains one row for each LC-HRMS platform (Orbitrap Fusion and Q-ToF Impact II)
- Recorded observed m/z matches the reported value (m/z 247.1441) for the recurrent positive-mode signal
- Annotation and confidence recorded match the article: hypaphorine at Metabolomics Standards Initiative level 2
- Each PeakForest record retrieved via the Web services agrees with the corresponding statement in the article text before being accepted

## Limitations

- PeakForest stores metadata related to raw data rather than raw data itself; users must supply completely-cleaned data following SOPs, and metadata must be manually entered
- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional, and initial database construction takes time
- Additional developments are needed to enhance the MS-MS and NMR-2D matching capabilities; ion mobility CCS and UV-Vis spectra are not yet integrated
- Installing a local PeakForest instance requires personnel with computing or bioinformatics skills

## Evidence

- [results] recurrent m/z 247.1441 signal annotated as hypaphorine: [evidence span withheld — anchor status: not_found]
- [abstract] PeakForest Web services used for annotation reuse: "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [results] Web service filters on chromatography columns: [evidence span withheld — anchor status: not_found]
- [methods] PeakForest stores metadata, not raw data: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [discussion] No built-in clean-up/extraction tools: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
