---
name: dose-response-curve-visualization
description: Use when a multiclass metabolomics dataset contains spiked reference
  metabolites with known dose levels and you must visually verify whether a ranked
  processing workflow (e.g., HAM+NON+CUT+[CON-RAN] vs KNN+NON+LOG+[VAS-LIN]) preserves
  the true dose gradient.
when_to_use_negative:
- The dataset provides no reference metabolites/spike-ins with known dose levels (as
  in most benchmark datasets, where criterion e cannot be applied at all)
- The goal is statistical workflow ranking itself (S5) rather than visual validation
  of a known dose gradient for spiked standards
- The data are time-course rather than multiclass with dose-level classes, so no discrete
  dose gradient exists to plot
edam_operation: http://edamontology.org/operation_3208
tools:
- name: NOREVA
  role: R package that scans and ranks thousands of processing workflows and supplies
    the processed intensities of the spiked metabolites per dose class
  repo: https://github.com/idrblab/NOREVA
- name: ggord
  role: R plotting utilities used to draw the side-by-side dose-response curves mirroring
    the Figure 6 layout
  repo: https://github.com/fawda123/ggord
- name: R
  role: Environment in which the NOREVA protocol runs and the dose-response fits and
    plots are produced
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:7a94e8f2c9ec3f0000f57b24150ccd91b4e18cb893b6ac77f94215eda56160fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  - XCMS
  - ggord
  edam_operation: http://edamontology.org/operation_3208
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dose-response-curve-visualization

## Summary

Visualizes dose-response curves of spiked internal-standard metabolites (e.g., aspartic acid and malic acid at 6/12/25/50 µL in MetaboLights MTBLS6) after NOREVA workflow ranking, mirroring Figure 6 of the NOREVA protocol to show which processing workflow preserves the true spiked dose gradient.

## When to use

Use when a multiclass metabolomics dataset contains spiked reference metabolites with known dose levels and you must visually verify whether a ranked processing workflow (e.g., HAM+NON+CUT+[CON-RAN] vs KNN+NON+LOG+[VAS-LIN]) preserves the true dose gradient.

## When NOT to use

- The dataset provides no reference metabolites/spike-ins with known dose levels (as in most benchmark datasets, where criterion e cannot be applied at all)
- The goal is statistical workflow ranking itself (S5) rather than visual validation of a known dose gradient for spiked standards
- The data are time-course rather than multiclass with dose-level classes, so no discrete dose gradient exists to plot

## Inputs

- Processed peak table (samples as rows, metabolites as columns) from the first- and last-ranked NOREVA workflows on MetaboLights MTBLS6 (GC-MS quadrupole, PMID22647087)
- Dose-level labels for the spiked internal standards (6, 12, 25, 50 µL per class for aspartic acid and malic acid)
- NOREVA workflow ranking output scoring thousands of candidate workflows by multiple criteria

## Outputs

- Side-by-side dose-response curves of spiked aspartic acid and malic acid under the first- and last-ranked workflows (Figure 6-style plot via ggord)
- Monotonicity verdict: confirmation that the top workflow preserves and the bottom workflow fails to preserve the 6/12/25/50 µL dose gradient

## How to apply

After running the NOREVA R protocol over the full combinatorial pipeline (S1 data filtering/imputation, S2 QCS correction, S3 transformation, S4 normalization, S5 multi-criteria assessment) on the standardized MTBLS6 peak table, extract the processed intensity values of the spiked internal standards (aspartic acid and malic acid) for each dose class (6, 12, 25, 50 µL) under the first-ranked workflow. Fit a dose-response relationship and confirm the intensities follow the 6/12/25/50 µL gradient monotonically. Repeat the extraction under the last-ranked workflow (e.g., KNN+NON+LOG+[VAS-LIN]) and show with the same fit that the gradient is non-monotonic or discordant with the true doses. Plot the two dose-response curves side by side using the ggord R plotting utilities (installed via devtools::install_github), mirroring the Figure 6 layout of the Nature Protocols paper.

## Related tools

- **NOREVA** (R package that scans and ranks thousands of processing workflows and supplies the processed intensities of the spiked metabolites per dose class) — https://github.com/idrblab/NOREVA
- **ggord** (R plotting utilities used to draw the side-by-side dose-response curves mirroring the Figure 6 layout) — https://github.com/fawda123/ggord
- **R** (Environment in which the NOREVA protocol runs and the dose-response fits and plots are produced) — https://www.r-project.org/

## Examples

```
library(NOREVA); normulticlassisall(fileName, IS)
```

## Evaluation signals

- Under the first-ranked workflow HAM+NON+CUT+[CON-RAN], processed intensities of aspartic acid and malic acid increase monotonically across the 6/12/25/50 µL spiked classes in the dose-response fit
- Under the last-ranked workflow KNN+NON+LOG+[VAS-LIN], the same fit is visibly non-monotonic or discordant with the true dose levels
- The plotted figure reproduces the qualitative ranking reported in the paper's Figure 6 for the MTBLS6 dataset
- Only the two identified internal-standard metabolites are used as reference; no other metabolites contribute to the dose gradient check
- The dose labels on the x-axis match the recorded spike-in levels (6, 12, 25, 50 µL per class)

## Limitations

- Criterion e (LogFC-based assessment using reference/spike-in metabolites) is infeasible for datasets that provide no reference metabolites, so this visualization cannot be applied there
- Workflow performance is dataset-dependent: a workflow that preserves the gradient on MTBLS6 may perform poorly on another dataset, so the plot's conclusions are study-specific
- Manual standardization of the peak table to the unified NOREVA format (samples as rows, metabolites as columns) is required first and is time-consuming due to diverse raw data file organizations

## Evidence

- [full_text] Top-ranked workflow preserves gradient: [evidence span withheld — anchor status: not_found]
- [full_text] Bottom-ranked workflow fails: [evidence span withheld — anchor status: not_found]
- [discussion] Spike-in criterion infeasible without references: [evidence span withheld — anchor status: not_found]
- [discussion] Peak table standardization requirement: [evidence span withheld — anchor status: not_found]
- [readme] ggord installation for plotting: [evidence span withheld — anchor status: external_doc]
