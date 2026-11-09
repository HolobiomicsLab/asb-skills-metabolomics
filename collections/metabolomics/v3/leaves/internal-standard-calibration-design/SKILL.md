---
name: internal-standard-calibration-design
description: Use when a mass-spectrometry metabolomics study requires absolute concentrations
  of selected metabolites (e.g., oleic, arachidonic, docosahexaenoic acids, creatine)
  to validate untargeted profiling between molecularly defined tumor subgroups.
when_to_use_negative:
- The goal is untargeted metabolite discovery rather than absolute validation of a
  pre-specified metabolite panel
- The input is already an absolute concentration table — re-running calibration design
  would duplicate the assay rather than validate it
- Group sample sizes or subgroup labels (e.g., phosphoAKT1/MYC k-means clusters) have
  not yet been defined, so no directed comparison can be tested
edam_operation: http://edamontology.org/operation_3660
edam_topics: []
tools:
- name: GraphPad Prism
  role: Runs the Mann-Whitney Test used to compare absolute metabolite concentrations
    between phosphoAKT1/MYC tumor subgroups at the p<0.05 threshold
- name: Python
  role: Assembles the five-metabolite validation table (metabolite, pathway annotation,
    direction, n, units, test, threshold) and checks traceability of each row to the
    article text
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:648dabdc1fe380131fec0dcc776a54695b8028c1038f3a6907bd254db397f518
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - Python
  edam_operation: http://edamontology.org/operation_3660
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# internal-standard-calibration-design

## Summary

Design and report targeted absolute-quantitation assays that validate untargeted metabolomics findings, using metabolite-specific internal standards, µg/g tissue concentration units, and nonparametric group comparisons (Mann-Whitney, p<0.05).

## When to use

Use when a mass-spectrometry metabolomics study requires absolute concentrations of selected metabolites (e.g., oleic, arachidonic, docosahexaenoic acids, creatine) to validate untargeted profiling between molecularly defined tumor subgroups.

## When NOT to use

- The goal is untargeted metabolite discovery rather than absolute validation of a pre-specified metabolite panel
- The input is already an absolute concentration table — re-running calibration design would duplicate the assay rather than validate it
- Group sample sizes or subgroup labels (e.g., phosphoAKT1/MYC k-means clusters) have not yet been defined, so no directed comparison can be tested

## Inputs

- Untargeted mass-spectrometry metabolomics profiles that nominate metabolites for validation
- Tumor/normal tissue specimens with molecular subgroup labels (e.g., phosphoAKT1-high/MYC-low vs. phosphoAKT1-low/MYC-high from K-means clustering of immunoblot densitometry)
- Metabolite-specific internal standards for each targeted analyte
- Methods text specifying the assay panel, µg/g tissue units, statistical test, and significance threshold

## Outputs

- Absolute concentration table (µg/g tissue) per metabolite per sample
- Structured validation table: metabolite, pathway annotation, group comparison direction, sample sizes, units, statistical test, significance threshold
- Significance calls per metabolite (Mann-Whitney Test, p<0.05), e.g., oleic/arachidonic/DHA significantly higher in MYC-high tumors

## How to apply

Parse the 'Absolute Quantitation Of Metabolites' methods subsection to fix the assay panel (here: oleic acid, arachidonic acid, docosahexaenoic acid, creatine, 2-aminoadipic acid), the µg/g tissue unit convention, and the metabolite-specific internal standards used for calibration. Extract sample-size statements — in this study 56 samples (40 tumors, 16 normals), with the fatty-acid validation run on phosphoAKT1-high/MYC-low n=14 vs phosphoAKT1-low/MYC-high n=5 tumors — before running any statistics. Measure absolute concentrations by mass spectrometry as a validation of the untargeted metabolomics, then compare subgroup concentrations with the Mann-Whitney Test at the p<0.05 significance threshold (analysis performed in GraphPad Prism). Assemble results into a structured table with metabolite name, pathway annotation (e.g., SCD1-derived fatty acid vs. lysine intermediate vs. creatine), direction of change, group sample sizes, units, test, and threshold. Validate that each row is traceable to a verbatim claim in the article text and that directions are consistent with the MYC/dysregulated-lipid-metabolism and AKT1/aerobic-glycolysis findings.

## Related tools

- **GraphPad Prism** (Runs the Mann-Whitney Test used to compare absolute metabolite concentrations between phosphoAKT1/MYC tumor subgroups at the p<0.05 threshold)
- **Python** (Assembles the five-metabolite validation table (metabolite, pathway annotation, direction, n, units, test, threshold) and checks traceability of each row to the article text)

## Evaluation signals

- Every metabolite row in the output table is traceable to a verbatim claim in the article text, and the direction summary is consistent with the stated MYC-overexpression/dysregulated-lipid-metabolism and AKT1-glycolysis findings
- All concentrations are reported in the µg/g tissue unit convention with the metabolite-specific internal standards named
- Sample sizes match the article's statements: 56 samples (40 tumors, 16 normals), with fatty-acid validation in phosphoAKT1-high/MYC-low n=14 vs. phosphoAKT1-low/MYC-high n=5
- Significance is assessed with the Mann-Whitney Test at p<0.05 — not a parametric test or a different threshold
- Direction of change matches the article: oleic, arachidonic, and DHA significantly higher in MYC-high tumors; creatine increased in phosphoAKT1-high/MYC-low vs. phosphoAKT1-low/MYC-high

## Limitations

- The MYC-high validation subgroup is small (n=5), limiting the power of the Mann-Whitney comparison for the fatty-acid endpoints
- Absolute quantitation covers only selected metabolites (oleic acid, arachidonic acid, DHA, creatine, 2-aminoadipic acid); it validates, but does not replace, the untargeted metabolomics
- Inferences are anchored to phosphoAKT1/MYC immunoblot-based subgroups, which were shown to be independent of Gleason grade and pathologic staging, so results should not be read as grade-dependent biomarkers

## Evidence
