---
name: clinvar-variant-classification
description: Use when the analysis goal is to quantify how annotation tool discrepancies
  in PVS1/PP3 calls translate into ACMG classification changes, requiring implementation
  of the 2015 ACMG rule engine with the paper's thresholds on ClinVar high-review-status
  variants.
when_to_use_negative:
- When the input variant set is not restricted to high-review-status ClinVar two-star
  variants, as the paper's PP5/BP6 star-based strength mapping is calibrated to that
  population
- When the analysis requires rules outside the GDK rule set (e.g. PS4, PM3, PP4, BP2,
  BP5), which are explicitly excluded from this engine
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: ANNOVAR
  role: Annotates variants to generate HGVSc/HGVSp and coding impact calls whose discrepancies
    drive PVS1 classification changes
- name: SnpEff
  role: Provides alternative annotation output to cross-validate coding impact and
    LoF calls feeding the ACMG engine
- name: VEP
  role: Generates annotations used for cross-validation; supports --per_gene and --coding_only
    filters to streamline relevant transcripts
- name: ClinVar
  role: Source of the 164,549 two-star variant set and the review-status star ratings
    used for PP5/BP6 strength
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:4fc0839be4e29e90861549a37f3a352ac0f4e69560cf31da24e7e6e050226f64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - VEP
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# clinvar-variant-classification

## Summary

Reconstructs the GDK platform's ACMG 2015 rule engine (PVS1 from LoF coding impact, PP3 thresholds of CADD>25.3 or SpliceAI>0.5, star-based PP5/BP6 strength) to reproduce the reported PLP downgrade rates (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%) observed when annotation tool discrepancies alter PVS1 calls.

## When to use

Use when the analysis goal is to quantify how annotation tool discrepancies in PVS1/PP3 calls translate into ACMG classification changes, requiring implementation of the 2015 ACMG rule engine with the paper's thresholds on ClinVar high-review-status variants.

## When NOT to use

- When the input variant set is not restricted to high-review-status ClinVar two-star variants, as the paper's PP5/BP6 star-based strength mapping is calibrated to that population
- When the analysis requires rules outside the GDK rule set (e.g. PS4, PM3, PP4, BP2, BP5), which are explicitly excluded from this engine

## Inputs

- 164,549 two-star ClinVar variants annotated by ANNOVAR, SnpEff, and VEP
- Predicted coding impacts and loss-of-function calls per variant
- CADD and SpliceAI scores for PP3 evaluation
- ClinVar star ratings (review status) for PP5/BP6 strength assignment

## Outputs

- Per-variant ACMG rule assignments (PVS1, PP3, PP5, BP6, etc.)
- ACMG classification changes per tool
- classification_changes.csv summarizing PLP downgrade rates

## How to apply

Load the 164,549 two-star ClinVar variants annotated by ANNOVAR, SnpEff, and VEP together with their predicted coding impacts and loss-of-function calls. Implement the ACMG rule engine: assign PVS1 for variants with LoF coding impact; apply PP3 using thresholds of CADD > 25.3 or SpliceAI > 0.5; apply PP5/BP6 strength on a star-based basis (PP5-moderate/BP6-strong for two-star, very strong for ≥3-star); disable PP3 whenever PVS1 is assigned; and exclude PS4, PM3, PP4, BP2, and BP5 from the rule set. Run the engine across the annotated variant set to derive ACMG classification changes, then compare to the reported downgrade rates for PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%). Standardize annotation practices by using identical reference genomes and transcript annotations across all tools, and cross-validate outputs from multiple independent tools to identify discrepancies. Write per-variant rule assignments and the classification-change summary to classification_changes.csv.

## Related tools

- **ANNOVAR** (Annotates variants to generate HGVSc/HGVSp and coding impact calls whose discrepancies drive PVS1 classification changes)
- **SnpEff** (Provides alternative annotation output to cross-validate coding impact and LoF calls feeding the ACMG engine)
- **VEP** (Generates annotations used for cross-validation; supports --per_gene and --coding_only filters to streamline relevant transcripts)
- **ClinVar** (Source of the 164,549 two-star variant set and the review-status star ratings used for PP5/BP6 strength)

## Examples

```
python acmg_engine.py --clinvar two_star_variants.vcf --tool annovar --cadd_threshold 25.3 --spliceai_threshold 0.5 --output classification_changes.csv
```

## Evaluation signals

- Reproduction of the reported PLP downgrade rates: ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3% driven by incorrect PVS1 interpretations
- Per-variant rule assignments written to classification_changes.csv match the expected PP3 threshold behavior (CADD > 25.3 or SpliceAI > 0.5) and PP3 suppression when PVS1 is present
- Excluded rules (PS4, PM3, PP4, BP2, BP5) are absent from all rule assignments
- PP5/BP6 strength mapping matches star ratings: PP5-moderate/BP6-strong for two-star, very strong for ≥3-star

## Limitations

- ClinVar contains HGVS syntax that does not adhere to HGVS recommendations, and inconsistencies exist between ClinVar web pages and its VCF
- Gene inheritance modes provided by CGD, ClinGen, and PanelApp may be inconsistent, affecting interpretation of rules such as PM2 and PM3
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones
- No software versions are reported for the compared annotation tools (ANNOVAR, SnpEff, VEP)

## Evidence

- [methods] PP3 thresholds and PVS1 interplay: [evidence span withheld — anchor status: not_found]
- [abstract] PLP downgrade rates: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)"
- [discussion] Standardization rationale: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
- [abstract] Variant set size: "using 164,549 two-star variants from ClinVar"
- [discussion] Discrepancies persist after transcript restriction: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
