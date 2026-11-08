---
name: annotation-mismatch-categorization
description: Use when comparing HGVSc/HGVSp or coding-impact outputs of ANNOVAR, SnpEff,
  or VEP against ClinVar two-star variants and you need to break down why each mismatch
  occurred rather than just count match rates.
when_to_use_negative:
- When the goal is only a single overall match rate per tool without causal breakdown
  of why mismatches occur.
- When the comparison is against a reference other than ClinVar two-star variants
  and no reference HGVS strings exist to compare against.
- When the inputs are coding-impact or PVS1 ACMG classifications rather than HGVSc/HGVSp
  nomenclature — the category taxonomy is defined for HGVS strings.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0625
tools:
- name: ANNOVAR
  role: Source of HGVSc/HGVSp annotations to be compared with ClinVar and categorized
    by mismatch cause
- name: SnpEff
  role: Source of HGVSc/HGVSp annotations to be compared with ClinVar and categorized
    by mismatch cause
- name: Variant Effect Predictor (VEP)
  role: Source of HGVSc/HGVSp annotations to be compared with ClinVar and categorized
    by mismatch cause
- name: ClinVar
  role: Reference database whose two-star variants provide the gold-standard HGVSc/HGVSp
    strings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:7a3f2021861fd387d59d57d4f9de3cd3e087834af8aba4dbbae2f452ec976872
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# annotation-mismatch-categorization

## Summary

Classifies disagreements between annotation-tool HGVS (HGVSc/HGVSp) and ClinVar reference strings into three causal categories — missing (NA), position/variant-type differences, and non-preferred/unknown syntax — and reports per-tool, per-field percentages so the reported concordance (e.g. ANNOVAR HGVSc 79.19% NA, SnpEff 78.00% position/type, VEP HGVSc 95.47% position/type) can be reproduced.

## When to use

Use when comparing HGVSc/HGVSp or coding-impact outputs of ANNOVAR, SnpEff, or VEP against ClinVar two-star variants and you need to break down why each mismatch occurred rather than just count match rates.

## When NOT to use

- When the goal is only a single overall match rate per tool without causal breakdown of why mismatches occur.
- When the comparison is against a reference other than ClinVar two-star variants and no reference HGVS strings exist to compare against.
- When the inputs are coding-impact or PVS1 ACMG classifications rather than HGVSc/HGVSp nomenclature — the category taxonomy is defined for HGVS strings.

## Inputs

- ClinVar reference HGVS strings (HGVSc and HGVSp) for 164,549 two-star ClinVar variants
- Tool-generated HGVSc annotations from ANNOVAR, SnpEff, and VEP for the same variant set
- Tool-generated HGVSp annotations from ANNOVAR, SnpEff, and VEP for the same variant set

## Outputs

- Per-tool, per-field, per-category mismatch percentage table (mismatch_categories.csv)
- Class labels (match / NA / position-or-variant-type difference / non-preferred-unknown syntax) for each variant-tool-field comparison

## How to apply

Load the ClinVar reference HGVSc and HGVSp strings for the 164,549 two-star ClinVar variants together with tool-generated annotations from ANNOVAR, SnpEff, and VEP. For each tool and each HGVS field, compare tool output against the ClinVar reference and label each record as match or mismatch. Assign every mismatch to one of three classes: missing (NA, tool produced no value), position/variant-type differences (same variant annotated at a shifted or differently typed position), or non-preferred/unknown HGVS syntax (ClinVar or tool syntax not adhering to HGVS recommendations). Compute the percentage of each mismatch category per tool per field so that category percentages plus the match rate sum to 100%. Sanity-check the reproduced match rates against the reported concordance (HGVSc 58.52%, HGVSp 84.04%) and confirm the per-tool category percentages reproduce the reported values.

## Related tools

- **ANNOVAR** (Source of HGVSc/HGVSp annotations to be compared with ClinVar and categorized by mismatch cause)
- **SnpEff** (Source of HGVSc/HGVSp annotations to be compared with ClinVar and categorized by mismatch cause)
- **Variant Effect Predictor (VEP)** (Source of HGVSc/HGVSp annotations to be compared with ClinVar and categorized by mismatch cause)
- **ClinVar** (Reference database whose two-star variants provide the gold-standard HGVSc/HGVSp strings)

## Examples

```
python categorize_mismatches.py --clinvar clinvar_2star_hgvs.tsv --tool_output annovar_hgvs.csv --field HGVSc --out mismatch_categories.csv
```

## Evaluation signals

- Reproduced HGVSc match rate is consistent with the reported concordance of 58.52%.
- Reproduced HGVSp match rate is consistent with the reported concordance of 84.04%.
- ANNOVAR HGVSc mismatch-category percentages reproduce 79.19% missing (NA), 19.67% position/variant-type, and 1.14% unknown.
- SnpEff HGVSc mismatch-category percentages reproduce 78.00% position/variant-type.
- VEP HGVSc mismatch-category percentages reproduce 95.47% position/variant-type.

## Limitations

- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations (Table S3 and Table S4), so mismatches may reflect reference-side problems rather than tool errors.
- Inconsistencies exist between ClinVar web pages and its VCF, which can propagate into the reference strings used for comparison.
- No software versions are reported for the compared annotation tools, limiting reproducibility of the exact mismatch percentages.
- Restricting transcripts to those selected by ClinVar improves consistency in HGVSc, HGVSp, and coding impact, but discrepancies with ClinVar persisted.

## Evidence

- [methods] mismatch category distribution: [evidence span withheld — anchor status: not_found]
- [abstract] reported concordance rates: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [abstract] per-tool best match rates: "SnpEff showed the highest match for HGVSc (0.988), while VEP bettered for HGVSp (0.977)."
- [discussion] ClinVar syntax non-compliance: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [discussion] transcript restriction caveat: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
