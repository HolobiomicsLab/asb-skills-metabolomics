---
name: loss-of-function-impact-annotation
description: Use when comparing predicted coding-impact calls across annotation tools
  against ClinVar high-review-status variants, or when LoF/PVS1 misassignment is expected
  to drive ACMG classification changes.
when_to_use_negative:
- When only HGVS nomenclature concordance (HGVSc 58.52%, HGVSp 84.04%) is of interest
  rather than coding impact / LoF-driven ACMG effects
- When the variant set is not high-review-status ClinVar data and lacks a trusted
  functional-impact ground truth to compare against
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Annotates ClinVar two-star variants to produce predicted coding impacts and
    LoF calls for concordance comparison and PVS1 assignment
- name: SnpEff
  role: Second annotation engine producing coding-impact categories compared against
    ClinVar (overall agreement 0.992; LoF concordance 0.975)
- name: Variant Effect Predictor (VEP)
  role: Third annotation engine; run with --per_gene and --coding_only to streamline
    relevant transcripts and reduce potential errors
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:5e222572a2e5099c2fd239346a901868759b5aecd413b49f725e148839245780
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - VEP
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# loss-of-function-impact-annotation

## Summary

Annotates ClinVar benchmark variants with ANNOVAR, SnpEff, and VEP to derive predicted coding impacts, with special focus on the Loss-of-Function (LoF) category where tool–ClinVar concordance is lowest, and propagates LoF calls into ACMG PVS1-driven reclassification.

## When to use

Use when comparing predicted coding-impact calls across annotation tools against ClinVar high-review-status variants, or when LoF/PVS1 misassignment is expected to drive ACMG classification changes.

## When NOT to use

- When only HGVS nomenclature concordance (HGVSc 58.52%, HGVSp 84.04%) is of interest rather than coding impact / LoF-driven ACMG effects
- When the variant set is not high-review-status ClinVar data and lacks a trusted functional-impact ground truth to compare against

## Inputs

- ClinVar two-star (high review status) variant set, 164,549 variants
- ANNOVAR, SnpEff, and VEP annotations of the same variants on identical reference genomes and transcript annotations
- Predicted coding-impact terms and loss-of-function calls per tool
- Per-variant pathogenicity classifications (PLP) from the ClinVar benchmark

## Outputs

- Per-tool coding-impact agreement and LoF concordance values in concordance_table.csv
- Per-variant ACMG rule assignments and classification-change summary in classification_changes.csv
- PLP downgrade rates per tool attributable to incorrect PVS1 interpretation (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)

## How to apply

Annotate the same ClinVar two-star variant set (164,549 variants) with ANNOVAR, SnpEff, and VEP using identical reference genomes and transcript annotations across all tools, normalizing each tool's impact terms to a common Sequence Ontology vocabulary and prioritizing severe/high-impact classes when resolving ambiguous mappings. Compare each tool's normalized call against the ClinVar-reported impact per variant, computing overall per-tool agreement (targets: ANNOVAR 0.990, SnpEff 0.992, VEP 0.994) and LoF-restricted concordance over the 31,640 LoF variants (targets: ANNOVAR 0.983, SnpEff 0.975, VEP 0.972). Feed LoF coding-impact calls into an ACMG 2015 rule engine (e.g., the GDK-style engine) that assigns PVS1 for LoF impact, applies PP3 at CADD > 25.3 or SpliceAI > 0.5 with PP3 disabled when PVS1 triggers, applies PP5/BP6 star-based strength, and excludes PS4, PM3, PP4, BP2, and BP5. Rationale: LoF is the category with the largest annotation discrepancies, so incorrect PVS1 interpretations downgrade PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives for clinically relevant variants.

## Related tools

- **ANNOVAR** (Annotates ClinVar two-star variants to produce predicted coding impacts and LoF calls for concordance comparison and PVS1 assignment)
- **SnpEff** (Second annotation engine producing coding-impact categories compared against ClinVar (overall agreement 0.992; LoF concordance 0.975))
- **Variant Effect Predictor (VEP)** (Third annotation engine; run with --per_gene and --coding_only to streamline relevant transcripts and reduce potential errors)

## Examples

```
vep --input_file clinvar_two_star.vcf --per_gene --coding_only --format vcf --vcf --output_file vep_annotated.vcf
```

## Evaluation signals

- Per-tool overall coding-impact agreement matches reported values within rounding tolerance: ANNOVAR 0.990, SnpEff 0.992, VEP 0.994 (three-tool level ~85.58%)
- LoF-restricted concordance matches 0.983 (ANNOVAR), 0.975 (SnpEff), 0.972 (VEP) over the 31,640 LoF variants
- Engine-reproduced PLP downgrade rates driven by incorrect PVS1 interpretations match the paper's 55.9% (ANNOVAR), 66.5% (SnpEff), 67.3% (VEP)
- concordance_table.csv and classification_changes.csv are produced with per-variant rule assignments populated
- PP3 is disabled whenever PVS1 is assigned, and excluded rules (PS4, PM3, PP4, BP2, BP5) never fire

## Limitations

- Restricting transcripts to ClinVar-selected ones improved consistency, but discrepancies with ClinVar persisted even then
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations, and inconsistencies exist between ClinVar web pages and its VCF, contaminating the ground truth
- No software versions were reported for ANNOVAR, SnpEff, or VEP, limiting exact reproducibility

## Evidence

- [abstract] The substantial discrepancies were noted in the Loss-of-Function (LoF) category.: "The substantial discrepancies were noted in the Loss-of-Function (LoF) category."
- [abstract] Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives"
- [discussion] using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors.: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
- [discussion] Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools.: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
