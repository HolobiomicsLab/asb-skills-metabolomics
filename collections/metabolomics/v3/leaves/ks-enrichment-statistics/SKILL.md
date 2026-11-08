---
name: ks-enrichment-statistics
description: Use when you have a CSA- or IDSL.CSA-annotated compound matrix (InChIKey,
  SMILES, precursor type, molecular formula) and need to identify which chemical classes
  are significantly different between disease and control groups.
when_to_use_negative:
- When the input is already a peak table without chemical annotation (no InChIKey/SMILES)
  — first run IDSL.CSA to aggregate annotated chemical structures
- When the analysis goal is peak detection or chemical annotation itself rather than
  downstream class-level statistical enrichment
- When there are no defined biological groups to compare (e.g. only a single sample
  condition) — KS enrichment requires a ranked differential-abundance list between
  groups
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0202
tools:
- name: ChemRICH
  role: Kolmogorov–Smirnov set enrichment of chemically similar compound families
    on a ranked differential-abundance list
- name: IDSL.CSA
  role: Generates the CSA-annotated aligned compound matrix that serves as the input
    for the enrichment analysis
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: R
  role: Execution environment for the enrichment workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:2dee532f5315cdfc0109390755b424dff37ddcb22fc349e8725fdc3370cee098
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemRICH
  - IDSL.CSA
  - R
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ks-enrichment-statistics

## Summary

A ChemRICH-based Kolmogorov–Smirnov set-enrichment skill that ranks CSA-annotated compounds by differential abundance and tests which chemical classes are significantly altered between two sample groups.

## When to use

Use when you have a CSA- or IDSL.CSA-annotated compound matrix (InChIKey, SMILES, precursor type, molecular formula) and need to identify which chemical classes are significantly different between disease and control groups.

## When NOT to use

- When the input is already a peak table without chemical annotation (no InChIKey/SMILES) — first run IDSL.CSA to aggregate annotated chemical structures
- When the analysis goal is peak detection or chemical annotation itself rather than downstream class-level statistical enrichment
- When there are no defined biological groups to compare (e.g. only a single sample condition) — KS enrichment requires a ranked differential-abundance list between groups

## Inputs

- CSA-annotated aligned compound matrix (with meta-variables InChIKey, SMILES, precursor type, molecular formula)
- Sample group labels (e.g. Crohn's disease vs healthy controls)

## Outputs

- chemrich_results.csv containing class-level enrichment results (significantly altered chemical families)

## How to apply

Load the CSA-annotated aligned compound matrix from the IDSL.CSA output (meta-variables: InChIKey, SMILES, precursor type, molecular formula) for the study of interest. Compute per-compound differential abundance between the two groups (e.g. Crohn's disease vs healthy controls) and rank compounds by effect size. Cluster the annotated compounds into chemically similar families and run the ChemRICH Kolmogorov–Smirnov enrichment test on the ranked list. Apply multiple-testing correction across chemical families and retain the significantly altered classes. In the ST001000 case this recovered cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, and vitamins as significantly different between Crohn's disease and controls.

## Related tools

- **ChemRICH** (Kolmogorov–Smirnov set enrichment of chemically similar compound families on a ranked differential-abundance list)
- **IDSL.CSA** (Generates the CSA-annotated aligned compound matrix that serves as the input for the enrichment analysis) — https://cran.r-project.org/package=IDSL.CSA
- **R** (Execution environment for the enrichment workflow)

## Examples

```
R: library(ChemRICH); res <- chemRICH(ranked_list = differential_scores, compound_meta = csa_matrix[, c("InChIKey","SMILES","precursor_type","molecular_formula")]); write.csv(res, "chemrich_results.csv")
```

## Evaluation signals

- The recovered significantly altered chemical classes should reproduce the reported set of cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, and vitamins for ST001000 (Crohn's disease vs healthy controls)
- chemrich_results.csv is produced and contains class-level enrichment entries
- Compounds are correctly grouped into chemically similar families before testing (using InChIKey/SMILES-based clustering)
- Multiple-testing correction is applied across chemical families and only significant classes are retained

## Limitations

- ChemRICH class-level enrichment depends on the quality and coverage of the upstream chemical annotation — compounds lacking InChIKey/SMILES cannot be clustered into chemical families
- The skill is validated on CSA-annotated LC/HRMS data; it presumes the IDSL.CSA cross-applicability model where libraries built from one study are applied to independent datasets
- Differential-abundance ranking method (e.g. effect size definition) is not specified in the article and must be chosen by the practitioner

## Evidence

- [results] ChemRICH analysis of the CSA-annotated ST001000 compounds revealed that cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, and vitamins were significantly different between Crohn's disease patients and healthy controls.: [evidence span withheld — anchor status: not_found]
