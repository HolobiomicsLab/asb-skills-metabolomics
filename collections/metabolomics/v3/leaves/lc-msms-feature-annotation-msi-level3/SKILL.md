---
name: lc-msms-feature-annotation-msi-level3
description: Use when mzML files and OpenMS-annotated featureXML adduct information
  are available and you need to attach SIRIUS molecular formula and CSI:FingerID structural
  identifications (MSI level 3) to detected LC-MS/MS features.
when_to_use_negative:
- You only need MSI level 2 identifications via spectral library matching from an
  MGF/MSP/mzML library - use the separate UmetaFlow spectral matching step instead.
- Input files are raw vendor files that have not gone through UmetaFlow pre-processing
  and OpenMS feature/adduct annotation; SiriusAdapter requires the featureXML with
  adduct information, not raw data.
edam_operation: http://edamontology.org/operation_3665
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: UmetaFlow
  role: Workflow that orchestrates the SIRIUS/CSI:FingerID formula and structural
    prediction step and writes sirius_annotations.tsv for downstream GNPS FBMN and
    graph-view integration
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: SIRIUS
  role: Ranks molecular formula candidates for feature-assigned MS2 spectra via SiriusAdapter
- name: CSI:FingerID
  role: Predicts structures from the top-ranked SIRIUS formula candidates
- name: OpenMS
  role: Provides the SiriusAdapter tool consumed by UmetaFlow (and pyOpenMS bindings
    used in the Jupyter implementation)
- name: pyOpenMS
  role: Python bindings to the OpenMS algorithms used in the Jupyter notebook variant
    (5_SIRIUS_CSI.ipynb and 6_Annotation.ipynb)
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Snakemake
  role: Execution engine for the Snakemake implementation of the SiriusAdapter annotation
    step
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:45491c7f3606f6c079600f8106ff4579596a81dd3cad784afb1f8192ccf5f8d4
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
  edam_operation: http://edamontology.org/operation_3665
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# lc-msms-feature-annotation-msi-level3

## Summary

Annotates an LC-MS/MS feature table with molecular formula and structural identifications at Metabolomics Standards Initiative level 3 by running SIRIUS formula ranking and CSI:FingerID structure prediction on feature-assigned MS2 spectra via OpenMS SiriusAdapter in UmetaFlow.

## When to use

Use when mzML files and OpenMS-annotated featureXML adduct information are available and you need to attach SIRIUS molecular formula and CSI:FingerID structural identifications (MSI level 3) to detected LC-MS/MS features.

## When NOT to use

- You only need MSI level 2 identifications via spectral library matching from an MGF/MSP/mzML library - use the separate UmetaFlow spectral matching step instead.
- Input files are raw vendor files that have not gone through UmetaFlow pre-processing and OpenMS feature/adduct annotation; SiriusAdapter requires the featureXML with adduct information, not raw data.

## Inputs

- mzML centroided LC-MS/MS files from the pre-processed dataset
- OpenMS featureXML files with adduct annotation from the upstream re-quantification/adduct annotation steps
- Ionisation mode and adduct information from config.yaml
- (pyOpenMS path) sirius_predictions output files from 5_SIRIUS_CSI.ipynb

## Outputs

- Feature table annotated with highest-ranked SIRIUS molecular formula predictions (MSI level 3)
- Feature table annotated with highest-ranked CSI:FingerID structure predictions (MSI level 3)
- sirius_annotations.tsv keyed by unique feature identification

## How to apply

Run the SiriusAdapter step of the UmetaFlow workflow (Snakemake or pyOpenMS/Jupyter implementation) on the mzML files and corresponding OpenMS featureXML files produced by the pre-processing, map alignment, adduct annotation, and re-quantification steps. SiriusAdapter builds an internal .ms temporary file so that only the MS2 spectra assigned to a feature are computed, and the ionisation mode and adduct information from config.yaml (e.g. positive/negative mode) are supplied as input. The tool ranks molecular formula candidates with SIRIUS and forwards top-ranked candidates to CSI:FingerID for structural prediction. Collect the highest-ranked SIRIUS formula and CSI:FingerID structure predictions, map them onto the unique feature identifications in the feature matrix, and write the annotations as a sirius_annotations.tsv table. For the pyOpenMS path this corresponds to the 5_SIRIUS_CSI.ipynb notebook followed by 6_Annotation.ipynb, which merges the #1 ranked predictions into the feature table as MSI level 3 identifications, optionally alongside MSI level 2 spectral matches from a local MGF file.

## Related tools

- **UmetaFlow** (Workflow that orchestrates the SIRIUS/CSI:FingerID formula and structural prediction step and writes sirius_annotations.tsv for downstream GNPS FBMN and graph-view integration) — https://github.com/biosustain/snakemake_UmetaFlow
- **SIRIUS** (Ranks molecular formula candidates for feature-assigned MS2 spectra via SiriusAdapter)
- **CSI:FingerID** (Predicts structures from the top-ranked SIRIUS formula candidates)
- **OpenMS** (Provides the SiriusAdapter tool consumed by UmetaFlow (and pyOpenMS bindings used in the Jupyter implementation))
- **pyOpenMS** (Python bindings to the OpenMS algorithms used in the Jupyter notebook variant (5_SIRIUS_CSI.ipynb and 6_Annotation.ipynb)) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Snakemake** (Execution engine for the Snakemake implementation of the SiriusAdapter annotation step) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- Each feature ID in the output feature table carries exactly one #1-ranked SIRIUS formula and one #1-ranked CSI:FingerID structure keyed by the unique feature identification.
- Annotation level is Metabolomics Standards Initiative level 3, not level 2 - level 2 results come from the separate spectral matching step against an MGF/MSP/mzML library.
- Only MS2 spectra assigned to a feature were submitted, i.e. SiriusAdapter consumed the feature-filtered spectra rather than raw MS/MS.
- sirius_annotations.tsv is produced and can be joined to the GNPS FBMN graphml for graph-view integration.
- UmetaFlow reports ~76% molecular formula and ~65% structure annotation accuracy on benchmark data as a realistic expectation.

## Limitations

- Formula and structural predictions are MSI level 3 only; confirming level 2 identifications requires the separate spectral matching step against an in-house or public MGF/MSP/mzML library with a score above 60.
- The workflow requires both mzML files and the corresponding pre-processed feature/adduct information (featureXML); running SIRIUS/CSI:FingerID without feature-assigned spectra would compute on unassigned MS2.
- Benchmark performance (76% formulas, 65% structures) was measured on the MTBLS733 and MTBLS736 datasets and may differ on other LC-MS/MS data.

## Evidence

- [methods] feature matrix is annotated with highest-ranked SIRIUS formula and CSI:FingerID structure predictions, keyed by unique feature identifications, classified as MSI level 3: [evidence span withheld — anchor status: not_found]
- [methods] SiriusAdapter step takes mzML files and pre-processed feature/adduct information: [evidence span withheld — anchor status: not_found]
- [intro] Structural and formula predictions via SIRIUS and CSI:FingerID: "structural and formula predictions via SIRIUS [18] and CSI:FingerID [19]"
- [readme] Jupyter README: SIRIUS/CSI step and annotation notebook merge #1 ranked predictions as MSI level 3: "5) Structural and formula predictions with SIRIUS and CSI:FingerID

6) [Annotations](6_Annotation.ipynb): annotate the feature tables with #1 ranked SIRIUS and CSI:FingerID predictions (MSI level 3)"
- [abstract] Abstract: UmetaFlow accurately annotates 76% of formulas and 65% of structures: "accurately annotated 76% of the molecular formulas and 65% of the structures"
