---
name: metabolite-pathway-annotation
description: Use when a metabolomics hit list needs biochemical pathway labels or
  pathway-level enrichment direction (e.g., KEGG sets, NES from single-sample GSEA)
  to interpret oncoprotein-driven metabolic phenotypes.
when_to_use_negative:
- Input is untargeted MS output with only spectral features and no metabolite identifiers
  — pathway sets like KEGG cannot be applied until metabolites are identified
- The goal is absolute concentration measurement itself (e.g., µg/g tissue quantitation
  with internal standards) rather than pathway-level annotation and enrichment interpretation
- The biological material lies outside the pathway set coverage (e.g., metabolites
  with no KEGG annotation), so enrichment scores would be undefined
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3389
tools:
- name: Python
  role: Scripted assembly of the annotated metabolite table and KEGG set/GSEA scoring
    pipeline
- name: GraphPad Prism
  role: Statistical testing (Mann-Whitney, p<0.05) of the annotated metabolite rows
    during validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:f61a5d6da89d65b8be5b0d079883d5b324e79afc34a0772b4f6371870f35e292
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3389
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-pathway-annotation

## Summary

Annotates mass-spectrometry-detected metabolites to biochemical pathways (e.g., KEGG metabolite sets) and quantifies pathway-level enrichment direction so that metabolite classes (aerobic glycolysis intermediates, SCD1-derived fatty acids, lysine-metabolism intermediates, creatine) can be linked to oncoprotein phenotypes such as activated AKT1 or MYC overexpression.

## When to use

Use when a metabolomics hit list needs biochemical pathway labels or pathway-level enrichment direction (e.g., KEGG sets, NES from single-sample GSEA) to interpret oncoprotein-driven metabolic phenotypes.

## When NOT to use

- Input is untargeted MS output with only spectral features and no metabolite identifiers — pathway sets like KEGG cannot be applied until metabolites are identified
- The goal is absolute concentration measurement itself (e.g., µg/g tissue quantitation with internal standards) rather than pathway-level annotation and enrichment interpretation
- The biological material lies outside the pathway set coverage (e.g., metabolites with no KEGG annotation), so enrichment scores would be undefined

## Inputs

- Metabolite abundance matrix from mass spectrometry-based metabolomics with metabolite identifiers (AKT1-/MYC-transformed RWPE-1 cells, transgenic mice, human tumors/normal samples)
- KEGG metabolite set annotations (Dataset S1, >50 metabolite sets)
- Sample subgroup labels from k-means clustering of phosphoAKT1/MYC densitometric values (e.g., phosphoAKT1-high/MYC-low n=21, phosphoAKT1-low/MYC-high n=9)

## Outputs

- Metabolite-to-pathway annotation table (metabolite name, pathway annotation, group-comparison direction, sample sizes, units, statistical test, significance threshold)
- Per-pathway normalized enrichment scores (NES) indicating direction and extent of enrichment for each pathway in each of the 3 datasets
- Validated subset of differentially accumulated metabolites selected for absolute quantitation

## How to apply

Map the metabolites detected by mass spectrometry-based metabolomics onto pathway sets — here, over 50 metabolite sets from KEGG annotation (Dataset S1) — and score each set per sample with single-sample GSEA. Use gene set-size-normalized enrichment scores (NES) to determine the extent and direction of enrichment for each pathway across the three datasets (engineered RWPE-1 cells, transgenic mouse prostates, human prostate specimens), then interpret direction against the molecular phenotype: AKT1 activation enriched aerobic glycolysis metabolites, MYC overexpression enriched dysregulated lipid metabolism. When building validation tables for selected metabolites, assign each an explicit pathway annotation (e.g., SCD1-derived fatty acid for oleic/arachidonic/DHA acids, lysine intermediate for 2-aminoadipic acid, creatine) and pair it with the group comparison direction, sample sizes, units (µg/g tissue), test (Mann-Whitney), and significance threshold (p<0.05). Ensure every annotation is traceable to a verbatim claim and consistent with the stated AKT1-glycolysis and MYC-lipid findings.

## Related tools

- **Python** (Scripted assembly of the annotated metabolite table and KEGG set/GSEA scoring pipeline)
- **GraphPad Prism** (Statistical testing (Mann-Whitney, p<0.05) of the annotated metabolite rows during validation)

## Evaluation signals

- Every metabolite row carries a pathway annotation (e.g., SCD1-derived fatty acid, lysine intermediate, creatine) traceable to a verbatim article claim
- NES direction is consistent with the stated findings: aerobic glycolysis metabolites enriched with AKT1 activation; dysregulated lipid metabolism with MYC overexpression
- Enrichment direction is reported for all 3 datasets (RWPE-1 cells, transgenic mice, human tumors), not just one
- Validation rows satisfy the stated significance convention: Mann-Whitney test with p<0.05, concentrations in µg/g tissue, correct subgroup n (oleic/arachidonic/DHA: phosphoAKT1-high/MYC-low n=14 vs phosphoAKT1-low/MYC-high n=5)
- Annotation-based subgroup contrasts reproduce the reported differential segregation of phosphoAKT1-low/MYC-high vs phosphoAKT1-high/MYC-low tumors (Fisher test, p<0.01)

## Limitations

- Pathway annotation depends on KEGG set coverage; metabolite sets outside the >50 KEGG-annotated sets (Dataset S1) cannot be assigned a pathway class
- Small validation subgroup sizes (e.g., phosphoAKT1-low/MYC-high n=5 for oleic/arachidonic/DHA) limit the statistical power of the annotated group-comparison directions
- AKT1/MYC status is independent of Gleason grade and pathologic staging, so pathway annotations should not be interpreted as grade- or stage-associated markers

## Evidence
