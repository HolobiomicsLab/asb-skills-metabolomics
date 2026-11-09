---
name: plp-pathogenicity-downgrade-fraction-quantification
description: Use when reproducing or auditing how per-tool coding-impact calls drive
  PVS1/ACMG outcomes for ClinVar two-star variants, and you need PVS1 loss/gain counts
  and PLP downgrade fractions per annotation tool.
when_to_use_negative:
- The input annotations have already been restricted to ClinVar-selected/MANE transcripts
  AND manually curated for ACMG classification — the paper notes restriction improves
  consistency but discrepancies persist, so this skill would only re-derive already-audited
  counts.
- The analysis targets non-coding or benign/benign-likely variants, where PVS1 (a
  loss-of-function rule) does not apply and downgrade fractions are undefined.
- The goal is general annotation-concordance measurement (HGVSc/HGVSp agreement rates)
  rather than the downstream ACMG/PVS1 impact quantification.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3325
tools:
- name: ANNOVAR
  role: Source of one set of predicted coding-impact annotations whose LoF calls drive
    the PVS1 assignments (185 loss / 74 gain) and PLP downgrade fraction (55.9%)
- name: SnpEff
  role: Source of a second coding-impact annotation set used to re-derive PVS1 assignments
    (161 loss / 50 gain) and PLP downgrade fraction (66.5%)
- name: Variant Effect Predictor (VEP)
  role: Source of a third coding-impact annotation set used to re-derive PVS1 assignments
    (67 loss / 12 gain) and PLP downgrade fraction (67.3%)
- name: ClinVar
  role: Provides the 164,549 two-star (high review status) variants and the pathogenic/likely-pathogenic
    reference labels used to compute downgrade fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:bbde3aec974274db74bd66c47d8144b5c84397f8535168ae016e5df424fc97af
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
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# plp-pathogenicity-downgrade-fraction-quantification

## Summary

Quantifies how inconsistent coding-impact annotations from ANNOVAR, SnpEff, and VEP propagate through automated PVS1 assignment and ACMG classification, yielding per-tool PVS1 loss/gain counts and the fraction of ClinVar pathogenic/likely-pathogenic (PLP) variants downgraded (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%).

## When to use

Use when reproducing or auditing how per-tool coding-impact calls drive PVS1/ACMG outcomes for ClinVar two-star variants, and you need PVS1 loss/gain counts and PLP downgrade fractions per annotation tool.

## When NOT to use

- The input annotations have already been restricted to ClinVar-selected/MANE transcripts AND manually curated for ACMG classification — the paper notes restriction improves consistency but discrepancies persist, so this skill would only re-derive already-audited counts.
- The analysis targets non-coding or benign/benign-likely variants, where PVS1 (a loss-of-function rule) does not apply and downgrade fractions are undefined.
- The goal is general annotation-concordance measurement (HGVSc/HGVSp agreement rates) rather than the downstream ACMG/PVS1 impact quantification.

## Inputs

- Per-tool coding-impact annotations (ANNOVAR, SnpEff, VEP) for 164,549 two-star ClinVar variants
- ClinVar variant records with star ratings and clinical significance (PLP vs. benign) labels
- ACMG/PVS1 rule specification (PVS1 applied only to predicted loss-of-function coding impacts)

## Outputs

- pvs1_plp_downgrade_summary.csv containing per-tool PVS1 loss/gain counts (ANNOVAR 185/74, SnpEff 161/50, VEP 67/12) and PLP downgrade fractions (55.9%, 66.5%, 67.3%)
- Per-variant table of PVS1-applied vs. incorrect-PVS1 status for PLP ClinVar variants, per tool

## How to apply

Load per-tool coding-impact annotations for the 164,549 two-star ClinVar variants produced by ANNOVAR, SnpEff, and VEP. For each tool, flag variants whose predicted coding impact is loss-of-function, because PVS1 is assigned only from predicted LoF coding impacts under the ACMG rule application described in the paper. Apply PVS1 per tool and tabulate LoF/gain-of-function assignments, expecting the reported counts ANNOVAR 185/74, SnpEff 161/50, VEP 67/12. Then, among ClinVar PLP variants, identify which receive an incorrect PVS1 interpretation and compute the downgrade fraction, comparing against 55.9% (ANNOVAR), 66.5% (SnpEff), and 67.3% (VEP). Write the per-tool PVS1 loss/gain counts and downgrade fractions to pvs1_plp_downgrade_summary.csv. The rationale is that annotation-tool discordance in the Loss-of-Function category (coding-impact concordance only 85.58% across tools) directly alters automated ACMG classification and can omit clinically relevant variants from reports.

## Related tools

- **ANNOVAR** (Source of one set of predicted coding-impact annotations whose LoF calls drive the PVS1 assignments (185 loss / 74 gain) and PLP downgrade fraction (55.9%))
- **SnpEff** (Source of a second coding-impact annotation set used to re-derive PVS1 assignments (161 loss / 50 gain) and PLP downgrade fraction (66.5%))
- **Variant Effect Predictor (VEP)** (Source of a third coding-impact annotation set used to re-derive PVS1 assignments (67 loss / 12 gain) and PLP downgrade fraction (67.3%))
- **ClinVar** (Provides the 164,549 two-star (high review status) variants and the pathogenic/likely-pathogenic reference labels used to compute downgrade fractions)

## Evaluation signals

- Computed PVS1 loss/gain counts must equal the reported values: ANNOVAR 185/74, SnpEff 161/50, VEP 67/12.
- Computed PLP downgrade fractions must match 55.9% (ANNOVAR), 66.5% (SnpEff), and 67.3% (VEP).
- PVS1 must be applied only to variants flagged as loss-of-function by each tool's coding-impact annotation; gain-of-function assignments should total the reported gain counts per tool.
- The variant denominator must be exactly the 164,549 two-star ClinVar variants; any count exceeding this indicates inclusion of lower-review-status records.
- Output file pvs1_plp_downgrade_summary.csv must contain one row per tool with loss, gain, and downgrade-fraction columns populated.

## Limitations

- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones, so per-tool PVS1 outcomes cannot be fully reconciled by transcript filtering alone.
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (Tables S3 and S4), and inconsistencies exist between ClinVar web pages and its VCF, affecting variant matching.
- No software versions are reported for ANNOVAR, SnpEff, or VEP, and results may not be reproducible across tool versions.
- Different MANE selections impact the final variant annotation, and VCF left/right shifts in nucleotide repeats can produce inconsistent HGVS notation.

## Evidence

- [abstract] PVS1 propagation across annotation tools: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)"
- [abstract] LoF category discrepancies drive the effect: "The substantial discrepancies were noted in the Loss-of-Function (LoF) category."
- [discussion] Transcript restriction does not eliminate discrepancies: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
- [discussion] ClinVar data-quality caveats: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
