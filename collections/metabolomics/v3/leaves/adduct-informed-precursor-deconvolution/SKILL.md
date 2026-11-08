---
name: adduct-informed-precursor-deconvolution
description: Use when a pre-processed LC–MS/MS feature table (featureXML/mzML) with
  adduct information needs molecular formula and structural assignments for downstream
  GNPS networking or graph-view integration.
when_to_use_negative:
- Input is a raw (unconverted) vendor file — file conversion or pre-processing must
  run first to produce featureXML/mzML with assigned MS2 spectra.
- Feature table lacks adduct/ionisation-mode information — SiriusAdapter ranking of
  formula candidates depends on the adduct information supplied from config.yaml.
- Goal is MSI level 2 library-matched identifications rather than formula/structure
  prediction — use the spectral-matching step against an MGF/MSP library instead.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: UmetaFlow
  role: workflow that orchestrates the SiriusAdapter step between re-quantification
    and GNPS export
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: SIRIUS
  role: ranks molecular formula candidates from feature-assigned MS2 spectra and adduct
    information
- name: CSI:FingerID
  role: generates structural predictions from the top-ranked SIRIUS formula candidates
- name: OpenMS
  role: provides SiriusAdapter and the underlying pyOpenMS bindings used in the Jupyter
    implementation
- name: pyOpenMS
  role: Python bindings through which the Jupyter notebook version calls the OpenMS/SIRIUS
    algorithms
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Snakemake
  role: rules engine that chains the SiriusAdapter step after re-quantification in
    the Snakemake implementation
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:cc1925f9f56856cd34962be6df1847f009ed3df2c8c0368da0e39e5f37cf4f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - CSI:FingerID
  - OpenMS
  - SIRIUS
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# adduct-informed-precursor-deconvolution

## Summary

Feeds feature-resolved MS2 spectra and adduct annotations from the UmetaFlow feature matrix into SIRIUS/CSI:FingerID via SiriusAdapter, then maps the top-ranked formula and structure predictions back onto unique feature IDs to produce MSI level 3 annotations.

## When to use

Use when a pre-processed LC–MS/MS feature table (featureXML/mzML) with adduct information needs molecular formula and structural assignments for downstream GNPS networking or graph-view integration.

## When NOT to use

- Input is a raw (unconverted) vendor file — file conversion or pre-processing must run first to produce featureXML/mzML with assigned MS2 spectra.
- Feature table lacks adduct/ionisation-mode information — SiriusAdapter ranking of formula candidates depends on the adduct information supplied from config.yaml.
- Goal is MSI level 2 library-matched identifications rather than formula/structure prediction — use the spectral-matching step against an MGF/MSP library instead.

## Inputs

- feature table / feature matrix from UmetaFlow pre-processing and re-quantification (featureXML)
- mzML files containing feature-assigned MS2 spectra
- ionisation mode and adduct information from config.yaml

## Outputs

- sirius_annotations.tsv — feature matrix annotated with highest-ranked SIRIUS formula and CSI:FingerID structure predictions (MSI level 3)
- top-ranked formula candidates and structural predictions keyed by unique feature IDs

## How to apply

Load the feature table and feature-filtered MS2 spectra produced by upstream pre-processing/re-quantification, together with ionisation mode and adduct information from config.yaml. Invoke SiriusAdapter (OpenMS/SIRIUS) on each feature's MS2 spectrum with the adduct information so that only MS2 spectra assigned to a feature are computed — SIRIUS builds an internal .ms temporary file for this. Forward the top-ranked formula candidates to CSI:FingerID for structural predictions, then map the highest-ranked SIRIUS formula and CSI:FingerID structure predictions back onto unique feature IDs in the feature matrix, classifying them as Metabolomics Standards Initiative level 3 annotations. Write the annotated feature matrix as sirius_annotations.tsv for downstream GNPS networking and graph-view integration.

## Related tools

- **UmetaFlow** (workflow that orchestrates the SiriusAdapter step between re-quantification and GNPS export) — https://github.com/biosustain/snakemake_UmetaFlow
- **SIRIUS** (ranks molecular formula candidates from feature-assigned MS2 spectra and adduct information)
- **CSI:FingerID** (generates structural predictions from the top-ranked SIRIUS formula candidates)
- **OpenMS** (provides SiriusAdapter and the underlying pyOpenMS bindings used in the Jupyter implementation)
- **pyOpenMS** (Python bindings through which the Jupyter notebook version calls the OpenMS/SIRIUS algorithms) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Snakemake** (rules engine that chains the SiriusAdapter step after re-quantification in the Snakemake implementation) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
python 5_SIRIUS_CSI.ipynb
```

## Evaluation signals

- sirius_annotations.tsv exists and every row maps a top-ranked formula and structure prediction to a unique feature ID.
- Annotated features are tagged as MSI level 3 in the feature matrix.
- Only MS2 spectra assigned to a feature were computed — no orphaned/unassigned MS2 entries appear in the .ms temp file output.
- Number of annotated features is consistent with the upstream feature matrix row count after optional minfrac filtering.
- In the paper's benchmark, ~76% of formulas and ~65% of structures were accurately annotated, giving a sanity-check ceiling for accuracy.

## Limitations

- Annotated identifications are only MSI level 3; they require downstream spectral matching (MSI level 2) or GNPS networking for higher-confidence assignments.
- Accuracy depends on the adduct/ionisation-mode configuration being correct in config.yaml.
- The article does not state SIRIUS or CSI:FingerID version numbers, so reproducibility may require pinning those externally.

## Evidence

- [methods] SiriusAdapter consumes both mzML and featureXML: [evidence span withheld — anchor status: not_found]
- [methods] SIRIUS builds internal .ms file from feature-assigned MS2: [evidence span withheld — anchor status: not_found]
- [methods] Output is MSI level 3: [evidence span withheld — anchor status: not_found]
- [readme] README describes step 5 as SIRIUS/CSI:FingerID predictions: [evidence span withheld — anchor status: external_doc]
- [readme] README: annotation notebook assigns MSI level 3: [evidence span withheld — anchor status: external_doc]
- [abstract] Benchmark accuracy of formula and structure annotation: "accurately annotated 76% of the molecular formulas and 65% of the structures"
- [methods] Output file name: [evidence span withheld — anchor status: not_found]
- [intro] Article reports SIRIUS/CSI:FingerID as structural and formula prediction tools: "structural and formula predictions via SIRIUS [18] and CSI:FingerID [19]"
