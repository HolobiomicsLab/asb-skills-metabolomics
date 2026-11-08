---
name: hgvs-nomenclature-syntax-validation
description: Use when tool-generated HGVS strings disagree with a trusted reference
  (e.g. ClinVar two-star variants) and you must attribute each mismatch to missing
  (NA) values, shifted position or variant type, or non-preferred/unknown HGVS syntax.
when_to_use_negative:
- When the goal is concordance of predicted coding impact or Loss-of-Function (LoF)
  categories rather than HGVS nomenclature — this skill classifies nomenclature mismatches
  only
- When there is no trusted reference HGVS set (e.g. variants absent from ClinVar),
  since the classification is entirely relative to the reference strings
- When the discrepancy has already been resolved to a single root cause (e.g. purely
  missing annotations) — a full three-way breakdown adds no value
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ANNOVAR
  role: Source of HGVSc/HGVSp annotations to be compared against ClinVar reference
    strings
- name: SnpEff
  role: Source of HGVSc/HGVSp annotations to be compared against ClinVar reference
    strings
- name: Variant Effect Predictor (VEP)
  role: Source of HGVSc/HGVSp annotations to be compared against ClinVar reference
    strings
- name: ClinVar
  role: Reference HGVS nomenclature source (two-star high review-status variants)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:e0e57523a8f7adbd93c5e4419772d491473491808ef054b62bef5ef1d8f24ba3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hgvs-nomenclature-syntax-validation

## Summary

A discrepancy-mode breakdown skill that classifies HGVSc/HGVSp mismatches between annotation tools (ANNOVAR, SnpEff, VEP) and a reference database (ClinVar) into three cause categories: missing (NA) annotations, position/variant-type differences, and non-preferred/unknown HGVS syntax. It quantifies why nomenclature concordance fails, not just whether it fails.

## When to use

Use when tool-generated HGVS strings disagree with a trusted reference (e.g. ClinVar two-star variants) and you must attribute each mismatch to missing (NA) values, shifted position or variant type, or non-preferred/unknown HGVS syntax.

## When NOT to use

- When the goal is concordance of predicted coding impact or Loss-of-Function (LoF) categories rather than HGVS nomenclature — this skill classifies nomenclature mismatches only
- When there is no trusted reference HGVS set (e.g. variants absent from ClinVar), since the classification is entirely relative to the reference strings
- When the discrepancy has already been resolved to a single root cause (e.g. purely missing annotations) — a full three-way breakdown adds no value

## Inputs

- ClinVar reference HGVS strings (HGVSc and HGVSp) for high review-status (two-star) variants
- Tool-generated HGVSc and HGVSp annotations from ANNOVAR
- Tool-generated HGVSc and HGVSp annotations from SnpEff
- Tool-generated HGVSc and HGVSp annotations from VEP

## Outputs

- mismatch_categories.csv: per-tool, per-field, per-category mismatch percentage table (missing/NA, position/variant-type, non-preferred/unknown)
- Per-tool match rates for HGVSc and HGVSp consistent with overall concordance (HGVSc 58.52%, HGVSp 84.04%)

## How to apply

Load the reference HGVSc/HGVSp strings from ClinVar (164,549 two-star variants in the source study) alongside tool-generated HGVSc/HGVSp from ANNOVAR, SnpEff, and VEP, then compare each record per tool per field to label match vs mismatch. Assign each mismatch to one of three classes: missing (tool produced no value), position/variant-type difference (same variant annotated at a shifted or differently typed position), or non-preferred/unknown syntax (ClinVar or tool syntax not adhering to HGVS recommendations). Compute the percentage of each mismatch category per tool per field (e.g. ANNOVAR HGVSc: 79.19% NA; SnpEff HGVSc: 78.00% position/type; VEP HGVSc: 95.47% position/type) so that category percentages sum with the match rate to 100%. Write the per-tool, per-field, per-category table to mismatch_categories.csv. The rationale is that overall concordance rates (58.52% HGVSc, 84.04% HGVSp) hide distinct failure modes that require different remediation: NA gaps are tool coverage problems, position/type shifts stem from VCF normalization and repeat shifting, and syntax issues reflect non-adherence to HGVS recommendations.

## Related tools

- **ANNOVAR** (Source of HGVSc/HGVSp annotations to be compared against ClinVar reference strings)
- **SnpEff** (Source of HGVSc/HGVSp annotations to be compared against ClinVar reference strings)
- **Variant Effect Predictor (VEP)** (Source of HGVSc/HGVSp annotations to be compared against ClinVar reference strings)
- **ClinVar** (Reference HGVS nomenclature source (two-star high review-status variants))

## Examples

```
df.groupby(['tool','field','category']).size().div(len(df)).mul(100).to_csv('mismatch_categories.csv')
```

## Evaluation signals

- Reproduced overall match rates are consistent with the reported concordance: 58.52% for HGVSc and 84.04% for HGVSp
- Per-tool mismatch-category percentages reproduce reported values: ANNOVAR HGVSc 79.19% NA, SnpEff HGVSc 78.00% position/type, VEP HGVSc 95.47% position/type
- For each tool and field, the sum of the three mismatch-category percentages plus the match rate equals 100%
- mismatch_categories.csv exists and contains one row per tool x field x category combination
- Every mismatch record is assigned exactly one of the three classes (missing/NA, position/variant-type, non-preferred/unknown) with no unclassified residuals

## Limitations

- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations (Table S3 and Table S4), so 'unknown syntax' mismatches may reflect reference-side problems
- VCFs generated by different callers can produce inconsistent HGVS notation for nucleotide repeats due to left/right shifts relative to the gene or transcript, confounding position/type classification
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones, so transcript restriction alone does not eliminate mismatches
- No software versions for ANNOVAR, SnpEff, or VEP were reported, limiting reproducibility of exact category percentages

## Evidence

- [abstract] concordance rates anchor validation: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [discussion] non-adherent syntax in reference: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [intro] caller-induced HGVS shifts: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation. In cases of nucleotide repeats, shifts to the left or right"
- [discussion] transcript restriction insufficient: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
