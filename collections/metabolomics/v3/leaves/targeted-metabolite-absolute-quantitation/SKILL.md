---
name: targeted-metabolite-absolute-quantitation
description: Use when untargeted metabolomic profiling has flagged differentially
  accumulated metabolites and directional group differences must be confirmed with
  absolute-quantitation assays rather than relative peak intensities.
when_to_use_negative:
- The goal is global untargeted metabolomic profiling rather than confirmation of
  a pre-selected small metabolite panel
- Group distributions are large, parametric, and matched — a t-test/ANOVA framework
  would be more appropriate than the Mann-Whitney test with the small n=14 vs n=5
  subgroup counts
- Only relative fold-changes are needed; absolute calibration with internal standards
  in µg/g tissue adds unnecessary assay burden
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GraphPad Prism
  role: Statistical testing of group differences between phosphoAKT1/MYC tumor subgroups
    using the Mann-Whitney test at p<0.05
- name: Python
  role: Assembling the five-metabolite validation table (metabolite, pathway annotation,
    direction, sample sizes, µg/g units, test, threshold) and checking traceability
    of each row to verbatim article claims
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:d5c26063731808c6a6786cf8edefa33fa7821bb9438fab72c7418c7c743b994b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# targeted-metabolite-absolute-quantitation

## Summary

Validates untargeted mass spectrometry-based metabolomics by measuring absolute concentrations of selected differentially accumulated metabolites (e.g. oleic, arachidonic, and docosahexaenoic acids) with specific internal standards, reporting µg/g tissue and testing group differences with the Mann-Whitney test at p<0.05.

## When to use

Use when untargeted metabolomic profiling has flagged differentially accumulated metabolites and directional group differences must be confirmed with absolute-quantitation assays rather than relative peak intensities.

## When NOT to use

- The goal is global untargeted metabolomic profiling rather than confirmation of a pre-selected small metabolite panel
- Group distributions are large, parametric, and matched — a t-test/ANOVA framework would be more appropriate than the Mann-Whitney test with the small n=14 vs n=5 subgroup counts
- Only relative fold-changes are needed; absolute calibration with internal standards in µg/g tissue adds unnecessary assay burden

## Inputs

- Untargeted MS-based metabolomics feature list identifying differentially accumulated metabolites (e.g. SCD1-derived fatty acids, creatine, 2-aminoadipic acid)
- Tissue extracts or cell/mouse/human prostate specimens for the selected metabolite panel
- Specific internal standards for each targeted metabolite
- Sample metadata with subgroup assignments (phosphoAKT1-high/MYC-low n=14, phosphoAKT1-low/MYC-high n=5)

## Outputs

- Absolute concentrations per metabolite in µg/g tissue
- Structured five-metabolite validation table: metabolite name, pathway annotation, group-comparison direction, sample sizes, units (µg/g), test (Mann-Whitney), significance threshold (p<0.05)

## How to apply

Parse the Absolute Quantitation Of Metabolites methods subsection to extract the assay panel (oleic, arachidonic, and docosahexaenoic acids, creatine, and the lysine-metabolism intermediate 2-aminoadipic acid), the µg/g tissue unit convention, the use of specific internal standards, the Mann-Whitney test, and the p<0.05 significance threshold. Extract the sample-size statement (56 samples: 40 tumors, 16 normals) and tumor subgroup validation counts (fatty acids validated in phosphoAKT1-high/MYC-low n=14 vs phosphoAKT1-low/MYC-high n=5). For each selected metabolite, run the targeted assay with its internal standard, compute absolute concentration in µg/g tissue, and apply the Mann-Whitney test between the biologically contrasting subgroups to determine the direction of change (e.g. fatty acids higher in MYC-high tumors, creatine higher in phosphoAKT1-high/MYC-low tumors). Assemble the metabolites into a structured table with columns for metabolite name, biochemical pathway annotation (SCD1-derived fatty acid vs lysine intermediate vs creatine), group-comparison direction, sample sizes, units, test, and significance threshold.

## Related tools

- **GraphPad Prism** (Statistical testing of group differences between phosphoAKT1/MYC tumor subgroups using the Mann-Whitney test at p<0.05)
- **Python** (Assembling the five-metabolite validation table (metabolite, pathway annotation, direction, sample sizes, µg/g units, test, threshold) and checking traceability of each row to verbatim article claims)

## Evaluation signals

- Every metabolite row is traceable to a verbatim claim in the article text (oleic, arachidonic, DHA, creatine, 2-aminoadipic acid)
- Concentrations are reported in µg/g tissue, matching the methods' unit convention, not arbitrary peak areas
- Direction summary is consistent with the MYC-overexpression/dysregulated-lipid-metabolism and AKT1-glycolysis findings (fatty acids higher in MYC-high; creatine higher in phosphoAKT1-high/MYC-low)
- Reported subgroup validation counts match the stated n=14 (phosphoAKT1-high/MYC-low) vs n=5 (phosphoAKT1-low/MYC-high)
- Group comparisons use the Mann-Whitney test with the stated p<0.05 significance threshold

## Limitations

- Validation subgroup for phosphoAKT1-low/MYC-high tumors is small (n=5), limiting statistical power for the fatty-acid comparisons
- Only a small selected panel (five metabolites) is absolutely quantified, not the full untargeted metabolome; conclusions are metabolite-specific
- Absolute quantitation validates untargeted metabolomics directionally; it does not by itself establish pathway causality for MYC-driven lipid dysregulation

## Evidence

- [methods] the assay panel (oleic, arachidonic, docosahexaenoic acids, creatine, 2-aminoadipic acid), the µg/g tissue unit convention, the use of specific internal standards: [evidence span withheld — anchor status: not_found]
- [results] All three validated fatty acids (oleic, arachidonic, and docosahexaenoic acids) were present at significantly higher concentrations in MYC-high tumors: [evidence span withheld — anchor status: not_found]
