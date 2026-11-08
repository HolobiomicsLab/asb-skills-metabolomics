---
name: mrmhub-status-flag-verification
description: Reference for checking that a processing step in a mrmhub MRMhubExperiment
  actually mutated the intended variable and its is_* status flag before downstream
  QC or reporting.
when_to_use_negative:
- Input is a plain CSV intensity table not wrapped in a MRMhubExperiment — no is_*
  status flags exist to verify.
- You only need peak integration (INTEGRATOR) and have not yet entered the QUANT post-processing
  pipeline.
- Flag state is already known and you only need to re-run a correction without auditing
  reproducibility.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3520
tools:
- name: mrmhub
  role: Provides the MRMhubExperiment container, correct_isotopic_interferences(),
    set_lipid_class() (via rgoslin) and the is_* status flags whose state is verified
  repo: https://github.com/SLINGhub/MRMhub
- name: enviPat
  role: Backend used by correct_isotopic_interferences() to calculate isotope patterns
    that drive interference correction
- name: rgoslin
  role: Parses lipid shorthand names so lipid classes can be assigned with set_lipid_class;
    parsed classes are part of the verified feature annotations
- name: R
  role: Execution environment for the QUANT module and its function calls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03060
    title: 'MRMkit: Automated Data Processing for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:118e8c2b9c4eb53d8bb4d2418f9a6e2cede88f3ce543b422f080cac1431719d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SLINGhub/MRMhub
  tools:
  - R
  - enviPat
  - mrmhub
  - rgoslin
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# mrmhub-status-flag-verification

## Summary

Verify that MRMhub/QUANT status flags (such as is_isotope_corr) are correctly set on a MRMhubExperiment after a correction or annotation step, and export the corrected dataset with updated feature annotations for traceability.

## When to use

Reference for checking that a processing step in a mrmhub MRMhubExperiment actually mutated the intended variable and its is_* status flag before downstream QC or reporting.

## When NOT to use

- Input is a plain CSV intensity table not wrapped in a MRMhubExperiment — no is_* status flags exist to verify.
- You only need peak integration (INTEGRATOR) and have not yet entered the QUANT post-processing pipeline.
- Flag state is already known and you only need to re-run a correction without auditing reproducibility.

## Inputs

- MRMhubExperiment object produced by import_data_mrmhub (e.g. from MRMhub_demo.tsv)
- intensity variable to be corrected (e.g. 'intensity' or 'norm_intensity')
- feature annotation fields such as lipid shorthand names for class assignment via rgoslin

## Outputs

- MRMhubExperiment with is_isotope_corr status flag set to TRUE and corrected feature areas
- corrected dataset exported with updated feature annotations (e.g. isotopic_corrected_annotations.csv)

## How to apply

Build the MRMhubExperiment from the source file (e.g. import_data_mrmhub on MRMhub_demo.tsv), then run the intended correction — for example correct_isotopic_interferences(variable = 'intensity', sequential_correction = TRUE, neg_to_na = TRUE) so feature areas are recomputed free of isotopic contributions using enviPat. After each mutating call, inspect the corresponding is_* status flag on the MRMhubExperiment (e.g. is_isotope_corr should be TRUE once isotopic correction has been applied) to confirm the step executed rather than silently skipping, which matters because MRMhub 'records a full digital footprint of every step for reproducibility and traceability'. Feature annotations such as lipid classes assigned via rgoslin (set_lipid_class) are likewise carried on the feature metadata and should be checked for non-NA class values. Finally, export the corrected dataset together with its updated feature annotations (e.g. isotopic_corrected_annotations.csv) so the flag state and the recomputed intensities travel together.

## Related tools

- **mrmhub** (Provides the MRMhubExperiment container, correct_isotopic_interferences(), set_lipid_class() (via rgoslin) and the is_* status flags whose state is verified) — https://github.com/SLINGhub/MRMhub
- **enviPat** (Backend used by correct_isotopic_interferences() to calculate isotope patterns that drive interference correction)
- **rgoslin** (Parses lipid shorthand names so lipid classes can be assigned with set_lipid_class; parsed classes are part of the verified feature annotations)
- **R** (Execution environment for the QUANT module and its function calls)

## Examples

```
mexp <- MRMhubExperiment() |> import_data_mrmhub(path = demo_file, import_metadata = TRUE) |> correct_isotopic_interferences(variable = 'intensity', sequential_correction = TRUE, neg_to_na = TRUE); stopifnot(mexp@is_isotope_corr)
```

## Evaluation signals

- is_isotope_corr flag on the MRMhubExperiment is TRUE after running correct_isotopic_interferences()
- Corrected feature areas differ from the raw intensities in at least the features affected by isotopic interferences
- Every analyte has a non-NA lipid class after set_lipid_class / rgoslin parsing
- Exported CSV contains both the corrected intensities and the updated feature annotations (e.g. isotopic_corrected_annotations.csv)
- Full digital footprint of the step is present, matching the README's traceability claim

## Limitations

- Specialised functions in mrmhub rely on additional packages (enviPat, rgoslin) that are not installed upfront; missing dependencies cause the correction or annotation step to fail without setting the flag.
- Sequential correction with neg_to_na can leave negative or undefined areas as NA, so downstream checks must tolerate NAs on affected features.
- Requires R ≥ 4.1 and a fresh RStudio/Positron session for the QUANT install; older environments may not expose the expected flags.

## Evidence

- [methods] MRMhub corrects isotopic interferences using enviPat via correct_isotopic_interferences, and assigns lipid classes by parsing lipid names with rgoslin: [evidence span withheld — anchor status: not_found]
- [intro] processing population-scale studies within minutes on standard hardware while recording a full digital footprint of every step for reproducibility and traceability: "processing population-scale studies within minutes on standard hardware while recording a full digital footprint of every step for reproducibility and traceability"
- [readme] demo_file <- system.file("extdata", "MRMhub_demo.tsv", package = "mrmhub"): "demo_file <- system.file("extdata", "MRMhub_demo.tsv", package = "mrmhub")"
- [methods] To keep the base installation light, a number of specialised functions in mrmhub rely on additional packages that are not installed upfront.: "To keep the base installation light, a number of specialised functions in `mrmhub` rely on additional packages that are not installed upfront."
