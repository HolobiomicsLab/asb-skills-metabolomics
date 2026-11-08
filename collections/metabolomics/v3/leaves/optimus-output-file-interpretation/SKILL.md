---
name: optimus-output-file-interpretation
description: Use when an Optimus workflow run has completed and you must verify, open,
  or interpret the output directory contents — e.g. checking that features_quantification_matrix.csv,
  Optimus_settings.ini, and OptimusViewer_input.db are present and correctly structured.
when_to_use_negative:
- Input is already a feature quantification table rather than output of an executed
  Optimus workflow — this skill validates Optimus-produced artifacts only
- You need to re-run feature detection or alignment instead of interpreting completed
  output files
- The file to inspect was not produced by the Optimus workflow (OptimusViewer opens
  only '*.db' files produced by Optimus)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Optimus
  role: KNIME workflow whose 'Save results' node writes the three output files (quantification
    matrix, settings INI, viewer database)
  repo: https://github.com/MolecularCartography/Optimus
- name: OptimusViewer
  role: Application for opening OptimusViewer_input.db to visualize XIC plots, MS
    peaks, and MS2 spectra of detected features and overlay them across features/runs
  repo: https://github.com/alexandrovteam/OptimusViewer
- name: KNIME
  role: Workflow management platform in which the Save results node is configured
    with an output directory and executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:07e78cec780763122cccc0ec0c0017a0ccc05dc994120bca9b89058047bfd123
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - KNIME
  - Optimus
  - OptimusViewer
  - Python
  - pyopenms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# optimus-output-file-interpretation

## Summary

Interprets and validates the three files produced by the Optimus (KNIME/OpenMS) Save results node: features_quantification_matrix.csv (samples x consensus features intensity table), Optimus_settings.ini (parameter record for reproducibility), and OptimusViewer_input.db (XICs and MS/MS spectra for downstream visualization).

## When to use

Use when an Optimus workflow run has completed and you must verify, open, or interpret the output directory contents — e.g. checking that features_quantification_matrix.csv, Optimus_settings.ini, and OptimusViewer_input.db are present and correctly structured.

## When NOT to use

- Input is already a feature quantification table rather than output of an executed Optimus workflow — this skill validates Optimus-produced artifacts only
- You need to re-run feature detection or alignment instead of interpreting completed output files
- The file to inspect was not produced by the Optimus workflow (OptimusViewer opens only '*.db' files produced by Optimus)

## Inputs

- Output directory configured in the Optimus 'Save results' KNIME node after feature detection, alignment, quantification, and (optional) blank filtering
- OptimusViewer_input.db file produced by the Save results node (for viewing in OptimusViewer)

## Outputs

- features_quantification_matrix.csv — table openable in any spreadsheet editor, rows = samples, columns = consensus features (mz_value RT charge (ID: n))
- Optimus_settings.ini — record of all parameter values used, for reproducibility
- OptimusViewer_input.db — XICs and MS/MS spectra of detected features for visualization with OptimusViewer

## How to apply

After executing the 'Save results' node in the Optimus v1.0.0 KNIME workflow with a configured output directory, confirm exactly three files were written. Open features_quantification_matrix.csv in any spreadsheet editor (e.g. Excel) and check that rows correspond to samples and columns are consensus features named in the format `mz_value RT charge (ID: n)` with numeric feature IDs assigned during alignment and quantification. Verify Optimus_settings.ini records the parameter values used, enabling reproducibility of the feature detection, alignment, blank-filtering, and quantification run. Load OptimusViewer_input.db into the OptimusViewer application (File > Open) to inspect extracted ion chromatograms (XIC), MS peaks, and MS2 spectra per consensus feature; the viewer presents the quantification matrix with runs as columns and the first five columns holding numeric feature ID, m/z, retention time, charge, and any metabolite annotations.

## Related tools

- **Optimus** (KNIME workflow whose 'Save results' node writes the three output files (quantification matrix, settings INI, viewer database)) — https://github.com/MolecularCartography/Optimus
- **OptimusViewer** (Application for opening OptimusViewer_input.db to visualize XIC plots, MS peaks, and MS2 spectra of detected features and overlay them across features/runs) — https://github.com/alexandrovteam/OptimusViewer
- **KNIME** (Workflow management platform in which the Save results node is configured with an output directory and executed)

## Examples

```
OptimusViewer  # launch executable, then File > Open -> OptimusViewer_input.db produced by the Save results node
```

## Evaluation signals

- All three named files exist in the output directory: features_quantification_matrix.csv, Optimus_settings.ini, OptimusViewer_input.db
- features_quantification_matrix.csv has a nonempty table whose column names match the format `mz_value RT charge (ID: numeric_identifier)`
- Optimus_settings.ini column values record the parameters actually used in the run
- OptimusViewer loads the .db and shows the feature quantification matrix with the first 5 special columns (numeric feature ID, m/z, retention time, charge, metabolite annotations)
- The workflow completed without the ValueError 'Input list of LC-MS features is empty' listed in the troubleshooting table

## Limitations

- OptimusViewer only opens '*.db' files produced by the Optimus workflow; outputs from other tools cannot be loaded
- Optimus runs only on 64-bit systems and creates temporary files that are not deleted automatically (a cleanup option exists inside the workflow)
- Putative mz-RT annotations implement Metabolomics Standards Initiative level 2 and MS/MS validation of putative annotations is not provided in Optimus

## Evidence

- [readme] Go to `File > Open` and select a "*.db" file produced by Optimus workflow.: [evidence span withheld — anchor status: external_doc]
- [readme] After the file is loaded, you will see an LC-MS feature quantification matrix where LC-MS runs correspond to columns, and features correspond to rows.: [evidence span withheld — anchor status: external_doc]
- [readme] The first 5 columns are special: they do not represent any run.: [evidence span withheld — anchor status: external_doc]
