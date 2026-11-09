---
name: variant-coding-impact-annotation
description: Use when you need to annotate coding variants with multiple tools (ANNOVAR,
  SnpEff, VEP), normalize their impact terms to SO, and quantify agreement against
  ClinVar-annotated coding impact.
when_to_use_negative:
- The goal is variant nomenclature (HGVSc/HGVSp) concordance rather than predicted
  coding impact — the paper reports these as separate metrics (HGVSc 58.52%, HGVSp
  84.04%).
- The variant set is not phased and normalized, or contains unnormalized multi-nucleotide
  variants (MNVs), which produce inconsistent HGVS notation and unreliable impact
  calls.
- Only a single annotation tool is available — the skill's core value is cross-tool
  concordance against ClinVar, not single-tool annotation.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Annotates coding impact of the 164,549 two-star ClinVar variants; achieved
    0.990 overall and 0.983 LoF concordance with ClinVar after SO normalization.
- name: SnpEff
  role: Annotates coding impact on the same variant set; achieved 0.992 overall and
    0.975 LoF concordance, and the highest HGVSc match (0.988).
- name: Variant Effect Predictor (VEP)
  role: Annotates coding impact using --per_gene and --coding_only to streamline relevant
    transcripts; achieved 0.994 overall and 0.972 LoF concordance.
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:f5e28017a86a8d386086f6ce51b54dcc4daaa93f0433891a1044001bfcd40232
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - VEP
  - Variant Effect Predictor (VEP)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# variant-coding-impact-annotation

## Summary

Annotates a benchmark variant set (164,549 two-star ClinVar variants) with ANNOVAR, SnpEff, and VEP, normalizes each tool's predicted coding-impact terms to a common Sequence Ontology vocabulary, and computes per-tool concordance against ClinVar-reported impact (overall coding-impact agreement 85.58%).

## When to use

Use when you need to annotate coding variants with multiple tools (ANNOVAR, SnpEff, VEP), normalize their impact terms to SO, and quantify agreement against ClinVar-annotated coding impact.

## When NOT to use

- The goal is variant nomenclature (HGVSc/HGVSp) concordance rather than predicted coding impact — the paper reports these as separate metrics (HGVSc 58.52%, HGVSp 84.04%).
- The variant set is not phased and normalized, or contains unnormalized multi-nucleotide variants (MNVs), which produce inconsistent HGVS notation and unreliable impact calls.
- Only a single annotation tool is available — the skill's core value is cross-tool concordance against ClinVar, not single-tool annotation.

## Inputs

- ClinVar two-star (high review status) variant set, 164,549 variants (VCF)
- Identical reference genome and transcript annotation (e.g., MANE) files shared across tools
- ANNOVAR, SnpEff, and VEP annotation outputs for the same variant set

## Outputs

- Per-variant normalized (Sequence Ontology) coding-impact calls from each tool
- Per-tool overall coding-impact agreement rates (target ANNOVAR 0.990, SnpEff 0.992, VEP 0.994)
- Per-tool LoF-category concordance values (31,640 LoF variants) written to concordance_table.csv

## How to apply

Obtain the ClinVar two-star (high review status) benchmark set of 164,549 variants and annotate the identical VCF with ANNOVAR, SnpEff, and VEP using identical reference genomes and transcript annotations across all three tools, since transcript selection (e.g., MANE) materially changes the final annotation. Normalize tool-specific coding-impact labels into a shared Sequence Ontology vocabulary; when mappings are ambiguous, prioritize severe/high-impact classes because they are often more clinically relevant. For VEP, use the --per_gene and --coding_only flags to streamline relevant transcripts and reduce complexity/errors. Compare each tool's normalized call to the ClinVar-reported impact per variant and compute per-tool agreement, targeting ANNOVAR 0.990, SnpEff 0.992, VEP 0.994, and restrict to the Loss-of-Function category (31,640 variants) for LoF concordance of 0.983 (ANNOVAR), 0.975 (SnpEff), and 0.972 (VEP). Write the per-tool overall and LoF concordance values to concordance_table.csv and validate against the reported rates within rounding tolerance (three-tool coding-impact level 85.58%). Note that MNVs must be phased and normalized in the VCF before annotation, since caller-dependent left/right shifting produces inconsistent HGVS notation.

## Related tools

- **ANNOVAR** (Annotates coding impact of the 164,549 two-star ClinVar variants; achieved 0.990 overall and 0.983 LoF concordance with ClinVar after SO normalization.)
- **SnpEff** (Annotates coding impact on the same variant set; achieved 0.992 overall and 0.975 LoF concordance, and the highest HGVSc match (0.988).)
- **Variant Effect Predictor (VEP)** (Annotates coding impact using --per_gene and --coding_only to streamline relevant transcripts; achieved 0.994 overall and 0.972 LoF concordance.)

## Examples

```
vep --per_gene --coding_only -i clinvar_2star.vcf --cache --vcf
```

## Evaluation signals

- Per-tool overall coding-impact agreement must match 0.990 (ANNOVAR), 0.992 (SnpEff), and 0.994 (VEP) within rounding tolerance.
- LoF-category concordance over 31,640 variants must equal 0.983 (ANNOVAR), 0.975 (SnpEff), and 0.972 (VEP).
- Three-tool coding-impact concordance level should be near the reported 85.58%.
- All tool-specific impact labels must resolve to a common Sequence Ontology vocabulary before comparison.
- concordance_table.csv contains both overall and LoF per-tool agreement values and passes the rounding-tolerance check against reported rates.

## Limitations

- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones, so transcript restriction alone does not eliminate discordance.
- No software versions were reported for ANNOVAR, SnpEff, and VEP, complicating exact reproducibility; version control per tool is needed for data reanalysis.
- ClinVar itself contains HGVS syntax that does not adhere to HGVS recommendations (Table S3/S4), and inconsistencies exist between ClinVar web pages and its VCF.

## Evidence

- [abstract] This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [abstract] The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact.: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [intro] Annotation tools like ANNOVAR14, SnpEff15, and VEP16 utilize RNA transcripts to annotate variants by using overlapping genomic characteristics.: "Annotation tools like ANNOVAR14, SnpEff15, and VEP16 utilize RNA transcripts to annotate variants by using overlapping genomic characteristics."
- [discussion] Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools.: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
- [discussion] using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors.: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
