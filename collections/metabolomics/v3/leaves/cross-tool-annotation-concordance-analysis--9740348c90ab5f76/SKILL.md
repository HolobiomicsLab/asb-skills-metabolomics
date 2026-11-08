---
name: cross-tool-annotation-concordance-analysis
description: Use when multiple variant annotation tools (ANNOVAR, SnpEff, VEP) have
  been run on the same VCF and you need to quantify HGVS/coding-impact disagreement,
  benchmark against ClinVar, or assess ACMG/PVS1 impact.
when_to_use_negative:
- The data are metabolomics/genomics linking tasks such as NPLinker MIBiG/GNPS analyses
  — this skill concerns variant annotation concordance, not natural product linking.
- Only a single annotation tool was run and no ClinVar (or other clinical reference)
  HGVS annotation exists to serve as a comparison baseline.
- The goal is de novo variant calling or normalization of raw reads rather than comparing
  already-generated annotation outputs across tools.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: ANNOVAR
  role: One of the three annotation tools run on the standardized two-star ClinVar
    VCF; its HGVSc/HGVSp and coding-impact outputs are string-matched against ClinVar
    and used for PVS1 assignment.
- name: SnpEff
  role: Annotation tool whose output is compared against ClinVar; achieved the highest
    HGVSc match fraction (0.988) in the study.
- name: Variant Effect Predictor (VEP)
  role: Annotation tool run with --per_gene and --coding_only to streamline transcripts;
    achieved the highest HGVSp match fraction (0.991) in the study.
- name: ClinVar
  role: Source of the 164,549 two-star variants and the deposited HGVS nomenclature
    used as the ground truth for concordance computation.
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:1ed596595d28d20b38a74a133bac34885c3248dabeec40269840c1e97f934580
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Cross-tool annotation concordance analysis

## Summary

A benchmarking skill that quantifies how consistently ANNOVAR, SnpEff, and VEP annotate the same variants (HGVSc, HGVSp, coding impact) against ClinVar ground truth, and traces downstream ACMG consequences such as erroneous PVS1 assignment.

## When to use

Use when multiple variant annotation tools (ANNOVAR, SnpEff, VEP) have been run on the same VCF and you need to quantify HGVS/coding-impact disagreement, benchmark against ClinVar, or assess ACMG/PVS1 impact.

## When NOT to use

- The data are metabolomics/genomics linking tasks such as NPLinker MIBiG/GNPS analyses — this skill concerns variant annotation concordance, not natural product linking.
- Only a single annotation tool was run and no ClinVar (or other clinical reference) HGVS annotation exists to serve as a comparison baseline.
- The goal is de novo variant calling or normalization of raw reads rather than comparing already-generated annotation outputs across tools.

## Inputs

- ClinVar VCF filtered to 164,549 two-star (high review status) variants
- ClinVar-deposited HGVS strings (HGVSc and HGVSp) as the concordance reference
- Per-tool annotation outputs from ANNOVAR, SnpEff, and VEP with HGVSc, HGVSp, and coding impact fields

## Outputs

- Per-variant match/mismatch tallies for HGVSc, HGVSp, and coding impact per tool
- Overall concordance rates (HGVSc, HGVSp, coding impact) and per-tool match fractions
- Per-tool PVS1 loss-of-function/gain-of-function assignment counts (ANNOVAR 185/74, SnpEff 161/50, VEP 67/12)
- Per-tool PLP downgrade fractions from incorrect PVS1 (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), e.g. pvs1_plp_downgrade_summary.csv

## How to apply

Filter a ClinVar VCF to high-review-status ('two-star') variants (164,549 in the reference study), then run ANNOVAR, SnpEff, and VEP on the standardized VCF using identical reference genomes and transcript annotations across all tools; for VEP use the plug-in flags --per_gene and --coding_only to streamline relevant transcripts. Parse each tool's output to extract HGVSc, HGVSp, and coding impact strings per variant, and string-match them against the ClinVar-deposited HGVS strings, tallying matches and mismatches. Compute overall concordance rates and per-tool match fractions (expected: 58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact; SnpEff HGVSc 0.988, VEP HGVSp 0.991). Then propagate discrepancies into ACMG interpretation: apply PVS1 from each tool's predicted loss-of-function coding impact, and quantify how many pathogenic/likely-pathogenic (PLP) variants are incorrectly downgraded per tool. The rationale is that transcript-selection variability and HGVS nomenclature differences (e.g., left/right shifts in nucleotide repeats from different VCF callers) silently change clinical classification, so restricting to ClinVar-selected transcripts and cross-validating across independent tools is required for reliable interpretation.

## Related tools

- **ANNOVAR** (One of the three annotation tools run on the standardized two-star ClinVar VCF; its HGVSc/HGVSp and coding-impact outputs are string-matched against ClinVar and used for PVS1 assignment.)
- **SnpEff** (Annotation tool whose output is compared against ClinVar; achieved the highest HGVSc match fraction (0.988) in the study.)
- **Variant Effect Predictor (VEP)** (Annotation tool run with --per_gene and --coding_only to streamline transcripts; achieved the highest HGVSp match fraction (0.991) in the study.)
- **ClinVar** (Source of the 164,549 two-star variants and the deposited HGVS nomenclature used as the ground truth for concordance computation.)

## Examples

```
vep --per_gene --coding_only -i clinvar_2star.vcf --hgvs --vcf out.vcf
```

## Evaluation signals

- Overall computed agreement must equal 58.52% HGVSc, 84.04% HGVSp, and 85.58% coding impact within rounding tolerance.
- Per-tool match fractions must match reported values: SnpEff 0.988 HGVSc, VEP 0.977 (abstract) / 0.991 HGVSp, ANNOVAR 0.753 HGVSc.
- Reproduced PVS1 loss/gain counts must equal ANNOVAR 185/74, SnpEff 161/50, VEP 67/12.
- PLP downgrade fractions must match 55.9% (ANNOVAR), 66.5% (SnpEff), and 67.3% (VEP).
- All three tools must have been run on the same reference genome and transcript annotations over the same filtered variant set (identical N = 164,549).

## Limitations

- Restricting transcripts to ClinVar-selected ones improved consistency, but discrepancies with ClinVar persisted even after this restriction.
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (Tables S3 and S4), and inconsistencies exist between ClinVar web pages and its VCF, contaminating the reference set.
- No software versions were reported for the compared annotation tools, hindering exact reanalysis; different VCF callers also shift HGVS notation for nucleotide repeats left or right.

## Evidence

- [abstract] Study scope on three tools and ClinVar set: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [abstract] Overall concordance rates to reproduce: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [abstract] Per-tool match fractions: "SnpEff showed the highest match for HGVSc (0.988), while VEP bettered for HGVSp (0.977)."
- [abstract] ACMG/PVS1 downstream impact: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)"
- [discussion] Standardized inputs across tools: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
- [discussion] VEP flag recommendation: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
- [discussion] Persistent discrepancies after transcript restriction: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
