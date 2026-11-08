---
name: accurate-mass-feature-matching-across-platforms
description: Use when an unassigned LC-HRMS feature recurs across instruments/platforms
  (e.g. positive-mode NIST plasma on Orbitrap Fusion and Q-ToF Impact II) and needs
  annotation to a named metabolite with an MSI confidence level.
when_to_use_negative:
- You need to store or clean up raw spectral data — PeakForest is designed for metabolite
  annotation report data rather than storage of raw spectral data and does not include
  data clean-up or extraction tools
- Your spectra are not yet in a PeakForest database — a minimum number of compounds
  and spectra need to be imported before local PeakForest databases are fully functional
- The feature cannot be linked to external spectral resources or matching tools —
  annotation in biological matrices relies on external peak matching tools
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: PeakForest
  role: Hosts the experimental MS spectrum and compound records; its REST protocol-based
    Web services (OpenAPI v3.0) are queried programmatically to pull the m/z 247.1441
    feature and hypaphorine records, with filter support for column characteristics
    and AND/OR logic-gate queries
  repo: https://github.com/peakforest
- name: Galaxy
  role: PeakForest databases can be used in conjunction with bespoke data analysis
    pipelines in the Galaxy environment for downstream analysis of pulled annotation
    records
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:9fafa75d2b6fcd2c706c1eb00f7ace56a0b8c4ccc61d73c70b5c5429e68c6afb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - PeakForest
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Accurate-mass feature matching across platforms

## Summary

Matches a recurrent unassigned LC-HRMS feature (e.g. m/z 247.1441 in positive-mode NIST plasma acquired on Orbitrap Fusion and Q-ToF Impact II) to a metabolite annotation (hypaphorine, MSI level 2) by pulling spectrum and compound records via PeakForest REST Web services and cross-checking against the published article results.

## When to use

Use when an unassigned LC-HRMS feature recurs across instruments/platforms (e.g. positive-mode NIST plasma on Orbitrap Fusion and Q-ToF Impact II) and needs annotation to a named metabolite with an MSI confidence level.

## When NOT to use

- You need to store or clean up raw spectral data — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data and does not include data clean-up or extraction tools
- Your spectra are not yet in a PeakForest database — a minimum number of compounds and spectra need to be imported before local PeakForest databases are fully functional
- The feature cannot be linked to external spectral resources or matching tools — annotation in biological matrices relies on external peak matching tools

## Inputs

- Recurrent unassigned LC-HRMS feature m/z 247.1441 observed in positive mode on Orbitrap Fusion and Q-ToF Impact II
- HRMS/MS spectra of NIST plasma acquired on both instruments
- PeakForest demonstrator database records (spectrum and compound entries) accessible via REST Web services (OpenAPI v3.0)
- Source article (DOI 10.1007/s11306-022-01899-3) results text

## Outputs

- nist_plasma_lchrms_annotations.csv with one row per platform × feature, columns: sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, MSI confidence level
- Verified annotation of m/z 247.1441 as hypaphorine at MSI confidence level 2

## How to apply

Locate the results section of the source article reporting LC-HRMS(+MS/MS) positive-mode annotation of NIST plasma on both the Orbitrap Fusion and the Q-ToF Impact II, then open the deposited demonstrator/supplementary data and the public PeakForest demonstrator at peakforest.org. Use the PeakForest REST protocol-based Web services (OpenAPI v3.0 compliant) to programmatically pull the spectrum and compound records for the target feature — here m/z 247.1441 in positive ion mode — and for the candidate metabolite hypaphorine; the Web service supports filters such as chromatography column characteristics and an advanced search filter system with 'AND'/'OR' logic gates. Cross-check each retrieved record against the article text: the recurrent unassigned signal at m/z 247.1441 observed on both platforms and the hypaphorine annotation at Metabolomics Standards Initiative confidence level 2. Compile one row per platform × feature into a structured CSV with columns for sample matrix, LC-HRMS platform, ionization mode, observed m/z, annotation, and MSI confidence level. The rationale is that PeakForest stores experimental MS spectral metadata and signal annotations in a structured, FAIR-aligned database whose Web services make annotation records machine-readable, addressing the bottleneck of spectral data annotation and metabolite identification across multiple instruments.

## Related tools

- **PeakForest** (Hosts the experimental MS spectrum and compound records; its REST protocol-based Web services (OpenAPI v3.0) are queried programmatically to pull the m/z 247.1441 feature and hypaphorine records, with filter support for column characteristics and AND/OR logic-gate queries) — https://github.com/peakforest
- **Galaxy** (PeakForest databases can be used in conjunction with bespoke data analysis pipelines in the Galaxy environment for downstream analysis of pulled annotation records)

## Examples

```
curl -X GET "https://peakforest.org/rest/spectra/ms/search?mz=247.1441&mode=positive" -H "Accept: application/json"
```

## Evaluation signals

- nist_plasma_lchrms_annotations.csv parses cleanly as CSV
- The CSV contains rows for both the Orbitrap Fusion and the Q-ToF Impact II platforms
- Recorded m/z value matches the reported 247.1441 in every row
- Annotation column reads 'hypaphorine' and MSI confidence level is 2 in all rows
- Retrieved PeakForest records are consistent with the article's reported results for the recurrent unassigned signal on both platforms

## Limitations

- PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database
- Users must supply completely-cleaned data following SOPs edited by the PeakForest team
- A minimum number of compounds and spectra need to be imported by users in order to make local PeakForest databases fully-functional
- Additional developments are needed to enhance the MS–MS and NMR-2D matching capabilities
- Technical installation requires personnel with computing or bioinformatics skills

## Evidence

- [results] hypaphorine annotation at MSI level 2 across two instruments: [evidence span withheld — anchor status: not_found]
- [abstract] PeakForest REST Web services queryable programmatically: "We developed the PeakForest infrastructure, an open-source Java tool with automatic programming interfaces that can be deployed locally"
- [results] Web service filter system with logic gates: "A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type"
- [abstract] PeakForest captures signal annotations for metabolite identification: "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [readme] Webapp build and deployment route (README): "Build command `mvn clean install` produce:
   - `peakforest-webapp-X.war` (java bin file, ready for a tomcat server deploy"
