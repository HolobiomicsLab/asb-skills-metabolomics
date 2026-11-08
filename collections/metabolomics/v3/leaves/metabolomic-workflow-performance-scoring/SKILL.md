---
name: metabolomic-workflow-performance-scoring
description: Use when multiple processed peak tables from candidate normalization/QCS/transformation
  workflows are available and each must be ranked study-specifically to pick the -performing
  workflow before downstream statistics or biomarker discovery.
when_to_use_negative:
- The dataset has fewer than 6 samples in one timepoint or class — criterion Cc is
  unsuitable and cannot be computed, so the full five-criteria assessment cannot be
  applied
- No reference metabolites (e.g., spike-ins / internal standards) are provided — criterion
  Ce (LogFC) cannot be applied and only the four remaining criteria are usable
- The goal is raw-data processing (file conversion, peak picking) rather than evaluating
  already-processed peak tables — use upstream tools like Proteowizard or XCMS instead
  of this assessment stage
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NOREVA
  role: R package and web tool implementing the five-criteria (Ca–Ce) performance
    assessment and Good/Fair/Poor classification of processing workflows
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: environment in which the R-based NOREVA protocol is run locally for the performance-assessment
    stage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:3453d91d95e68298c3fcf11286655f369de62f52a238cf7e81fc4f91e96ec320
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomic-workflow-performance-scoring

## Summary

NOREVA's final assessment stage (S5) scores each candidate metabolomic processing workflow using five independent criteria (Ca/PMAD, Cb/purity, Cc/CWrel, Cd/AUC, Ce/LogFC) and classifies outcomes as Good/Fair/Poor, enabling study-specific selection of the optimal workflow.

## When to use

Use when multiple processed peak tables from candidate normalization/QCS/transformation workflows are available and each must be ranked study-specifically to pick the best-performing workflow before downstream statistics or biomarker discovery.

## When NOT to use

- The dataset has fewer than 6 samples in one timepoint or class — criterion Cc is unsuitable and cannot be computed, so the full five-criteria assessment cannot be applied
- No reference metabolites (e.g., spike-ins / internal standards) are provided — criterion Ce (LogFC) cannot be applied and only the four remaining criteria are usable
- The goal is raw-data processing (file conversion, peak picking) rather than evaluating already-processed peak tables — use upstream tools like Proteowizard or XCMS instead of this assessment stage

## Inputs

- processed peak table for each candidate workflow (unified NOREVA format: rows = samples, columns = metabolites), output of stages S1–S4
- sample label file for time-course/multi-class datasets (via PrepareInuputFiles)
- column indices of internal standards (IS) in the peak table, when reference/spike-in metabolites are available for criterion Ce

## Outputs

- per-criterion scores (Ca PMAD, Cb purity, Cc CWrel, Cd AUC, Ce LogFC) for every scanned workflow
- workflow ranking based on multi-criteria performance
- Good/Fair/Poor classification label per workflow (e.g., written to criterion_scores.csv)

## How to apply

Load each processed peak table (output of the upstream normalization stage S4) into R via the NOREVA package, preserving the row/column structure (rows = samples by name, columns = metabolites). Compute criterion a (Ca, PMAD) quantifying reproducibility of within-group variability, criterion b (Cb, purity), and criterion c (Cc, CWrel) — skipping Cc when any class or timepoint has fewer than 6 samples. Compute criterion d (Cd, AUC) and criterion e (Ce, LogFC), applying Ce only when reference (spike-in) metabolites are available. Aggregate per-criterion results for each workflow and apply NOREVA's Good/Fair/Pair classification cutoffs to label every candidate; the multi-criteria design is required because each criterion has a distinct underlying theory and any single criterion is insufficient. Validate by checking that computed Ca–Ce values reproduce the tabulated benchmark values (e.g., Table 3) and yield identical Good/Fair/Poor labels.

## Related tools

- **NOREVA** (R package and web tool implementing the five-criteria (Ca–Ce) performance assessment and Good/Fair/Poor classification of processing workflows) — https://github.com/idrblab/NOREVA
- **R** (environment in which the R-based NOREVA protocol is run locally for the performance-assessment stage)

## Examples

```
library(NOREVA); data <- PrepareInuputFiles(dataformat=1, rawdata="peak_table.csv", label="labels.csv"); normulticlassqcall(data, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Computed Ca–Ce values for benchmark workflows reproduce the tabulated values (e.g., Table 3 of the paper) and yield identical Good/Fair/Poor labels
- Results are written to criterion_scores.csv with one score set per candidate workflow
- Criterion Cc is skipped (not set to a value) for any dataset with <6 samples in a class or timepoint; criterion Ce is skipped when no spike-in/reference metabolites exist
- The optimal workflow identified (e.g., HAM+LLR+NON+[SUM-LEV] for PMID28528106 among 5,904 scanned) matches the known top-ranked combination
- Dataset-dependence check: a workflow scoring 'Good' on one benchmark may score 'Fair'/'Poor' on another (e.g., MDI+NON+NON+[CON-AUT] on PMID29520289 vs PMID18582365), confirming study-specific rather than fixed scoring

## Limitations

- Assessing thousands of workflows is time-consuming: hundreds to thousands of minutes, and >200 min for large datasets (e.g., PMID28528106) despite parallel computing and memory management
- Partial assessment functions can overwrite output files from previous commands (identical filenames to Step 10 outputs), producing misleading outputs
- Successful application depends on upfront experimental design that adopts both QCSs and ISs; the tool does not enforce this
- Coverage is limited to commonly applied processing methods and cannot be extended automatically; new methods require manual updates by bioinformaticians

## Evidence

- [intro] five independent criteria with distinct theories: [evidence span withheld — anchor status: not_found]
- [discussion] criterion Ce reference requirement: [evidence span withheld — anchor status: not_found]
- [discussion] optimality example: [evidence span withheld — anchor status: not_found]
- [readme] five criteria in README: [evidence span withheld — anchor status: external_doc]
