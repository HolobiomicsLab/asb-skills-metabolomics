---
name: lcms-feature-detection-and-alignment
description: Use when you have raw centroided LC-MS runs (mzXML/mzML) from a spatially
  mapped sample set and need a per-run feature table aligned across all runs to build
  intensity maps or heat maps.
when_to_use_negative:
- The input is already an aligned feature quantification table — proceed directly
  to filtering, annotation, normalization, or 'ili mapping instead.
- You need MS/MS-based identification validation — Optimus annotation is mz-RT matching
  at Metabolomics Standards Initiative level 2 and 'MS/MS validation of putative annotations
  is needed (currently not provided in Optimus)'.
- You plan to post large raw datasets (~a hundred or more LC-MS runs) on a 2 GB RAM
  machine — Optimus states 2 GB is minimal and 'not enough for analysis of large datasets'.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Optimus
  role: KNIME workflow that orchestrates feature detection, alignment, blank filtering,
    and quantification of LC-MS runs
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: provides the state-of-the-art feature detection and quantification algorithms
    executed inside Optimus
- name: KNIME
  role: workflow management platform hosting and executing the Optimus pipeline
  repo: https://www.knime.org
- name: pyopenms
  role: Python bindings for OpenMS installed as an Optimus dependency
- name: Python 2.7
  role: runtime required by the KNIME-Python integration of the Optimus workflow
- name: OptimusViewer
  role: visualizes XIC plots, MS peaks, and MS2 spectra for the detected features
    stored in OptimusViewer_input.db
  repo: https://github.com/alexandrovteam/OptimusViewer
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:5b3feaf7f3f64f7d5e5e1da3d10acd588a8e1db0e498a19ad12e4e0277c04e5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - KNIME
  - OpenMS
  - Optimus
  - OptimusViewer
  - Python
  - Python 2.7
  - pyopenms
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lcms-feature-detection-and-alignment

## Summary

Detect LC-MS features in each run and align/quantify them across all runs using the OpenMS algorithms embedded in the Optimus KNIME workflow, producing a consensus feature quantification matrix ready for blank filtering, annotation, and spatial mapping in 'ili.

## When to use

Use when you have raw centroided LC-MS runs (mzXML/mzML) from a spatially mapped sample set and need a per-run feature table aligned across all runs to build intensity maps or heat maps.

## When NOT to use

- The input is already an aligned feature quantification table — proceed directly to filtering, annotation, normalization, or 'ili mapping instead.
- You need MS/MS-based identification validation — Optimus annotation is mz-RT matching at Metabolomics Standards Initiative level 2 and 'MS/MS validation of putative annotations is needed (currently not provided in Optimus)'.
- You plan to post large raw datasets (~a hundred or more LC-MS runs) on a 2 GB RAM machine — Optimus states 2 GB is minimal and 'not enough for analysis of large datasets'.

## Inputs

- LC-MS sample runs in mzXML or mzML format (centroided internally by Optimus)
- Experimental-design CSV generated from the runs, with blank runs manually labelled BLANK in the LC-run type column
- stub.txt placeholder file for KNIME nodes without real inputs
- Optional: list of molecules of interest (CSV or exported from GNPS) for mz-RT annotation after alignment

## Outputs

- features_quantification_matrix.csv — rows = samples, columns = consensus features named `mz_value RT charge (ID: n)`
- Optimus_settings.ini — all parameter values used, for reproducibility
- OptimusViewer_input.db — XICs and MS/MS spectra of detected features for OptimusViewer
- ili_input.csv plus PNG background image and coordinates CSV for spatial maps in 'ili

## How to apply

Load the LC-MS runs into the Optimus KNIME workflow via the 'Read LC-MS runs' node (mzXML is internally converted to centroided mzML), and generate an experimental-design CSV with the 'Generate template of experimental design' node, labelling blank runs as BLANK. Execute the 'Detect LC-MS features' node, which runs OpenMS feature detection (FeatureFinderMetabo-style); for QTOF-type data use an m/z tolerance of ~20 ppm and, for a ~10-min UHPLC gradient, an RT tolerance of ~30 s. Then run the 'Align and quantify features' node, which aligns features across all runs, assigns numeric feature IDs to consensus features, and computes intensities per consensus feature. Optionally enable the 'Filter features' node to exclude background features detected in BLANK-labelled runs before saving. Executing the 'Save results' node writes three files: features_quantification_matrix.csv (rows = samples, columns = consensus features), Optimus_settings.ini (all parameter values), and OptimusViewer_input.db (XICs and MS/MS spectra of detected features).

## Related tools

- **Optimus** (KNIME workflow that orchestrates feature detection, alignment, blank filtering, and quantification of LC-MS runs) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (provides the state-of-the-art feature detection and quantification algorithms executed inside Optimus)
- **KNIME** (workflow management platform hosting and executing the Optimus pipeline) — https://www.knime.org
- **pyopenms** (Python bindings for OpenMS installed as an Optimus dependency)
- **Python 2.7** (runtime required by the KNIME-Python integration of the Optimus workflow)
- **OptimusViewer** (visualizes XIC plots, MS peaks, and MS2 spectra for the detected features stored in OptimusViewer_input.db) — https://github.com/alexandrovteam/OptimusViewer

## Examples

```
pip install six pandas protobuf pymspec pyopenms
```

## Evaluation signals

- The 'Save results' node produces exactly three files: features_quantification_matrix.csv, Optimus_settings.ini, and OptimusViewer_input.db in the configured output directory.
- features_quantification_matrix.csv is nonempty and its column headers follow the format `mz_value RT charge (ID: numeric_identifier)`.
- Optimus_settings.ini records the parameter values actually used during the run.
- After blank filtering, features attributable to BLANK-labelled control runs are removed from the quantification matrix while genuine sample features (e.g. phone-swab wells of rows B-H) are retained.
- The workflow completes without the troubleshooting-table ValueError 'Input list of LC-MS features is empty'.

## Limitations

- Optimus creates temporary files 'that can occupy up to few times more space than initial dataset' and are not deleted automatically, so disk usage must be planned (cleanup is available as an in-workflow option).
- 2 GB RAM is minimal; it is 'not enough for analysis of large datasets containing about a hundred or more LC-MS runs'.
- Blank filtering and the other exclusion steps are optional nodes; skipping the experimental-design BLANK labelling leaves background features in the quantification matrix.

## Evidence

- [readme] Save-results outputs: [evidence span withheld — anchor status: not_found]
