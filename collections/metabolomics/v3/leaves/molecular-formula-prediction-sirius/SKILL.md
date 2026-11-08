---
name: molecular-formula-prediction-sirius
description: Use when post-processing pre-processed featureXML/mzML files from OpenMS
  (UmetaFlow) — each feature's MS2 spectrum plus its adduct/ionisation-mode information
  from config.yaml — to rank molecular formula candidates and record the top-ranked
  SIRIUS formula per feature.
when_to_use_negative:
- Input is already a feature table annotated by spectral-library matching at MSI level
  2 — those matches supersede tentative SIRIUS predictions.
- The MS2 spectra have not been assigned to features via the upstream pre-processing/feature-linking
  steps; SiriusAdapter needs featureXML adduct information to build its internal .ms
  files.
- The dataset lacks MS/MS spectra (MS1-only acquisition) — SIRIUS formula ranking
  relies on MS2 fragmentation patterns.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Ranks molecular formula candidates per feature from feature-filtered MS2 spectra
    with adduct information; SiriusAdapter from OpenMS invokes it and builds the internal
    .ms temporary file
- name: CSI:FingerID
  role: Receives the top-ranked SIRIUS formula candidates and returns structural predictions;
    both are mapped back onto the feature matrix as MSI level 3
- name: UmetaFlow
  role: Wraps SIRIUS/CSI:FingerID as a Snakemake rule (structural, formula and compound
    class predictions) or as pyOpenMS Jupyter step 5, then writes sirius_annotations.tsv
    and integrates predictions into the GNPS graphml
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS
  role: Provides the SiriusAdapter tool through pyOpenMS bindings; also supplies the
    upstream featureXML/mzML inputs and handles feature detection, alignment, adduct
    annotation, re-quantification and feature linking before SIRIUS runs
- name: pyOpenMS
  role: Python bindings used in the Jupyter notebook (5_SIRIUS_CSI.ipynb) implementation
    to invoke SIRIUS and CSI:FingerID on pre-processed OpenMS feature data
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:95abe9cfd4402155ea9521c77479e0131e2357e4ea0d0721ee6f464ab68073ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - CSI:FingerID
  - OpenMS
  - Python
  - SIRIUS
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-formula-prediction-sirius

## Summary

Annotate LC–MS/MS feature matrices with the highest-ranked molecular formula predictions from SIRIUS, assigned as MSI level 3 identifications, so downstream CSI:FingerID structural predictions and GNPS molecular networking can key on the same unique feature identifications.

## When to use

Use when post-processing pre-processed featureXML/mzML files from OpenMS (UmetaFlow) — each feature's MS2 spectrum plus its adduct/ionisation-mode information from config.yaml — to rank molecular formula candidates and record the top-ranked SIRIUS formula per feature.

## When NOT to use

- Input is already a feature table annotated by spectral-library matching at MSI level 2 — those matches supersede tentative SIRIUS predictions.
- The MS2 spectra have not been assigned to features via the upstream pre-processing/feature-linking steps; SiriusAdapter needs featureXML adduct information to build its internal .ms files.
- The dataset lacks MS/MS spectra (MS1-only acquisition) — SIRIUS formula ranking relies on MS2 fragmentation patterns.

## Inputs

- mzML files with feature-filtered MS2 spectra (output of pre-processing/re-quantification steps)
- featureXML (feature/adduct information) from OpenMS pre-processing
- ionisation mode and adduct information from config.yaml

## Outputs

- sirius_annotations.tsv — feature matrix annotated with highest-ranked SIRIUS formula (and CSI:FingerID structure) predictions keyed by unique feature identifications
- MSI level 3 formula annotations per feature

## How to apply

