---
name: per-tool-percentage-tabulation
description: Use when comparing tool-generated HGVS annotations (HGVSc/HGVSp) against
  a reference standard such as ClinVar and you need to quantify, per tool and per
  field, what fraction of mismatches are missing values, position/variant-type shifts,
  or non-preferred syntax.
when_to_use_negative:
- The analysis target is predicted coding impact or ACMG/PVS1 reclassification (e.g.
  LoF category discrepancies) rather than HGVS nomenclature differences — use a coding-impact
  concordance workflow instead
- There is no high-quality reference annotation (e.g. ClinVar two-star variants) to
  classify mismatches against; category percentages would be uninterpretable
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ANNOVAR
  role: Source of tool-generated HGVSc/HGVSp annotations to compare against ClinVar
    and categorize mismatch causes for
- name: SnpEff
  role: Source of tool-generated HGVSc/HGVSp annotations to compare against ClinVar
    and categorize mismatch causes for
- name: Variant Effect Predictor (VEP)
  role: Source of tool-generated HGVSc/HGVSp annotations to compare against ClinVar
    and categorize mismatch causes for; can be run with --per_gene and --coding_only
    to streamline transcripts
- name: ClinVar
  role: Reference standard HGVS nomenclature used to classify each tool annotation
    as match or mismatch
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:190b027c01267355d9883a909a2756be4810c81f114eab2162b2ceb0e5c056fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# per-tool-percentage-tabulation

## Summary

Tabulates the percentage of HGVSc/HGVSp annotation mismatches between each variant annotation tool (ANNOVAR, SnpEff, VEP) and ClinVar, categorized as missing (NA), position/variant-type differences, or non-preferred/unknown HGVS syntax, so the causes of annotation discordance can be quantified per tool and per field.

## When to use

Use when comparing tool-generated HGVS annotations (HGVSc/HGVSp) against a reference standard such as ClinVar and you need to quantify, per tool and per field, what fraction of mismatches are missing values, position/variant-type shifts, or non-preferred syntax.

## When NOT to use

- The analysis target is predicted coding impact or ACMG/PVS1 reclassification (e.g. LoF category discrepancies) rather than HGVS nomenclature differences — use a coding-impact concordance workflow instead
- There is no high-quality reference annotation (e.g. ClinVar two-star variants) to classify mismatches against; category percentages would be uninterpretable

## Inputs

- ClinVar reference HGVS strings (HGVSc and HGVSp) for 164,549 two-star (high review status) ClinVar variants
- Tool-generated HGVSc/HGVSp annotations from ANNOVAR, SnpEff, and VEP for the same variant set

## Outputs

- mismatch_categories.csv — per-tool, per-field (HGVSc/HGVSp), per-category (missing/NA, position-variant-type, non-preferred/unknown) percentage table
- Per-tool match rates for HGVSc and HGVSp

## How to apply

Load the ClinVar reference HGVS strings (HGVSc and HGVSp) for the 164,549 two-star ClinVar variants together with the tool-generated HGVSc/HGVSp annotations from ANNOVAR, SnpEff, and VEP, and classify each record per tool per field as match or mismatch. Assign each mismatch to one of three classes: missing (NA, the tool produced no value), position/variant-type differences (the same variant annotated at a shifted or differently typed position, e.g. from nucleotide-repeat left/right shifts in VCF), and non-preferred/unknown HGVS syntax (ClinVar or tool syntax not adhering to HGVS recommendations). Compute the percentage of each mismatch category per tool per field so that the category percentages sum with the match rate to 100% — e.g. ANNOVAR HGVSc 79.19% NA, SnpEff HGVSc 78.00% position/type, VEP HGVSc 95.47% position/type. Rationale: the paper found low overall concordance (58.52% HGVSc, 84.04% HGVSp) and needed to attribute the discordance to specific causes per tool rather than report a single match rate. Write the resulting per-tool, per-field, per-category percentage table to mismatch_categories.csv for downstream audit and cross-tool comparison.

## Related tools

- **ANNOVAR** (Source of tool-generated HGVSc/HGVSp annotations to compare against ClinVar and categorize mismatch causes for)
- **SnpEff** (Source of tool-generated HGVSc/HGVSp annotations to compare against ClinVar and categorize mismatch causes for)
- **Variant Effect Predictor (VEP)** (Source of tool-generated HGVSc/HGVSp annotations to compare against ClinVar and categorize mismatch causes for; can be run with --per_gene and --coding_only to streamline transcripts)
- **ClinVar** (Reference standard HGVS nomenclature used to classify each tool annotation as match or mismatch)

## Examples

```
import pandas as pd; df = pd.read_csv('annotations_vs_clinvar.tsv', sep='\t'); df.groupby(['tool','field','category']).size().groupby(level=[0,1]).apply(lambda s: 100*s/s.sum()).to_csv('mismatch_categories.csv')
```

## Evaluation signals

- Overall reproduced match rates are consistent with the reported concordance: 58.52% for HGVSc and 84.04% for HGVSp
- ANNOVAR HGVSc mismatches reproduce the reported 79.19% missing (NA) category
- SnpEff mismatches reproduce the reported 78.00% position/variant-type category
- VEP HGVSc mismatches reproduce the reported 95.47% position/variant-type category
- For every tool and field, the per-category percentages plus the match rate sum to exactly 100%

## Limitations

- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations, so some 'mismatches' are reference-standard artifacts (see Table S3 and Table S4 of the paper)
- Discrepancies with ClinVar persisted even after restricting annotations to ClinVar-selected transcripts
- Nucleotide-repeat variants shifted left or right in the gene/transcript produce inconsistent HGVS notation depending on the VCF caller, inflating the position/variant-type category
- Inconsistencies exist between ClinVar web pages and its VCF, and no software versions were reported for the compared annotation tools

## Evidence

- [abstract] reported concordance rates: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [discussion] ClinVar non-preferred syntax: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [discussion] persistent discrepancies after transcript restriction: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
- [intro] repeat-shift HGVS notation: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation. In cases of nucleotide repeats, shifts to the left or right concerning the gene or transcript"
