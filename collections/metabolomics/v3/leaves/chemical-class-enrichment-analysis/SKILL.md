---
name: chemical-class-enrichment-analysis
description: Use when an IDSL.CSA-aligned compound matrix annotated with InChIKey/SMILES/molecular
  formula meta-variables must be clustered into chemical families and statistically
  tested for class-level abundance differences between two sample groups (e.g. Crohn's
  vs healthy controls).
when_to_use_negative:
- When the input is a raw mzXML/mzML/netCDF file rather than an already-aligned, CSA-annotated
  compound matrix — use IDSL.IPA/IDSL.CSA first
- When the question is peak-level (single compound) rather than class-level enrichment
  across chemically similar families
- When MS2-based annotation is required — this skill depends on CSA MS1-derived annotations
  and is not a replacement for MS/MS library searches via IDSL.FSA
edam_operation: http://edamontology.org/operation_3743
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: IDSL.CSA
  role: Produces the CSA-annotated aligned compound matrix with InChIKey/SMILES/molecular-formula
    meta-variables that is the input to the enrichment analysis
  repo: https://github.com/idslme/IDSL.CSA
- name: R
  role: Statistical environment in which the differential abundance ranking, ChemRICH
    enrichment test, and multiple-testing correction are executed
- name: ChemRICH
  role: Set enrichment tool used to run the Kolmogorov–Smirnov test on the effect-size-ranked
    compound list clustered into chemical families
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:45c106eaa4923080a458b666bc45122525c4a3dab34e13120cdaa543b6a7759d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.CSA
  tools:
  - ChemRICH
  - IDSL.CSA
  - R
  edam_operation: http://edamontology.org/operation_3743
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# chemical-class-enrichment-analysis

## Summary

A downstream metabolomics analysis skill that performs ChemRICH set enrichment analysis on chemically similar families of compounds derived from IDSL.CSA-annotated aligned peak tables, using a Kolmogorov–Smirnov test on effect-size-ranked compound lists to identify significantly altered chemical classes between disease and control groups.

## When to use

Use when an IDSL.CSA-aligned compound matrix annotated with InChIKey/SMILES/molecular formula meta-variables must be clustered into chemical families and statistically tested for class-level abundance differences between two sample groups (e.g. Crohn's vs healthy controls).

## When NOT to use

- When the input is a raw mzXML/mzML/netCDF file rather than an already-aligned, CSA-annotated compound matrix — use IDSL.IPA/IDSL.CSA first
- When the question is peak-level (single compound) rather than class-level enrichment across chemically similar families
- When MS2-based annotation is required — this skill depends on CSA MS1-derived annotations and is not a replacement for MS/MS library searches via IDSL.FSA

## Inputs

- CSA-annotated aligned compound matrix (from IDSL.CSA) with meta-variables: InChIKey, SMILES, precursor type, molecular formula
- Sample-group labels (e.g. Crohn's disease vs healthy controls) for the aligned peak table
- Chemically similar family clustering of the annotated compounds (for ChemRICH enrichment)

## Outputs

- Class-level enrichment results (e.g. chemrich_results.csv) listing significantly altered chemical classes
- Ranked compound list by differential abundance effect size
- Set of significantly altered chemical classes (e.g. cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, vitamins)

## How to apply

Start from the aligned compound matrix produced by the IDSL.CSA structure-aggregation step, which carries meta-variables such as InChIKey, SMILES, precursor type, and molecular formula on each aligned LC/HRMS peak. Compute per-compound differential abundance between the case and control samples and rank compounds by effect size. Cluster the annotated compounds into chemically similar families and run the ChemRICH Kolmogorov–Smirnov enrichment test on this ranked compound list, as demonstrated for study ST001000 where cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, and vitamins were recovered as significantly altered classes between Crohn's disease patients and healthy controls. Apply multiple-testing correction across chemical families to control the false discovery rate, then retain the significantly altered classes. The rationale, per the article, is that cross-applicability of CSA-derived annotations enables biological insight that would otherwise be missed when MS2 fragmentation data are absent.

## Related tools

- **IDSL.CSA** (Produces the CSA-annotated aligned compound matrix with InChIKey/SMILES/molecular-formula meta-variables that is the input to the enrichment analysis) — https://github.com/idslme/IDSL.CSA
- **R** (Statistical environment in which the differential abundance ranking, ChemRICH enrichment test, and multiple-testing correction are executed)
- **ChemRICH** (Set enrichment tool used to run the Kolmogorov–Smirnov test on the effect-size-ranked compound list clustered into chemical families)

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("CSA_parameters.xlsx") # then rank compounds by effect size and run ChemRICH on the InChIKey/SMILES-annotated aligned table
```

## Evaluation signals

- The output should recover the reported significantly altered classes for ST001000: cholic acids, amino acids, aminosalicylic acids, biogenic amines, hexosamines, and vitamins
- The ranked compound list must be sorted by effect size before running the ChemRICH Kolmogorov–Smirnov test
- Multiple-testing correction must be applied across chemical families before declaring significance
- The input matrix must carry CSA meta-variables (InChIKey, SMILES, precursor type, molecular formula) on each aligned compound

## Limitations

- ChemRICH enrichment depends on prior CSA annotation quality; not all LC/HRMS peaks are annotated because MS2 fragmentation data are not available for every detected peak
- Available MS2 libraries poorly cover the expected chemical space, so classes relying solely on MS2-based annotation may be underrepresented in the enrichment analysis
- Enrichment results are specific to the compound clustering used; different clustering schemes may yield different significant classes

## Evidence

- [abstract] Cross-applicability of these libraries across independent studies may provide access to new biological insights that may be missed due to the lack of MS2 fragmentation data: "The cross-applicability of these libraries across independent studies may provide access to new biological insights that may be missed due to the lack of MS2 fragmentation data"
- [methods] Aggregating annotated chemical structures on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, molecular formula: "Aggregating annotated chemical structures on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, molecular formula"
- [intro] not all peaks in an LC/HRMS dataset are annotated with chemical information, mainly for two reasons 1) MS2 fragmentation data are not available for every detected LC/HRMS peak: "not all peaks in an LC/HRMS dataset are annotated with chemical information, mainly for two reasons 1) MS2 fragmentation data are not available for every detected LC/HRMS peak"
- [readme] Features of IDSL.CSA - Aggregating annotated chemical structures on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, molecular formula: "Aggregating annotated chemical structures on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, molecular formula,... depending on the information in the reference"
