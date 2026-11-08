---
name: spectral-library-cleaning-pipeline-configuration
description: Use when a public MS/MS spectral library (e.g. GNPS msp, MassBank, MoNA)
  has inconsistent or missing metadata, missing/broken SMILES-InChI-InChIKey annotations,
  or parent mass/adduct mismatches that must be repaired or used to remove spectra
  before library searching or ML training.
when_to_use_negative:
- The goal is spectral similarity scoring or library searching itself, not cleaning
  the library beforehand — this skill only harmonizes/repairs/validates metadata and
  discards spectra.
- 'Wrong chemical annotations consistent with the measured parent mass are the target
  of the cleaning: the pipeline lacks plausibility checks that consider both metadata
  and measured fragments, so such errors go unnoticed and another validation approach
  is needed.'
- Standardization of fields like instrument type or collision energies is required
  — current matchms filters do not clean or standardize these metadata fields.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the Pipeline/SpectrumProcessor classes, the library-cleaning filter
    set in a fixed global order, and automatic YAML/processing-report generation;
    version 0.26.4 was used.
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and cross-compares SMILES, InChI and InChIKey annotations to validate
    structure annotations and remove incomplete/mismatching spectra.
- name: Python
  role: Runtime language in which matchms imports, processes, cleans, and exports
    the spectral libraries.
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:8c75320773b043d9bdb557366f4e6407a58e62a13fdcc28b690e0558099b9599
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectral-library-cleaning-pipeline-configuration

## Summary

Configure and run the matchms 0.26.4 library-cleaning pipeline (SpectrumProcessor/Pipeline) that harmonizes metadata, repairs structure annotations (SMILES/InChI/InChIKey via RDKit and PubChem), and discards spectra with invalid annotations from public MS/MS libraries such as GNPS, MassBank, and MoNA, with reproducibility captured in an auto-generated YAML file and processing report.

## When to use

Use when a public MS/MS spectral library (e.g. GNPS msp, MassBank, MoNA) has inconsistent or missing metadata, missing/broken SMILES-InChI-InChIKey annotations, or parent mass/adduct mismatches that must be repaired or used to remove spectra before library searching or ML training.

## When NOT to use

- The goal is spectral similarity scoring or library searching itself, not cleaning the library beforehand — this skill only harmonizes/repairs/validates metadata and discards spectra.
- Wrong chemical annotations consistent with the measured parent mass are the target of the cleaning: the pipeline lacks plausibility checks that consider both metadata and measured fragments, so such errors go unnoticed and another validation approach is needed.
- Standardization of fields like instrument type or collision energies is required — current matchms filters do not clean or standardize these metadata fields.

## Inputs

- Raw public mass spectral library file (e.g. GNPS public library msp, 500,569 spectra as downloaded 21-08-2023; MassBank; MoNA) importable by matchms (mzML, mzXML, msp, MGF, JSON)
- Pipeline configuration selecting the library-cleaning filter set (basic + default filters plus repair and require-annotation filters in fixed global order)
- matchms 0.26.4 environment with RDKit for structure-annotation validation; PubChem access for name-derived annotations

## Outputs

- Cleaned mass spectral library file (e.g. 448,485 curated GNPS spectra)
- Automatically generated YAML workflow file containing all settings and filters used
- Processing report giving per-filter counts of removed spectra, changed metadata, and changed peaks (comparable to supplementary Tables S1-S5)

## How to apply

Assemble the cleaning configuration from matchms' filter sets: the library-cleaning pipeline runs all default filters (basic metadata harmonization plus deriving missing metadata) and additionally the repair filters (Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, Derive Annotation From Compound Name) plus Require Valid Annotation, in a fixed global filter order. Annotations are validated by loading SMILES, InChI and InChIKey with RDKit and comparing them; Derive Annotation From Compound Name fetches canonical SMILES/InChI/InChIKey from PubChem only when the derived structure matches the given parent mass. Run the pipeline via the matchms Pipeline class (or a custom SpectrumProcessor for ablation runs, e.g. omitting the repair filters), then let it auto-generate a YAML file with all settings and a processing report enumerating per-filter counts of removed spectra and changed metadata/peaks. A reference run on the GNPS library (500,569 spectra, downloaded 21-08-2023) with repair functions took 6 h 45 min, removed 31,758 spectra, and yielded 448,485 curated spectra; the identical YAML configuration can be reused unchanged on MassBank and MoNA.

## Related tools

- **matchms** (Provides the Pipeline/SpectrumProcessor classes, the library-cleaning filter set in a fixed global order, and automatic YAML/processing-report generation; version 0.26.4 was used.) — https://github.com/matchms/matchms
- **RDKit** (Loads and cross-compares SMILES, InChI and InChIKey annotations to validate structure annotations and remove incomplete/mismatching spectra.)
- **Python** (Runtime language in which matchms imports, processes, cleans, and exports the spectral libraries.)

## Examples

```
from matchms import Pipeline; pipeline = Pipeline("library_cleaning.yaml"); pipeline.run()
```

## Evaluation signals

- The run automatically produced both a YAML workflow file capturing all settings/filters and a processing report enumerating filters in the fixed global order with per-filter spectrum-change counts.
- For the GNPS reference run, the total spectra removed equals 31,758 and the cleaned library contains 448,485 spectra out of the original 500,569.
- For a no-repair ablation run (custom SpectrumProcessor without the five repair filters), the removal count in the report must be strictly greater than 31,758 (reference value 83,843).
- Every retained spectrum passes Require Valid Annotation: its SMILES, InChI and InChIKey load in RDKit and are mutually consistent.
- Re-running the exported YAML configuration on MassBank and MoNA yields cleaned library files plus reports comparable to the supplementary report tables (Tables S3-S5).

## Limitations

- Filters do not perform plausibility checks combining metadata with measured fragments, so chemically wrong annotations consistent with the measured mass go unnoticed; fragment-matching filters are only a proposed future expansion.
- Instrument type and collision energies are potentially important fields that are not yet standardized by the current filters.
- SMILES is derived from compound names via PubChem only if it matches the given parent mass, so name-only annotations may remain unrepaired when masses disagree; a validator tool for curation at upload time remains unbuilt future work.

## Evidence

- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] yaml reproducibility: "it automatically creates a YAML file containing all settings and filters used."
- [methods] processing report: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [readme] Pipeline class: [evidence span withheld — anchor status: external_doc]
- [results] runtime cost of reference run: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