Load the feature table and feature-filtered MS2 spectra produced by the upstream pre-processing/re-quantification steps, along with the ionisation mode and adduct information from config.yaml. Invoke SiriusAdapter (OpenMS) on each feature's MS2 spectrum with the adduct information so it builds an internal .ms temporary file containing only the MS2 spectra assigned to that feature, then rank molecular formula candidates. Forward the top-ranked formula candidates to CSI:FingerID to obtain structural predictions. Map the highest-ranked SIRIUS formula onto the unique feature IDs in the feature matrix and write the annotations as sirius_annotations.tsv, classifying them as Metabolomics Standards Initiative (MSI) level 3. In UmetaFlow's Snakemake implementation, SIRIUS/CANOPUS formula predictions are optionally paired with CSI:FingerID structural and compound-class predictions in the same rule; in the pyOpenMS Jupyter implementation this is step 5 (5_SIRIUS_CSI.ipynb). Validation on in-house actinomycete and commercial-standard data showed SIRIUS accurately predicted 76% of molecular formulas, so treat the output as tentative — verify against spectral-library matches (MSI level 2) when available.

## Related tools

- **SIRIUS** (Ranks molecular formula candidates per feature from feature-filtered MS2 spectra with adduct information; SiriusAdapter from OpenMS invokes it and builds the internal .ms temporary file)
- **CSI:FingerID** (Receives the top-ranked SIRIUS formula candidates and returns structural predictions; both are mapped back onto the feature matrix as MSI level 3)
- **UmetaFlow** (Wraps SIRIUS/CSI:FingerID as a Snakemake rule (structural, formula and compound class predictions) or as pyOpenMS Jupyter step 5, then writes sirius_annotations.tsv and integrates predictions into the GNPS graphml) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS** (Provides the SiriusAdapter tool through pyOpenMS bindings; also supplies the upstream featureXML/mzML inputs and handles feature detection, alignment, adduct annotation, re-quantification and feature linking before SIRIUS runs)
- **pyOpenMS** (Python bindings used in the Jupyter notebook (5_SIRIUS_CSI.ipynb) implementation to invoke SIRIUS and CSI:FingerID on pre-processed OpenMS feature data) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all  # runs the SIRIUS/CSI:FingerID rule (structural, formula and compound class predictions) as part of the UmetaFlow pipeline
```

## Evaluation signals

- sirius_annotations.tsv must contain an annotation row for each feature ID present in the input feature matrix (unique feature identifications are the join key).
- Formula annotations should be flagged MSI level 3 — only spectral matches scoring above 60 qualify as MSI level 2, so a SIRIUS-only annotation must not be promoted.
- Validation metric from the paper: in-house actinomycete and commercial-standards datasets gave ~76% accurate molecular formula predictions; a new dataset scoring wildly below this baseline warrants re-checking adduct/ionisation-mode parameters.
- The internal .ms temporary file must contain only the MS2 spectra assigned to each feature (no unassigned spectra) — verify via the SiriusAdapter log or by re-running with the featureXML supplied.
- Downstream FBMN graphml integration must resolve the same unique feature IDs, so the sirius_annotations.tsv key column must match the GNPS feature index.

## Limitations

- SIRIUS formula predictions are tentative (MSI level 3); the article reports only ~76% accurate formula predictions on in-house data, so a substantial minority may be wrong.
- Only the MS2 spectra assigned to a feature are computed — unassigned MS2 spectra are skipped, so features without linked MS2 spectra receive no formula.
- The supplementary tables (Tables S9–S11) that document per-compound validation are in Additional files 2–4, not the main text, so per-metabolite accuracy must be re-derived from SI.
- No explicit software version numbers are stated for SIRIUS or CSI:FingerID in the article text.

## Evidence

- [methods] SiriusAdapter builds internal .ms temporary file so only MS2 spectra assigned to a feature are computed: [evidence span withheld — anchor status: not_found]
- [methods] MSI level 3 classification for SIRIUS formula annotations: [evidence span withheld — anchor status: not_found]
- [readme] README confirms step 5 covers SIRIUS and CSI:FingerID predictions: [evidence span withheld — anchor status: external_doc]
- [abstract] In-house validation accuracy for SIRIUS formula predictions: "accurately annotated 76% of the molecular formulas and 65% of the structures"
- [readme] Spectral matches above 60 are MSI level 2, distinguishing them from SIRIUS MSI level 3: "Spectral matching with in-house or a publicly available library (MGF/MSP/mzML format) and annotation of the feature matrix with matches that have a score above 60 (MSI level 2)."
