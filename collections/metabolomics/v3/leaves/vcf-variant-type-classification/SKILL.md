---
name: vcf-variant-type-classification
description: Use when you have per-tool HGVSc/HGVSp/coding-impact annotations for
  a normalized ClinVar-style VCF and need concordance rates broken down by variant
  type rather than pooled across all 164,549 two-star variants.
when_to_use_negative:
- The goal is ACMG/PVS1 pathogenicity impact interpretation rather than variant-type-stratified
  annotation concordance (use the PVS1/LoF workflow instead).
- Input is raw unphased caller VCF output where multi-nucleotide variants and repeat
  shifts have not been normalized — phase/normalize first or the type strata are unreliable.
- The analysis targets metabolomics/genomics linkage (NPLinker) data rather than small-variant
  annotation records.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Annotation tool whose HGVSc/HGVSp/coding-impact output is stratified by variant
    type against ClinVar
- name: SnpEff
  role: Annotation tool; concordance stratified by variant type, worst on Inversion
- name: Variant Effect Predictor (VEP)
  role: Annotation tool; concordance stratified by variant type, worst on MS and Duplication
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:88df82d63f06c728ada24bd8baaa24447d02ed3cf6069df12b269b3d914760f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# vcf-variant-type-classification

## Summary

Classify normalized ClinVar VCF variants into type categories (SNV, Ins, Del, Indel, MS, Dup, Inv) and stratify per-tool annotation concordance (ANNOVAR, SnpEff, VEP vs ClinVar) by these categories, because concordance varies strongly by variant type.

## When to use

Use when you have per-tool HGVSc/HGVSp/coding-impact annotations for a normalized ClinVar-style VCF and need concordance rates broken down by variant type rather than pooled across all 164,549 two-star variants.

## When NOT to use

- The goal is ACMG/PVS1 pathogenicity impact interpretation rather than variant-type-stratified annotation concordance (use the PVS1/LoF workflow instead).
- Input is raw unphased caller VCF output where multi-nucleotide variants and repeat shifts have not been normalized — phase/normalize first or the type strata are unreliable.
- The analysis targets metabolomics/genomics linkage (NPLinker) data rather than small-variant annotation records.

## Inputs

- Normalized, phased VCF representation of 164,549 ClinVar two-star variants
- Per-tool annotation records (HGVSc, HGVSp, predicted coding impact) from ANNOVAR, SnpEff, and VEP
- ClinVar reference annotations for the same variants

## Outputs

- variant_type_concordance.csv with columns: tool, variant type, annotation field, n variants, concordance rate
- Per-tool ranking of variant-type categories by concordance

## How to apply

First normalize and phase multi-nucleotide variants in the VCF, since repeat-shifted alleles (left vs right shifts relative to the gene or transcript) from different callers otherwise group inconsistently. Derive each variant's type category (SNV, Ins, Del, Indel, MS, Dup, Inv) from the normalized VCF representation. For each tool (ANNOVAR, SnpEff, VEP) versus the ClinVar reference, use pandas groupby aggregation over the type categories to compute concordance of HGVSc, HGVSp, and predicted coding impact per stratum. Validate against the article's expected ordering: SNV is the most consistent category, Ins/Del/Indel exceed ~0.995 for SnpEff and VEP, MS is the least consistent overall, SnpEff is worst on Inversion, and VEP is worst on MS and Duplication.

## Related tools

- **ANNOVAR** (Annotation tool whose HGVSc/HGVSp/coding-impact output is stratified by variant type against ClinVar)
- **SnpEff** (Annotation tool; concordance stratified by variant type, worst on Inversion)
- **Variant Effect Predictor (VEP)** (Annotation tool; concordance stratified by variant type, worst on MS and Duplication)

## Examples

```
out = df.groupby(['tool','variant_type','field']).apply(lambda d: (d.tool_ann==d.clinvar_ann).mean()).reset_index(name='concordance'); out.to_csv('variant_type_concordance.csv', index=False)
```

## Evaluation signals

- For SnpEff and VEP, Ins, Del, and Indel concordance fractions exceed 0.995.
- SNV is the highest-concordance category per tool and MS is the lowest (or near-lowest) category.
- SnpEff's worst category is Inversion; VEP's worst categories are MS and Duplication — reproduce this ranking pattern.
- Output CSV contains the exact columns tool, variant type, annotation field, n variants, concordance rate, and n sums to 164,549 variants across strata per tool/field.

## Limitations

- VCFs generated by different callers yield inconsistent HGVS notation for nucleotide repeats shifted left or right, so without normalization repeat-shifted alleles are misgrouped.
- Multi-nucleotide variants must be phased and normalized or the variant-type strata are unreliable.
- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations, and ClinVar web pages and its VCF contain inconsistencies, capping achievable concordance.

## Evidence

- [intro] the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation. In cases of nucleotide repeats, shifts to the left or right concerning the gene or transcript: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation. In cases of nucleotide repeats, shifts to the left or right concerning the gene or transcript"
- [discussion] Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF.: "Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF."
- [abstract] using 164,549 two-star variants from ClinVar: "using 164,549 two-star variants from ClinVar"
- [discussion] Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted.: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
