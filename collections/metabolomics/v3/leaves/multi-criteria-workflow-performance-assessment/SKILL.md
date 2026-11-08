---
name: multi-criteria-workflow-performance-assessment
description: Use when a metabolomic peak table has been processed by multiple candidate
  workflows (filtering/imputation, QCS correction, transformation, normalization combinations)
  and you must rank or grade which workflow performs for that specific study.
when_to_use_negative:
- The dataset has fewer than 6 samples in one timepoint or class — criterion Cc is
  unsuitable for such small-sample data
- No reference metabolites (e.g., spike-ins) are provided — criterion Ce (LogFC) cannot
  be applied, so the full five-criteria evaluation is impossible
- The goal is raw-file conversion or peak-table generation rather than judging already-processed
  workflows — use Proteowizard/XCMS-type preparation tools instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NOREVA
  role: R package providing the built-in multi-criteria assessment functions (normulticlassqcall,
    normulticlassnoall, normulticlassisall, nortimecourseqcall, nortimecoursenoall)
    that score and rank workflows in stage S5
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Runtime environment in which the NOREVA assessment functions are executed
    locally to protect data privacy and save computing time
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:d6b9e93ba8963a401f04d052b937a7fea65a90c211881c1a94000d1e06aaaece
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multi-criteria-workflow-performance-assessment

## Summary

Assess candidate metabolomic peak-table processing workflows (e.g., MDI+NON+NON+[CON-AUT], HAM+LLR+NON+[SUM-LEV]) against five independent performance criteria (Ca–Cd plus LogFC) in stage S5 of NOREVA, converting numeric scores into Good/Fair/Poor classifications so the best workflow for a specific study can be identified among thousands of combinatorial candidates.

## When to use

Use when a metabolomic peak table has been processed by multiple candidate workflows (filtering/imputation, QCS correction, transformation, normalization combinations) and you must rank or grade which workflow performs best for that specific study.

## When NOT to use

- The dataset has fewer than 6 samples in one timepoint or class — criterion Cc is unsuitable for such small-sample data
- No reference metabolites (e.g., spike-ins) are provided — criterion Ce (LogFC) cannot be applied, so the full five-criteria evaluation is impossible
- The goal is raw-file conversion or peak-table generation rather than judging already-processed workflows — use Proteowizard/XCMS-type preparation tools instead

## Inputs

- Processed peak-table outputs from each candidate workflow (rows = samples, columns = metabolites, NOREVA unified format)
- Label file indicating class or time-point membership for time-course/multi-class data
- Internal-standard column indices (if ISs are present)
- Study assumption flags (SAalpha, SAbeta, SAgamma)

## Outputs

- Numeric performance scores per workflow per criterion (Ca–Cd, and Ce/LogFC when reference metabolites exist)
- Overall ranking of all scanned workflows (e.g., ranking_table.csv reproducing Figure 4 ordering)
- Good/Fair/Poor classification per workflow–dataset pair

## How to apply

After running all candidate workflows through the four processing stages (S1 data filtering and imputation, S2 QCS correction, S3 data transformation, S4 data normalization after determining the study assumption alpha/beta/gamma), apply NOREVA's built-in assessment functions in stage S5 to score each processed output against five well-established criteria, each with a distinct underlying theory. Choose the function matching the data type and composition: normulticlassqcall/nortimecourseqcall for datasets with quality control samples but no internal standards, normulticlassisall for datasets with internal standards but no QCS, or normulticlassnoall/nortimecoursenoall for datasets with neither. Pass the study assumptions (SAalpha, SAbeta, SAgamma) as Y/N flags and the IS column numbers where applicable. Note that criterion Cc is unsuitable for datasets with fewer than 6 samples per class/timepoint, and criterion Ce (LogFC) cannot be applied when no reference metabolites such as spike-ins are available. Finally, convert numeric scores into the Good/Fair/Poor classification and compare against rankings such as Figure 4 of PMID28528106, where HAM+LLR+NON+[SUM-LEV] ranks first of 5,904 scanned workflows.

## Related tools

- **NOREVA** (R package providing the built-in multi-criteria assessment functions (normulticlassqcall, normulticlassnoall, normulticlassisall, nortimecourseqcall, nortimecoursenoall) that score and rank workflows in stage S5) — https://github.com/idrblab/NOREVA
- **R** (Runtime environment in which the NOREVA assessment functions are executed locally to protect data privacy and save computing time)

## Examples

```
library(NOREVA); normulticlassqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Reproduced Good/Fair/Poor classifications match Table 3 for each workflow–dataset pair, with agreements and discrepancies recorded
- ranking_table.csv reproduces the Figure 4 ordering for PMID28528106, with HAM+LLR+NON+[SUM-LEV] at rank 1 of 5,904 workflows and KNN+NWE+LOG+[PAR-LIN] at the last rank
- Every candidate workflow was assessed collectively against the five independent criteria, not a single criterion
- Assessment output files are checked against Step 10 outputs to detect overwriting by identical output-file names from later commands
- Dataset-dependent sanity check: a workflow that scored 'Fair'/'Poor' on another dataset (e.g., MDI+NON+NON+[CON-AUT] on PMID18582365) is not assumed to transfer

## Limitations

- Assessing thousands of workflows is very time-consuming, taking hundreds to thousands of minutes even with parallel computing; datasets like PMID28528106 still require >200 min despite parallel computing and memory management
- Partial assessment functions can overwrite output files from previous commands (identical names to Step 10 outputs), producing misleading outputs
- Only commonly applied processing methods are covered; extension requires manual updates by bioinformaticians

## Evidence

- [readme] Five criteria with distinct theories in README: [evidence span withheld — anchor status: external_doc]
- [discussion] Optimal workflow finding: [evidence span withheld — anchor status: not_found]
- [readme] Assessment function signature for multi-class with QCS: [evidence span withheld — anchor status: external_doc]
