---
name: clinvar-high-review-status-variant-filtering
description: Use when building a benchmark set from ClinVar for comparing variant-annotation
  outputs or ACMG rule application and you need only variants with a high (two-star)
  review status.
when_to_use_negative:
- When the analysis goal is to survey ClinVar records with low or conflicting review
  status (e.g. one-star or 'no assertion provided' records) — this skill deliberately
  excludes them.
- When you need a clinically curated single-submitter variant set where expert panel
  (three-star) review exists — the two-star filter is a different, broader confidence
  tier.
- When the input is already a pre-filtered benchmark VCF (e.g. an in-house high-confidence
  set) and re-filtering would discard records that lack the two-star string but were
  independently validated.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3577
tools:
- name: ClinVar
  role: Source database whose review-status field defines the two-star (high review
    status) filter used to build the 164,549-variant benchmark
- name: ANNOVAR
  role: Downstream annotation tool applied to the filtered two-star variant set for
    HGVSc/HGVSp/coding-impact comparison
- name: SnpEff
  role: Downstream annotation tool applied to the filtered two-star variant set for
    HGVSc/HGVSp/coding-impact comparison
- name: Variant Effect Predictor (VEP)
  role: Downstream annotation tool applied to the filtered two-star variant set; supports
    --per_gene and --coding_only flags to streamline relevant transcripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:06dcaf603ebbd1f4bcc20b87b3e0673a934e8d10551a7c12424d4140cb162c53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clinvar-high-review-status-variant-filtering

## Summary

Filter the ClinVar variant set to 164,549 two-star (high review status) variants so that cross-tool annotation concordance (ANNOVAR, SnpEff, VEP) and downstream ACMG/PVS1 impact are evaluated on a ClinVar high-confidence benchmark rather than the full, uncurated ClinVar release.

## When to use

Use when building a benchmark set from ClinVar for comparing variant-annotation outputs or ACMG rule application and you need only variants with a high (two-star) review status.

## When NOT to use

- When the analysis goal is to survey ClinVar records with low or conflicting review status (e.g. one-star or 'no assertion provided' records) — this skill deliberately excludes them.
- When you need a clinically curated single-submitter variant set where expert panel (three-star) review exists — the two-star filter is a different, broader confidence tier.
- When the input is already a pre-filtered benchmark VCF (e.g. an in-house high-confidence set) and re-filtering would discard records that lack the two-star string but were independently validated.

## Inputs

- ClinVar VCF release (variant_005_summary.txt / clinvar.vcf.gz) with per-variant review-status field
- Reference genome build (GRCh37/GRCh38) matching the ClinVar release for downstream annotation

## Outputs

- Filtered two-star variant VCF (164,549 high review status ClinVar variants)
- Audit table mapping each retained variant to its ClinVar review-status string

## How to apply

Download the ClinVar VCF release and identify variants whose ClinVar review-status field carries the two-star designation ('criteria provided, multiple submitters, no conflicts'), which ClinVar uses to flag high-confidence records; this yields the 164,549-variant benchmark the paper used. Restrict the input VCF to these two-star records before running any of the three annotation tools (ANNOVAR, SnpEff, VEP) so that every tool annotates exactly the same variant list on identical reference genome and transcript annotations (MANE where possible). Because the review-status filter operates on ClinVar metadata rather than on variant consequence, apply it as a pre-annotation subsetting step, not as a post-annotation filter. Keep the filtered variant identifiers and the review-status strings in an audit file so that the downstream PVS1/ACMG concordance analysis can be traced back to ClinVar submissions. Note that ClinVar web pages and its VCF can be inconsistent and some HGVS syntax in ClinVar does not adhere to HGVS recommendations, so treat the VCF review status as the authoritative filter key.

## Related tools

- **ClinVar** (Source database whose review-status field defines the two-star (high review status) filter used to build the 164,549-variant benchmark)
- **ANNOVAR** (Downstream annotation tool applied to the filtered two-star variant set for HGVSc/HGVSp/coding-impact comparison)
- **SnpEff** (Downstream annotation tool applied to the filtered two-star variant set for HGVSc/HGVSp/coding-impact comparison)
- **Variant Effect Predictor (VEP)** (Downstream annotation tool applied to the filtered two-star variant set; supports --per_gene and --coding_only flags to streamline relevant transcripts)

## Examples

```
bcftools view -i 'INFO/CLNREVSTAT[*] ~ "criteria provided, multiple submitters, no conflicts"' clinvar.vcf.gz -Oz -o clinvar_two_star.vcf.gz && bcftools index clinvar_two_star.vcf.gz
```

## Evaluation signals

- Retained variant count equals 164,549 when filtering the paper's ClinVar release; any deviation indicates a ClinVar version mismatch or an incorrect review-status key.
- Every retained record's review-status string contains 'criteria provided, multiple submitters, no conflicts' (two-star tier).
- No variant in the filtered set has a ClinVar review status of one-star, zero-star, or 'no assertion provided'.
- Running ANNOVAR, SnpEff, and VEP on the filtered set produces non-empty annotation output for the same variant identifiers across all three tools, confirming the subset is complete and well-formed.
- The filter is applied on ClinVar review status prior to annotation, not on predicted coding impact or ACMG class, so downstream PVS1 loss/gain counts (ANNOVAR 185/74, SnpEff 161/50, VEP 67/12) remain reproducible on the retained set.

## Limitations

- ClinVar web pages and its VCF can be inconsistent, so the review-status filter applied to the VCF may not perfectly mirror what the ClinVar web interface reports for the same variant.
- Some HGVS syntax present in ClinVar does not adhere to HGVS recommendations (Tables S3 and S4), so variants passing the review-status filter may still carry malformed nomenclature that complicates cross-tool matching.
- The two-star filter is tied to a specific ClinVar release; different releases will yield different two-star variant counts, and no software versions of the annotation tools themselves are reported in the paper for reproducibility.

## Evidence

- [abstract] Sample size of high-review-status ClinVar benchmark: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [intro] Definition of the ClinVar dataset used: "comparing the concordance of variant nomenclature and predicted coding impacts generated by ANNOVAR, SnpEff, and VEP for 164,549 high review status ClinVar variants"
- [discussion] ClinVar web page vs VCF inconsistency caveat: "inconsistencies were between ClinVar web pages and its VCF"
- [discussion] ClinVar HGVS non-compliance caveat: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [intro] Acknowledged gap in benchmark quality this skill addresses: "However, comprehensive evaluations of consistency with reputable clinical databases are lacking, and the impact on ACMG interpretation needs to be explored."
- [discussion] Recommendation to anchor cross-validation on a ClinVar high-confidence set: "Use an in-house benchmark or ClinVar high-confidence set."
