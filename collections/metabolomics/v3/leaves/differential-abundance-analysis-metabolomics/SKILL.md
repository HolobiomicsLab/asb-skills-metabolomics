---
name: differential-abundance-analysis-metabolomics
description: Use when you have a group-labelled, CSA-annotated aligned compound matrix
  (InChIKey/SMILES/formula meta-variables) and need per-compound differential statistics
  plus chemical-class enrichment (e.g. ChemRICH) between disease and control groups.
when_to_use_negative:
- Input is an unannotated aligned peak table without InChIKey/SMILES/formula meta-variables
  — differential analysis alone will not support chemical-class enrichment.
- The dataset is MS2-library-annotated but you need a chemical-class enrichment view
  rather than a simple differential test — this skill's value is the CSA-compound-family
  layer.
- There are no group labels (e.g. no disease/control split) — differential abundance
  is undefined.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: IDSL.CSA
  role: 'Generates the CSA-annotated aligned compound matrix (meta-variables: InChIKey,
    SMILES, precursor type, molecular formula) that is the prerequisite input for
    class-level differential enrichment.'
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: R
  role: Analysis environment for running IDSL.CSA, ChemRICH set enrichment, and writing
    chemrich_results.csv.
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: ChemRICH
  role: Performs set enrichment (Kolmogorov–Smirnov test) on the ranked differential
    compound list to identify significantly altered chemical families.
- name: IDSL.IPA
  role: Upstream peak-detection/deconvolution tool that produces the peaklists and
    peak_alignment directories feeding the CSA workflow.
  repo: https://github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:fe0f3ff30cd259818176e373811b49ff962fb32bca837f094194e1741fa05f8e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemRICH
  - IDSL.CSA
  - IDSL.IPA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# differential-abundance-analysis-metabolomics

## Summary

Tests which annotated metabolites (e.g. CSA-annotated compound families) differ significantly between sample groups such as disease vs. controls, then enriches those hits into chemical-class-level conclusions (e.g. via ChemRICH set enrichment on CSA compound sets).

## When to use

Use when you have a group-labelled, CSA-annotated aligned compound matrix (InChIKey/SMILES/formula meta-variables) and need per-compound differential statistics plus chemical-class enrichment (e.g. ChemRICH) between disease and control groups.

## When NOT to use

- Input is an unannotated aligned peak table without InChIKey/SMILES/formula meta-variables — differential analysis alone will not support chemical-class enrichment.
- The dataset is MS2-library-annotated but you need a chemical-class enrichment view rather than a simple differential test — this skill's value is the CSA-compound-family layer.
- There are no group labels (e.g. no disease/control split) — differential abundance is undefined.

## Inputs

- CSA-annotated aligned compound matrix (meta-variables: InChIKey, SMILES, precursor type, molecular formula) for a study such as ST001000
- Sample-group labels (e.g. Crohn's disease vs. healthy control) per sample
- Chemical reference/family definitions for clustering annotated compounds into chemically similar sets (e.g. ChemRICH class definitions)

## Outputs

- Per-compound differential abundance statistics ranked by effect size
- Class-level enrichment results saved as chemrich_results.csv listing significantly altered chemical families
- Significantly altered chemical classes (e.g. cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, vitamins)

## How to apply

Start from the CSA-annotated aligned compound table produced by IDSL.CSA (meta-variables: InChIKey, SMILES, precursor type, molecular formula) so each row is a chemically identified compound rather than an unannotated m/z-RT peak. Compute per-compound differential abundance between the phenotypic groups (e.g. Crohn's disease vs. healthy controls from study ST001000) and rank compounds by effect size. Group annotated compounds into chemically similar families and run ChemRICH set enrichment, which uses a Kolmogorov–Smirnov test on the ranked compound list. Apply multiple-testing correction across chemical families and retain the significantly altered classes (e.g. cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, vitamins). Rationale: the article's motivating gap is that many LC/HRMS peaks lack chemical annotation; using the CSA-aggregated, structure-annotated table gives differential analysis a chemical-class dimension that unannotated peak tables cannot support.

## Related tools

- **IDSL.CSA** (Generates the CSA-annotated aligned compound matrix (meta-variables: InChIKey, SMILES, precursor type, molecular formula) that is the prerequisite input for class-level differential enrichment.) — https://cran.r-project.org/package=IDSL.CSA
- **R** (Analysis environment for running IDSL.CSA, ChemRICH set enrichment, and writing chemrich_results.csv.) — https://cran.r-project.org/package=IDSL.CSA
- **ChemRICH** (Performs set enrichment (Kolmogorov–Smirnov test) on the ranked differential compound list to identify significantly altered chemical families.)
- **IDSL.IPA** (Upstream peak-detection/deconvolution tool that produces the peaklists and peak_alignment directories feeding the CSA workflow.) — https://github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.CSA); # after producing CSA-annotated compound matrix and computing per-compound effect sizes, run ChemRICH on the ranked list and save: write.csv(chemrich_results, "chemrich_results.csv")
```

## Evaluation signals

- Per-compound differential table contains both effect-size estimates and a ranking column — no row should be missing an effect size.
- ChemRICH output contains a corrected p-value column; only families with significant (post-correction) results should be reported as altered.
- The set of significantly altered classes should match the reported expectations for the study (e.g. cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, vitamins for ST001000).
- chemrich_results.csv is produced and contains class-level entries rather than individual peaks/compounds.
- Input matrix is confirmed to carry InChIKey/SMILES/precursor-type/molecular-formula meta-variables; if any are absent the class aggregation step cannot be valid.

## Limitations

- Only annotated compounds (carrying InChIKey/SMILES/meta-variables from the CSA library) can participate in class-level enrichment; unannotated peaks are excluded by construction.
- Class enrichment granularity is limited by the chemical-family definitions supplied to ChemRICH — the reported classes reflect the chosen ontology of chemically similar families.
- Differential statistics depend on per-compound effect sizes computed upstream; the skill does not itself prescribe the statistical model for the per-compound test.

## Evidence

- [other] ChemRICH analysis of the CSA-annotated ST001000 compounds revealed that cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, and vitamins were significantly different between Crohn's disease patients and healthy controls.: [evidence span withheld — anchor status: not_found]
- [other] CSA-annotated compound matrix with meta-variables for class-level differential enrichment: [evidence span withheld — anchor status: not_found]
- [other] ChemRICH set enrichment analysis uses Kolmogorov–Smirnov test on ranked compound list: [evidence span withheld — anchor status: not_found]
- [other] Multiple-testing correction across chemical families is required before reporting altered classes: [evidence span withheld — anchor status: not_found]
