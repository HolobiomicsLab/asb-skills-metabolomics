---
name: gc-cims-dataframe-transformation
description: Use when you have a data frame of isotopologue integration areas from
  isoSCAN's autoQ and need per-sample percentage-normalized values, or must confirm
  the transformed frame feeds metBarPlot.
when_to_use_negative:
- Input is raw GC-CI-MS instrument data (e.g. Agilent 7890A/7000 QqQ or 7890B/7250
  QTOF output) that has not yet been integrated into an autoQ results data frame
- The goal is isotopologue quantification itself rather than post-hoc normalization
  — use autoQ first
- A plotting or statistical comparison is needed without transformation — metBarPlot
  can be applied to the untransformed autoQ results
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: isoSCAN
  role: Provides QTransform and metBarPlot functions; autoQ integration results are
    the input to QTransform and its output feeds metBarPlot as autoQres
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment in which QTransform, metBarPlot, and write.csv are executed
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:634a33c5a9c716164f079b28d61e186933fbac9751b79ee1c522d1036d8b2dd1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gc-cims-dataframe-transformation

## Summary

Normalize isoSCAN autoQ integration results to percentage-normalized isotopologue values per sample using the QTransform function (val.to.use="area", val.trans="P"), and verify the resulting data frame remains compatible with downstream isoSCAN plotting (metBarPlot).

## When to use

Use when you have a data frame of isotopologue integration areas from isoSCAN's autoQ and need per-sample percentage-normalized values, or must confirm the transformed frame feeds metBarPlot.

## When NOT to use

- Input is raw GC-CI-MS instrument data (e.g. Agilent 7890A/7000 QqQ or 7890B/7250 QTOF output) that has not yet been integrated into an autoQ results data frame
- The goal is isotopologue quantification itself rather than post-hoc normalization — use autoQ first
- A plotting or statistical comparison is needed without transformation — metBarPlot can be applied to the untransformed autoQ results

## Inputs

- Data frame of isoSCAN autoQ integration results (peak areas per metabolite, isotopologue, and sample)
- QTransform arguments: val.to.use="area" and val.trans="P"

## Outputs

- Percentage-normalized data frame (trans_integ) with metabolite, isotopologue, sample, and normalized value columns
- qtransform_normalized.csv written via write.csv with row.names=FALSE

## How to apply

Load the autoQ integration results produced by the isoSCAN R-package in R. Apply QTransform with val.to.use="area" (using peak integration areas as the quantified value) and val.trans="P" (percentage normalization) to convert areas into percentage-normalized isotopologue values per sample. Inspect the output (trans_integ) to confirm it retains the expected columns for metabolite, isotopologue, sample, and normalized value. Pass the transformed data frame directly to metBarPlot as its autoQres argument to verify downstream compatibility, as demonstrated in the isoSCAN Package Vignette. Persist the result with write.csv(..., "qtransform_normalized.csv", row.names=FALSE).

## Related tools

- **isoSCAN** (Provides QTransform and metBarPlot functions; autoQ integration results are the input to QTransform and its output feeds metBarPlot as autoQres) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment in which QTransform, metBarPlot, and write.csv are executed) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
trans_integ <- QTransform(autoQ_integrations, val.to.use="area", val.trans="P"); metBarPlot(trans_integ); write.csv(trans_integ, "qtransform_normalized.csv", row.names=FALSE)
```

## Evaluation signals

- Output data frame retains the expected columns for metabolite, isotopologue, sample, and normalized value
- Percentage-normalized values (val.trans="P") fall in the expected 0–100 percentage range per sample for each metabolite
- The transformed data frame is accepted directly by metBarPlot as its autoQres argument without schema errors, matching the vignette workflow
- qtransform_normalized.csv is written with row.names=FALSE and re-reads into R with the same structure as trans_integ

## Limitations

- The percentage normalization is demonstrated in the vignette specifically with val.to.use="area"; behavior with other val.to.use options is not covered by this evidence
- The article/README provides no software version or changelog for isoSCAN, so function signatures may drift across package releases
- Downstream compatibility was verified only for metBarPlot as autoQres; other downstream consumers of QTransform output are not documented

## Evidence

- [intro] isoSCAN R-package scope: [evidence span withheld — anchor status: not_found]
- [results] isoSCAN open-source availability: [evidence span withheld — anchor status: not_found]
- [readme] README installation instructions: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
