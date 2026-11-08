---
name: pathogenicity-downgrade-rate-analysis
description: Use when a variant-annotation pipeline (ANNOVAR/SnpEff/VEP on ClinVar
  two-star variants) needs its clinical impact assessed, specifically to quantify
  PLP downgrades caused by incorrect PVS1 interpretations.
when_to_use_negative:
- When the input variants have no ClinVar two-star (or higher) review status — the
  downgrade-rate metric is calibrated on high-review-status ClinVar variants
- When the goal is gene-level or cohort-level pathogenicity burden rather than per-variant
  ACMG reclassification drift from annotation tool differences
- When the variant set is not annotated independently by all three tools (ANNOVAR,
  SnpEff, VEP), since the downgrade-rate comparison requires per-tool outputs
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3168
tools:
- name: ANNOVAR
  role: Annotation tool whose predicted coding impacts and LoF calls feed the ACMG
    reclassification engine; its downgrade rate (55.9%) is a validation target
- name: SnpEff
  role: Annotation tool producing alternative coding-impact calls; shows highest HGVSc
    match (0.988) and a 66.5% downgrade rate
- name: Variant Effect Predictor (VEP)
  role: Annotation tool producing alternative coding-impact calls; shows highest HGVSp
    match (0.977) and a 67.3% downgrade rate
- name: ClinVar
  role: Source of two-star variants, pathogenicity labels, and review-status (star)
    metadata used as the classification ground truth
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:6ad222b80405e77c570df58686ddf7e42706d1a2f61e5dac0d1d9eceed3ab292
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# pathogenicity-downgrade-rate-analysis

## Summary

Quantifies how annotation-tool discrepancies (ANNOVAR, SnpEff, VEP) propagate through ACMG/AMP classification by measuring the fraction of likely-pathogenic/pathogenic ClinVar variants that are downgraded when PVS1 is misinterpreted from inconsistent loss-of-function coding-impact calls.

## When to use

Use when a variant-annotation pipeline (ANNOVAR/SnpEff/VEP on ClinVar two-star variants) needs its clinical impact assessed, specifically to quantify PLP downgrades caused by incorrect PVS1 interpretations.

## When NOT to use

- When the input variants have no ClinVar two-star (or higher) review status — the downgrade-rate metric is calibrated on high-review-status ClinVar variants
- When the goal is gene-level or cohort-level pathogenicity burden rather than per-variant ACMG reclassification drift from annotation tool differences
- When the variant set is not annotated independently by all three tools (ANNOVAR, SnpEff, VEP), since the downgrade-rate comparison requires per-tool outputs

## Inputs

- ClinVar two-star variant list (164,549 variants) with per-tool annotations from ANNOVAR, SnpEff, and VEP (VCF/annotation output)
- Per-variant predicted coding impacts and loss-of-function calls from each annotation tool
- CADD and SpliceAI scores per variant
- ClinVar star rating (review status) per variant
- ClinVar pathogenicity classification (PLP) per variant

## Outputs

- Per-variant ACMG rule assignments and classification-change records (classification_changes.csv)
- Downgrade rate per annotation tool (fraction of PLP variants downgraded, e.g. ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)
- PVS1 agreement/conflict flags between tool-predicted coding impact and ClinVar classification

## How to apply

Load a pre-annotated ClinVar two-star variant set (164,549 variants annotated independently by ANNOVAR, SnpEff, and VEP) with per-tool predicted coding impacts and loss-of-function calls. Assign PVS1 to variants whose coding impact is loss-of-function, then run an ACMG rule engine that applies PP3 with thresholds of CADD > 25.3 or SpliceAI > 0.5 (PP3 disabled whenever PVS1 fires), applies PP5/BP6 with star-based strength (moderate for two-star, very strong for ≥3-star), and excludes PS4, PM3, PP4, BP2, and BP5. Compare each tool's ACMG class to the ClinVar pathogenicity, flagging any variant downgraded from likely-pathogenic/pathogenic due to a PVS1 assignment difference. Report downgrade rates per tool and cross-validate against the published rates (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%).

## Related tools

- **ANNOVAR** (Annotation tool whose predicted coding impacts and LoF calls feed the ACMG reclassification engine; its downgrade rate (55.9%) is a validation target)
- **SnpEff** (Annotation tool producing alternative coding-impact calls; shows highest HGVSc match (0.988) and a 66.5% downgrade rate)
- **Variant Effect Predictor (VEP)** (Annotation tool producing alternative coding-impact calls; shows highest HGVSp match (0.977) and a 67.3% downgrade rate)
- **ClinVar** (Source of two-star variants, pathogenicity labels, and review-status (star) metadata used as the classification ground truth)

## Examples

```
python run_acmg_engine.py --variants clinvar_2star.vcf --tools annovar,snpeff,vep --output classification_changes.csv
```

## Evaluation signals

- Reproduced PLP downgrade rates match published values: ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%
- PP3 is never assigned concurrently with PVS1 (mutual-exclusion invariant from the GDK rule engine)
- PP5/BP6 strength follows the star-based rule: moderate for two-star ClinVar variants, very strong for ≥3-star
- Excluded rules (PS4, PM3, PP4, BP2, BP5) appear nowhere in per-variant rule assignments
- PVS1 assignment correlates with loss-of-function coding-impact predictions from each annotation tool

## Limitations

- Downgrade metrics are only calibrated on the 164,549 two-star ClinVar variants used in the paper; extending to other variant sets requires re-validation
- ClinVar itself contains HGVS syntax that does not adhere to HGVS recommendations (Table S3 and Table S4), which can propagate into the ground truth
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones, so transcript restriction alone does not eliminate downgrade noise
- No software versions were reported for ANNOVAR, SnpEff, or VEP, complicating exact replication of the downgrade rates

## Evidence

- [abstract] Incorrect PVS1 interpretations downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives in reports: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives"
- [other] GDK assigns PVS1 to variants with LoF coding impact: [evidence span withheld — anchor status: not_found]
- [other] Excluded ACMG rules: [evidence span withheld — anchor status: not_found]
- [discussion] Restricting transcripts improved but did not eliminate discrepancies: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
- [discussion] Some HGVS syntax in ClinVar does not adhere to HGVS recommendations: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
