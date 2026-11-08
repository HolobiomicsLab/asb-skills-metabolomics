---
name: processing-report-generation-and-filter-impact-summarization
description: 'Use when running a matchms spectrum-cleaning pipeline (e.g. on a GNPS,
  MassBank, or MoNA .msp/.MGF/JSON library) and you need machine-readable provenance:
  a rerunnable YAML of all filter settings plus a per-filter impact report.'
when_to_use_negative:
- You expect the report to catch wrong chemical annotations consistent with the measured
  mass — the pipeline lacks plausibility checks over both metadata and measured fragments;
  fragment-matching filters are only future work
- You need standardized instrument type or collision energy fields — current matchms
  filters do not clean/standardize these metadata
- Your goal is peak picking or raw-data preprocessing — that is handled by tools like
  MZMine, MSDial, or OpenMS, not this reporting step
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package whose Pipeline/SpectrumProcessor class runs the cleaning filters
    and automatically emits the YAML workflow file and processing report
  repo: github.com/matchms/matchms
- name: Python
  role: Language in which matchms is installed (via bioconda/PyPI) and the pipeline
    is invoked
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:6525b925fbdb87fc82942c0543c958a69133b63ea69d5b1a7a4dabaab887ee95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# processing-report-generation-and-filter-impact-summarization

## Summary

Automatic emission of two logging artifacts by the matchms library cleaning pipeline (matchms 0.26.4): a YAML file recording all pipeline settings and filters, and a processing report (processing_report.csv) quantifying how many spectra each filter removed, metadata-changed, or peak-changed — the mechanism that makes each GNPS-style cleaning run reproducible and transparent.

## When to use

Use when running a matchms spectrum-cleaning pipeline (e.g. on a GNPS, MassBank, or MoNA .msp/.MGF/JSON library) and you need machine-readable provenance: a rerunnable YAML of all filter settings plus a per-filter impact report.

## When NOT to use

- You expect the report to catch wrong chemical annotations consistent with the measured mass — the pipeline lacks plausibility checks over both metadata and measured fragments; fragment-matching filters are only future work
- You need standardized instrument type or collision energy fields — current matchms filters do not clean/standardize these metadata
- Your goal is peak picking or raw-data preprocessing — that is handled by tools like MZMine, MSDial, or OpenMS, not this reporting step

## Inputs

- Mass spectral library file (msp, MGF, mzML, mzXML, or JSON; e.g. GNPS public library, 500,569 spectra)
- matchms SpectrumProcessor pipeline configuration (library cleaning filter set)
- Matched spectrum objects with metadata (SMILES, InChI, InChIKey, adduct, parent mass)

## Outputs

- workflow.yaml — YAML file containing all settings and filters used, reusable to rerun the cleaning pipeline
- processing_report.csv — per-filter overview of spectra removed, metadata-changed, and peak-changed

## How to apply

Load the input mass spectral library (e.g. the GNPS public library downloaded 21-08-2023, 500,569 spectra) with matchms 0.26.4 and build the pipeline with SpectrumProcessor using the library cleaning filter set (metadata harmonization via basic/default filters, derivation of missing metadata such as SMILES from compound names via PubChem, annotation repair, and annotation-requirement filters). Run the pipeline over all spectra; during the run matchms automatically creates a YAML file containing all settings and filters used, which is directly reusable to rerun the pipeline unchanged. In the same run it generates a processing report giving an overview of the number of spectra changed by each filter, differentiating between completely removing spectra, changing metadata, and changing peak information. Store the artifacts as workflow.yaml and processing_report.csv; the YAML is the provenance record, the report is the audit trail of filter impact.

## Related tools

- **matchms** (Python package whose Pipeline/SpectrumProcessor class runs the cleaning filters and automatically emits the YAML workflow file and processing report) — github.com/matchms/matchms
- **Python** (Language in which matchms is installed (via bioconda/PyPI) and the pipeline is invoked)

## Examples

```
from matchms import SpectrumProcessor; pipeline = SpectrumProcessor("library-cleaning"); pipeline.run(spectra)  # emits workflow.yaml + processing_report.csv
```

## Evaluation signals

- The stored workflow.yaml can rerun the cleaning pipeline unchanged (byte-for-byte identical filter list and settings)
- processing_report.csv contains one row per applied filter with counts split into three effect classes: spectra completely removed, metadata changed, and peak information changed
- On the GNPS reference run: 500,569 input spectra, 448,485 curated spectra in the final cleaned library, run time ≈ 6 h 45 min
- Every filter in the report corresponds to a filter in the YAML file (no unrecorded filters, no recorded-but-unrun filters)
- GNPS MS/MS libraries cleaned this way every 24 h show stable report structure across successive runs

## Limitations

- Processing report examples are only available as supplementary tables (Table S1 and Table S2); Supplementary file 1 appears without content in the document
- The report only reflects filter effects on metadata and peaks — no fragment-based plausibility checking, so wrong annotations consistent with the measured mass go unnoticed
- Report reproducibility is version-dependent: matchms 0.26.4 was the version used to run the published GNPS pipelines
- The Zenodo record for the final cleaned GNPS library (reference 5) is cited but not resolved to a DOI or record number in the article text

## Evidence

- [methods] Per-filter impact report: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] YAML provenance: "it automatically creates a YAML file containing all settings and filters used."
- [results] Artifacts generated on GNPS run: "The new library cleaning pipeline was run on the GNPS public mass spectral library. A YAML file and processing report were automatically generated."
- [discussion] Report examples location: "Examples of such processing reports can be found in Table S1 and Table S2."
- [readme] Pipeline/yaml capability in README: [evidence span withheld — anchor status: external_doc]
- [results] Scale and runtime: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
