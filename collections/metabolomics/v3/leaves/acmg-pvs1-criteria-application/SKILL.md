---
name: acmg-pvs1-criteria-application
description: Use when per-tool coding-impact annotations exist for ClinVar variants
  and you must assign PVS1, count LoF/gain-of-function assignments per tool, or quantify
  PLP downgrades caused by inconsistent annotations.
when_to_use_negative:
- The goal is variant annotation itself rather than evaluating how annotation differences
  affect ACMG/PVS1 classification
- The dataset contains only one annotation tool's output, so cross-tool PVS1 discrepancy
  cannot be assessed
- The task is metabolomics/genomics linkage with NPLinker (GNPS, AntiSMASH BGC data),
  which is unrelated to variant annotation despite repos appearing in the same index
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Source of coding-impact annotations from which PVS1 is derived; yielded 185
    LoF/74 GoF PVS1 assignments and 55.9% PLP downgrades
- name: SnpEff
  role: Source of coding-impact annotations from which PVS1 is derived; yielded 161
    LoF/50 GoF PVS1 assignments and 66.5% PLP downgrades
- name: Variant Effect Predictor (VEP)
  role: Source of coding-impact annotations from which PVS1 is derived; yielded 67
    LoF/12 GoF PVS1 assignments and 67.3% PLP downgrades
- name: ClinVar
  role: Reference clinical database supplying the two-star variant set and PLP status
    used as ground truth for downgrade fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:3d58bb026b3aa904a863b7c837065afecfcd592707955842ad92e876f78b283d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# acmg-pvs1-criteria-application

## Summary

A skill for applying the ACMG PVS1 rule from tool-predicted loss-of-function coding impacts and quantifying how annotation discordance between ANNOVAR, SnpEff, and VEP downgrades pathogenic/likely-pathogenic (PLP) ClinVar variants. It exposes how tool-specific coding-impact calls propagate into final ACMG pathogenicity classifications.

## When to use

Use when per-tool coding-impact annotations exist for ClinVar variants and you must assign PVS1, count LoF/gain-of-function assignments per tool, or quantify PLP downgrades caused by inconsistent annotations.

## When NOT to use

- The goal is variant annotation itself rather than evaluating how annotation differences affect ACMG/PVS1 classification
- The dataset contains only one annotation tool's output, so cross-tool PVS1 discrepancy cannot be assessed
- The task is metabolomics/genomics linkage with NPLinker (GNPS, AntiSMASH BGC data), which is unrelated to variant annotation despite repos appearing in the same index

## Inputs

- Per-tool coding-impact annotations for 164,549 two-star ClinVar variants produced by ANNOVAR, SnpEff, and VEP
- ClinVar review-status (two-star) variant list
- ClinVar pathogenic/likely-pathogenic (PLP) clinical significance status
- ACMG guideline rule definitions (PVS1, PM2, PM3)

## Outputs

- pvs1_plp_downgrade_summary.csv with per-tool PVS1 loss/gain-of-function counts (ANNOVAR 185/74, SnpEff 161/50, VEP 67/12)
- Per-tool PLP downgrade fractions (55.9%, 66.5%, 67.3%)
- Per-variant PVS1 assignment table annotated by tool

## How to apply

Load the per-tool coding-impact annotations for the 164,549 two-star ClinVar variants annotated by ANNOVAR, SnpEff, and VEP. For each tool, identify variants with predicted loss-of-function coding impact, since PVS1 is assigned only from predicted LoF coding impacts under the ACMG rule application in the paper. Apply PVS1 per tool and tabulate PVS1-applied loss-of-function and gain-of-function counts (expected: ANNOVAR 185/74, SnpEff 161/50, VEP 67/12). For each tool, determine which ClinVar PLP variants receive an incorrect PVS1 interpretation and compute the downgrade fraction, comparing against the reported 55.9% (ANNOVAR), 66.5% (SnpEff), and 67.3% (VEP). Write the per-tool counts and fractions to pvs1_plp_downgrade_summary.csv, and interpret discrepancies in light of the paper's concordance rates (58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact).

## Related tools

- **ANNOVAR** (Source of coding-impact annotations from which PVS1 is derived; yielded 185 LoF/74 GoF PVS1 assignments and 55.9% PLP downgrades)
- **SnpEff** (Source of coding-impact annotations from which PVS1 is derived; yielded 161 LoF/50 GoF PVS1 assignments and 66.5% PLP downgrades)
- **Variant Effect Predictor (VEP)** (Source of coding-impact annotations from which PVS1 is derived; yielded 67 LoF/12 GoF PVS1 assignments and 67.3% PLP downgrades)
- **ClinVar** (Reference clinical database supplying the two-star variant set and PLP status used as ground truth for downgrade fractions)

## Examples

```
vep --per_gene --coding_only -i clinvar_two_star.vcf -o vep_annotated.vcf --vcf --format vcf
```

## Evaluation signals

- Computed PVS1 loss/gain counts per tool exactly equal the reported values: ANNOVAR 185/74, SnpEff 161/50, VEP 67/12
- Computed PLP downgrade fractions match 55.9% (ANNOVAR), 66.5% (SnpEff), and 67.3% (VEP)
- Variant universe totals 164,549 two-star ClinVar variants with no variants dropped between tools
- Coding-impact category concordance across tools is consistent with the reported 85.58% (with HGVSc 58.52% and HGVSp 84.04% as context)
- pvs1_plp_downgrade_summary.csv is produced with per-tool rows and both count and fraction columns populated

## Limitations

- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations, confounding direct annotation comparison
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones
- No software versions are reported for ANNOVAR, SnpEff, or VEP, limiting reproducibility
- Substantial discrepancies concentrate in the Loss-of-Function (LoF) category, so PVS1 results are especially sensitive to transcript selection and MANE choice

## Evidence

- [abstract] Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives of: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives of"
- [abstract] The substantial discrepancies were noted in the Loss-of-Function (LoF) category.: "The substantial discrepancies were noted in the Loss-of-Function (LoF) category."
- [discussion] using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors.: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
- [abstract] The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact.: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [discussion] Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools.: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
