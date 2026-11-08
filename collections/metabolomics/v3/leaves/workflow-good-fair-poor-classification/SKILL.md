---
name: workflow-good-fair-poor-classification
description: Use when one or more normalized peak tables produced by candidate processing
  workflows (S1–S4 outputs) must be evaluated for study-specific performance and ranked
  via the five-criterion Good/Fair/Poor scheme of NOREVA.
when_to_use_negative:
- Any class or timepoint in the dataset has fewer than 6 samples — criterion Cc (CWrel)
  is unsuitable and must be skipped
- No reference metabolites (e.g., spike-ins/internal standards) are available — criterion
  Ce (LogFC) cannot be applied, so the full five-criterion labeling is not feasible
- The input is still raw instrument data (e.g., vendor-format MS files) — the skill
  expects an already-normalized peak table, and raw-to-open-format conversion/preparation
  is handled upstream (Proteowizard, XCMS, NMRProc Flow)
tools:
- name: NOREVA
  role: R package implementing the five-criterion assessment functions (e.g., normulticlassqcall,
    nortimecourseqcall, normulticlassisall) that compute Ca–Ce and produce the Good/Fair/Poor
    labels
  repo: github.com/idrblab/NOREVA
- name: R
  role: Runtime environment in which the R-based NOREVA assessment protocol is executed
    locally to save computing time and protect data privacy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:9f9bbe944c4781b4c987ff2b9e869e1f5e3216e532b0c824b92729510ca2df2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NOREVA
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# workflow-good-fair-poor-classification

## Summary

NOREVA's final (S5) performance-assessment stage: score each candidate metabolomic processing workflow with five independent criteria — Ca (PMAD), Cb (purity), Cc (CWrel), Cd (AUC), Ce (LogFC) — and label it Good/Fair/Poor. This is how the protocol picks the study-specific optimal workflow instead of relying on a single metric.

## When to use

Use when one or more normalized peak tables produced by candidate processing workflows (S1–S4 outputs) must be evaluated for study-specific performance and ranked via the five-criterion Good/Fair/Poor scheme of NOREVA.

## When NOT to use

- Any class or timepoint in the dataset has fewer than 6 samples — criterion Cc (CWrel) is unsuitable and must be skipped
- No reference metabolites (e.g., spike-ins/internal standards) are available — criterion Ce (LogFC) cannot be applied, so the full five-criterion labeling is not feasible
- The input is still raw instrument data (e.g., vendor-format MS files) — the skill expects an already-normalized peak table, and raw-to-open-format conversion/preparation is handled upstream (Proteowizard, XCMS, NMRProc Flow)

## Inputs

- Normalized peak table (output of the upstream normalization stage S4) in NOREVA's unified row=sample / column=metabolite format
- Label file mapping samples to class/time-point groups (for multi-class or time-course data)
- Study assumption flags SAalpha, SAbeta, SAgamma (all metabolites equally important; constant abundance; majority of metabolites unchanged)
- Column indices of internal-standard (IS) metabolites, when ISs are present
- Reference/spike-in metabolite annotations, required only for criterion Ce (LogFC)

## Outputs

- Per-criterion score values Ca (PMAD), Cb (purity), Cc (CWrel), Cd (AUC), Ce (LogFC) for every candidate workflow
- Good/Fair/Poor classification label per workflow
- criterion_scores.csv aggregating criterion values and resulting labels
- Ranked list of processing workflows by assessed performance

## How to apply

Load each processed peak table (the S4-normalized output) into R via the NOREVA package, preserving the row/column structure (rows = samples by name, columns = metabolites), and select the NOREVA function matching the study design (multi-class or time-course; with QCS, without QCS, with internal standards). Compute criterion a (Ca, PMAD) for within-group variability reproducibility, criterion b (Cb, purity), and criterion c (Cc, CWrel) — skipping Cc entirely when any class or timepoint has fewer than 6 samples. Compute criterion d (Cd, AUC) and criterion e (Ce, LogFC), applying Ce only when reference (spike-in) metabolites are available; if none exist in the dataset, proceed with four criteria. Aggregate the per-criterion values per workflow and apply the protocol's Good/Fair/Poor classification cutoffs to assign each workflow a label, then rank workflows (e.g. the benchmark PMID28528106 case ranked 'HAM + LLR + NON + [SUM-LEV]' first out of 5,904 workflows). Validate by reproducing the tabulated Ca–Ce values and identical Good/Fair/Poor labels for the benchmark workflows (Table 3) and writing results to criterion_scores.csv.

## Related tools

- **NOREVA** (R package implementing the five-criterion assessment functions (e.g., normulticlassqcall, nortimecourseqcall, normulticlassisall) that compute Ca–Ce and produce the Good/Fair/Poor labels) — github.com/idrblab/NOREVA
- **R** (Runtime environment in which the R-based NOREVA assessment protocol is executed locally to save computing time and protect data privacy)

## Examples

```
normulticlassqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Computed Ca–Ce values for the benchmark workflows reproduce the values tabulated in Table 3 of the protocol
- The Good/Fair/Poor labels assigned to benchmark outputs are identical to those in Table 3
- criterion_scores.csv is produced with per-workflow criterion scores and labels
- Criterion Cc is absent (skipped) for any dataset where a class or timepoint has <6 samples
- Criterion Ce is absent for datasets without reference/spike-in metabolites, with the assessment reported over the remaining criteria

## Limitations

- Datasets with small sample size (<6) in one timepoint or class are unsuitable for criterion Cc assessment
- Without a reference metabolite (e.g., spike-in), criterion e cannot conduct the assessment, so the full five-criterion evaluation cannot be completed
- Partial assessment functions generate output file names identical to those of earlier steps, which can overwrite existing output files and produce misleading results

## Evidence

- [readme] Five criteria with distinct theories: [evidence span withheld — anchor status: external_doc]
- [discussion] Ce requires reference metabolites: [evidence span withheld — anchor status: not_found]
- [discussion] Output overwrite failure mode: [evidence span withheld — anchor status: not_found]
