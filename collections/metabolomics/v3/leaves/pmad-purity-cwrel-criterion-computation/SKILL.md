---
name: pmad-purity-cwrel-criterion-computation
description: Use when a processed peak table (output of upstream normalization, S4)
  must be scored study-specifically across multiple criteria to rank or label candidate
  metabolomic processing workflows as Good/Fair/Poor.
when_to_use_negative:
- Any one class or timepoint in the dataset has fewer than 6 samples — criterion Cc
  (CWrel) is unsuitable for such small-sample datasets
- No reference metabolites (e.g., spike-in internal standards) are available — criterion
  e (LogFC) cannot be applied, so a four-criterion assessment is used instead
- The goal is upstream data processing (filtering, imputation, QCS correction, transformation,
  normalization) rather than assessment of already-processed workflow outputs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package providing the criterion-computation and workflow-ranking functions
    for the S5 performance assessment stage
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Runtime environment for the NOREVA R-based protocol used to compute Ca–Ce
    locally
  repo: https://github.com/idrblab/NOREVA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:accd41c0943c5e4ca7f75ba62ac68f22032743d6c081feb64a02136f87436bf2
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pmad-purity-cwrel-criterion-computation

## Summary

Implements the NOREVA protocol's final stage (S5), scoring each candidate processing workflow on five independent criteria (Ca/PMAD, Cb/purity, Cc/CWrel, Cd/AUC, Ce/LogFC) and classifying outcomes as Good/Fair/Poor.

## When to use

Use when a processed peak table (output of upstream normalization, S4) must be scored study-specifically across multiple criteria to rank or label candidate metabolomic processing workflows as Good/Fair/Poor.

## When NOT to use

- Any one class or timepoint in the dataset has fewer than 6 samples — criterion Cc (CWrel) is unsuitable for such small-sample datasets
- No reference metabolites (e.g., spike-in internal standards) are available — criterion e (LogFC) cannot be applied, so a four-criterion assessment is used instead
- The goal is upstream data processing (filtering, imputation, QCS correction, transformation, normalization) rather than assessment of already-processed workflow outputs

## Inputs

- Processed peak table in NOREVA's unified format (rows = samples by name, columns = metabolites), produced by the S4 normalization stage
- Label file indicating class/timepoint membership for multi-class or time-course data
- Column indices of internal standard (IS) metabolites, when spike-ins are available (e.g., 2,6,9,n)

## Outputs

- Per-criterion values (Ca–Ce) for each candidate processing workflow
- criterion_scores.csv containing aggregated criterion scores and Good/Fair/Poor labels per workflow

## How to apply

Load the processed peak table into R via the NOREVA R package, preserving row/column structure (rows = samples by name, columns = metabolites). For each candidate workflow outcome, compute criterion a (Ca, PMAD) quantifying reproducibility of within-group variability, then criterion b (Cb, purity) and criterion c (Cc, CWrel) — skipping Cc when any class or timepoint has fewer than 6 samples. Next compute criterion d (Cd, AUC) and, only when reference (spike-in) metabolites are available, criterion e (Ce, LogFC). Aggregate per-criterion results per workflow and apply the protocol's Good/Fair/Poor classification cutoffs. The multi-criteria design is deliberate: five well-established criteria with distinct underlying theories give a more comprehensive evaluation than any single criterion.

## Related tools

- **NOREVA** (R package providing the criterion-computation and workflow-ranking functions for the S5 performance assessment stage) — https://github.com/idrblab/NOREVA
- **R** (Runtime environment for the NOREVA R-based protocol used to compute Ca–Ce locally) — https://github.com/idrblab/NOREVA

## Examples

```
normulticlassqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Computed Ca–Ce values for benchmark workflow outputs must reproduce the values tabulated in Table 3 of the protocol
- Good/Fair/Poor labels derived from the computed cutoffs must match Table 3 exactly for the benchmark workflows
- Criterion values and labels must be written to criterion_scores.csv without overwriting output files from prior NOREVA steps (output names can collide with Step 10 outputs)
- Cc is omitted (rather than zero-filled) for datasets with <6 samples in any class or timepoint, and Ce is omitted when no reference metabolites exist
- The choice of the four- vs five-criterion function matches the dataset's QC-sample/internal-standard availability (e.g., qcall vs noall vs isall variants)

## Limitations

- Criterion Cc is unsuitable for datasets with small sample size (<6) in any one timepoint or class
- Criterion Ce (LogFC) is infeasible when no reference metabolite (e.g., spike-in) is provided, which was the case for all analyzed benchmark datasets
- Some partial assessment functions generate output files with names identical to those from Step 10 and can overwrite previous outputs, producing misleading results
- Successful application depends on upfront experimental design adopting both QCSs and ISs, which the tool does not enforce

## Evidence

- [readme] Rationale for multi-criteria evaluation: [evidence span withheld — anchor status: external_doc]
- [discussion] Constraint on criterion Ce (LogFC): [evidence span withheld — anchor status: not_found]
- [discussion] Output-overwriting failure mode: [evidence span withheld — anchor status: not_found]
