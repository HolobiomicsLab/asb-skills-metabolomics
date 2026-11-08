---
name: acmg-rule-engine-implementation
description: Use when re-deriving ACMG classifications from annotated ClinVar variants
  (two-star review status) to measure how differences between ANNOVAR, SnpEff, and
  VEP coding-impact/LoF calls propagate into PVS1-driven classification changes.
when_to_use_negative:
- When the goal is guideline-compliant clinical ACMG classification including criteria
  that this GDK-derived engine excludes (PS4, PM3, PP4, BP2, BP5)
- When variants lack ClinVar star-rating metadata, since PP5/BP6 strengths are applied
  on a star-based basis
- When you need disease-specific evidence-weighting or inheritance-model rules (PM2/PM3),
  as gene inheritance modes from CGD/ClinGen/PanelApp may themselves be inconsistent
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: ANNOVAR
  role: Produces HGVSc/HGVSp, coding-impact, and LoF annotations that feed the rule
    engine; its PVS1 misassignments drive 55.9% PLP downgrades
- name: SnpEff
  role: Alternate annotator whose coding-impact/LoF calls are run through the same
    engine (66.5% PLP downgrades)
- name: Variant Effect Predictor (VEP)
  role: Third annotator (Ensembl transcripts) run through the engine (67.3% PLP downgrades);
    supports --per_gene and --coding_only to streamline relevant transcripts
- name: ClinVar
  role: Source of the 164,549 two-star variant benchmark set used as engine input
    and validation reference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:fd4a34a5ccb9b9df8341280a8ad194ca6c66305b745db5791c043912f15f948a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# acmg-rule-engine-implementation

## Summary

Implements an in-house ACMG 2015 rule engine (mirroring the GDK platform) that assigns PVS1 from loss-of-function coding impact, applies PP3 at CADD > 25.3 or SpliceAI > 0.5, handles PP5/BP6 strength by ClinVar star rating, and excludes PS4, PM3, PP4, BP2, BP5 to quantify how annotation-tool discrepancies change pathogenicity classifications.

## When to use

Use when re-deriving ACMG classifications from annotated ClinVar variants (two-star review status) to measure how differences between ANNOVAR, SnpEff, and VEP coding-impact/LoF calls propagate into PVS1-driven classification changes.

## When NOT to use

- When the goal is guideline-compliant clinical ACMG classification including criteria that this GDK-derived engine excludes (PS4, PM3, PP4, BP2, BP5)
- When variants lack ClinVar star-rating metadata, since PP5/BP6 strengths are applied on a star-based basis
- When you need disease-specific evidence-weighting or inheritance-model rules (PM2/PM3), as gene inheritance modes from CGD/ClinGen/PanelApp may themselves be inconsistent

## Inputs

- 164,549 two-star review-status ClinVar variants
- Per-tool variant annotations (HGVSc, HGVSp, predicted coding impacts) from ANNOVAR, SnpEff, and VEP
- Loss-of-function (LoF) coding-impact calls per variant per tool
- Variant-level CADD and SpliceAI scores

## Outputs

- Per-variant ACMG rule assignments (PVS1, PP3, PP5, BP6, etc.)
- Per-variant ACMG classification changes across the three annotation tools
- classification_changes.csv with rule assignments and classification-change summary
- PLP downgrade-rate comparison vs. reported rates (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)

## How to apply

Load the 164,549 two-star ClinVar variants annotated by ANNOVAR, SnpEff, and VEP together with their predicted coding impacts and LoF calls. Implement the GDK rule engine: assign PVS1 to variants with LoF coding impact; assign PP3 when CADD > 25.3 or SpliceAI > 0.5, but disable PP3 whenever PVS1 is triggered; treat PP5 as moderate and BP6 as strong for two-star ClinVar variants (very strong for >=3-star); and exclude PS4, PM3, PP4, BP2, and BP5 from the rule set. Run the engine per annotation tool to derive classification changes, then validate by reproducing the paper's PLP downgrade rates caused by incorrect PVS1 interpretations: 55.9% for ANNOVAR, 66.5% for SnpEff, and 67.3% for VEP. Rationale: the paper shows substantial discrepancies in the LoF category across annotators, so the engine isolates how a single rule (PVS1) drives discordant final classifications.

## Related tools

- **ANNOVAR** (Produces HGVSc/HGVSp, coding-impact, and LoF annotations that feed the rule engine; its PVS1 misassignments drive 55.9% PLP downgrades)
- **SnpEff** (Alternate annotator whose coding-impact/LoF calls are run through the same engine (66.5% PLP downgrades))
- **Variant Effect Predictor (VEP)** (Third annotator (Ensembl transcripts) run through the engine (67.3% PLP downgrades); supports --per_gene and --coding_only to streamline relevant transcripts)
- **ClinVar** (Source of the 164,549 two-star variant benchmark set used as engine input and validation reference)

## Evaluation signals

- Reproduced PLP downgrade rates match reported values: ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%, driven by incorrect PVS1 interpretations
- PP3 is never co-assigned with PVS1 on the same variant (engine disables PP3 when PVS1 triggers)
- PP3 is only assigned when CADD > 25.3 or SpliceAI > 0.5; spot-check threshold boundary variants
- No variant receives any of the excluded rules (PS4, PM3, PP4, BP2, BP5)
- PP5/BP6 strengths in output match ClinVar star ratings (moderate/strong for two-star, very strong for >=3-star)

## Limitations

- The engine reproduces the GDK platform's deliberately reduced rule set (excludes PS4, PM3, PP4, BP2, BP5), so it is not a full ACMG 2015 implementation
- Classification changes are driven primarily by incorrect PVS1 calls from LoF-category annotation discrepancies; concordance is only 85.58% for coding impact across tools
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones, and some HGVS syntax in ClinVar does not adhere to HGVS recommendations
- No software versions are reported for the compared annotation tools, limiting exact reproducibility

## Evidence

- [abstract] PVS1-driven PLP downgrades: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)"
- [abstract] Benchmark dataset scale: "using 164,549 two-star variants from ClinVar"
- [abstract] Annotation concordance rates: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [discussion] Cross-validation rationale: "Implement cross-validation by comparing outputs from multiple independent tools for the same variants to identify discrepancies"
- [discussion] Inheritance-rule caveat: "gene inheritance modes provided by CGD30 ClinGen31, and PanelApp32 may also be inconsistent, impact the interpretation of PM2 and PM3 rules"
