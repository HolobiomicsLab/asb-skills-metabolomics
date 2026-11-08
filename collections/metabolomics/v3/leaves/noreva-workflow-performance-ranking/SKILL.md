---
name: noreva-workflow-performance-ranking
description: Use when a multiclass or time-course metabolomic peak table (e.g. GC-MS
  quadrupole MTBLS6 fetal calf serum with spiked aspartic acid and malic acid at 6/12/25/50
  µL) must be processed by an optimized workflow chosen from thousands of stage combinations.
when_to_use_negative:
- Datasets with fewer than 6 samples per class or timepoint, for which criterion Cc
  is unsuitable
- Datasets with no reference metabolites (no spike-ins/internal standards), where
  criterion Ce (LogFC) cannot be applied
- Raw LC-MS/NMR files still needing peak picking — use peak-table preparation tools
  (XCMS, NMRProc Flow) first; this skill assumes an existing peak table
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package that scans thousands of processing workflows, scores them with five
    criteria, and ranks them for multiclass/time-course data
  repo: github.com/idrblab/NOREVA
- name: R
  role: Execution environment for the NOREVA protocol and for dose-response verification
    of spike-in gradients
- name: XCMS
  role: Upstream peak-table preparation of raw GC-MS quadrupole data before NOREVA
    ranking
- name: ggord
  role: R plotting utility for visualizing workflow comparisons mirroring the paper's
    Figure 6 layout
  repo: fawda123/ggord
- name: NormalizeMets
  role: Dependency installed for the NOREVA normalization/assessment functions
  repo: metabolomicstats/NormalizeMets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:f10ecc32bb43ecd1ba83e7396c3f04ec92edd5e3e33d35f3564c24bb4a5d2525
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NOREVA
  - NormalizeMets
  - R
  - XCMS
  - ggord
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# noreva-workflow-performance-ranking

## Summary

Ranks the thousands of candidate peak-table processing workflows (S1 filtering/imputation through S4 normalization, e.g. HAM+NON+CUT+[CON-RAN] vs KNN+NON+LOG+[VAS-LIN]) using NOREVA's five multiple performance criteria, so the best- and worst-ranked workflows can be validated against known biology such as spike-in dose gradients.

## When to use

Use when a multiclass or time-course metabolomic peak table (e.g. GC-MS quadrupole MTBLS6 fetal calf serum with spiked aspartic acid and malic acid at 6/12/25/50 µL) must be processed by an optimized workflow chosen from thousands of stage combinations.

## When NOT to use

- Datasets with fewer than 6 samples per class or timepoint, for which criterion Cc is unsuitable
- Datasets with no reference metabolites (no spike-ins/internal standards), where criterion Ce (LogFC) cannot be applied
- Raw LC-MS/NMR files still needing peak picking — use peak-table preparation tools (XCMS, NMRProc Flow) first; this skill assumes an existing peak table

## Inputs

- Metabolomic peak table in NOREVA unified format (rows = samples by name, columns = metabolites), e.g. from MetaboLights MTBLS6 GC-MS quadrupole fetal calf serum data
- Label file for time-course/multi-class class assignment
- Column numbers of internal-standard (spike-in) metabolites for normulticlassisall (e.g. aspartic acid and malic acid)
- Study-assumption flags (SAalpha, SAbeta, SAgamma)

## Outputs

- Ranked list of processing workflows with multiple-criterion performance scores (first- and last-ranked workflow identified)
- Processed (normalized) intensity tables per workflow for downstream dose-response checks
- Dose-response plots of spiked metabolites under top- and bottom-ranked workflows (ggord/ggplot-style Figure 6 layout)

## How to apply

Standardize the peak table into NOREVA's unified format (samples as rows, metabolites as columns) with PrepareInuputFiles, then run the multiclass assessment function matching the data design — with QCS but no IS (normulticlassqcall), neither QCS nor IS (normulticlassnoall), or with internal standards but no QCS (normulticlassisall, passing the IS column numbers). The functions scan the full combinatorial pipeline of S1 data filtering/imputation, S2 QCS correction, S3 transformation and S4 normalization (after choosing study assumptions SAalpha/SAbeta/SAgamma), then score every workflow with the five criteria and output a ranking. For datasets with reference metabolites (spike-ins), criterion Ce/LogFC becomes applicable and directly tests whether processed intensities preserve known dose levels. Extract processed intensities of the spiked metabolites under the first- and last-ranked workflows and fit the 6/12/25/50 µL dose-response to confirm the top workflow (e.g. HAM+NON+CUT+[CON-RAN]) is monotonic while the bottom one (e.g. KNN+NON+LOG+[VAS-LIN]) is not, mirroring the paper's Figure 6 layout. Workflow quality is dataset-dependent, so the ranking must be recomputed per dataset rather than reused.

## Related tools

- **NOREVA** (R package that scans thousands of processing workflows, scores them with five criteria, and ranks them for multiclass/time-course data) — github.com/idrblab/NOREVA
- **R** (Execution environment for the NOREVA protocol and for dose-response verification of spike-in gradients)
- **XCMS** (Upstream peak-table preparation of raw GC-MS quadrupole data before NOREVA ranking)
- **ggord** (R plotting utility for visualizing workflow comparisons mirroring the paper's Figure 6 layout) — fawda123/ggord
- **NormalizeMets** (Dependency installed for the NOREVA normalization/assessment functions) — metabolomicstats/NormalizeMets

## Examples

```
library(NOREVA); normulticlassisall(fileName, IS)
```

## Evaluation signals

- The first-ranked workflow (e.g. HAM+NON+CUT+[CON-RAN]) yields monotonic processed intensities for spiked aspartic acid and malic acid across the 6/12/25/50 µL dose classes
- The last-ranked workflow (e.g. KNN+NON+LOG+[VAS-LIN]) visibly fails to preserve the dose gradient under the same dose-response fit
- All four pipeline stages are represented in every scored workflow combination (S1 filtering/imputation, S2 QCS correction, S3 transformation, S4 normalization)
- All five NOREVA criteria contribute to the ranking, and any criterion known to be inapplicable (Cc for <6 samples/class, Ce without reference metabolites) is excluded
- Ranking is recomputed on the target dataset rather than imported, since workflow performance is dataset-dependent

## Limitations

- Assessing thousands of workflows is time-consuming — hundreds to thousands of minutes, and >200 min even with parallel computing for large datasets like PMID28528106
- Partial assessment functions reuse output-file names and can overwrite results from previous commands, producing misleading outputs
- Successful application depends on upfront experimental design with both QCSs and ISs, which the tool does not enforce
- The protocol covers only commonly applied processing methods and must be manually updated by bioinformaticians to add new ones

## Evidence

- [discussion] the workflow 'HAM + LLR + NON + [SUM-LEV]' is discovered as the optimal (ranked the first) one: [evidence span withheld — anchor status: not_found]
- [intro] thousands of workflows from combinatorial diversity: [evidence span withheld — anchor status: not_found]
- [readme] five criteria integrated for comprehensive evaluation: [evidence span withheld — anchor status: external_doc]
- [readme] multiclass function scans and ranks thousands of workflows: [evidence span withheld — anchor status: external_doc]
