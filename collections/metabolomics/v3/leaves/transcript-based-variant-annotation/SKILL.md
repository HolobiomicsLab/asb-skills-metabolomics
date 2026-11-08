---
name: transcript-based-variant-annotation
description: Use when you need to assess how consistently RNA-transcript-based annotators
  (ANNOVAR, SnpEff, VEP) reproduce ClinVar HGVSc/HGVSp nomenclature and predicted
  coding impacts, or to diagnose why tool annotations disagree with ClinVar.
when_to_use_negative:
- When the goal is ACMG rule impact (e.g. PVS1/LoF downgrading of PLP variants) rather
  than HGVS nomenclature and coding-impact concordance itself
- When the variant set has no clinical reference annotations (e.g. no ClinVar two-star
  HGVSc/HGVSp strings) to benchmark against
- When a single tool's output is the required deliverable and no cross-tool validation
  or ClinVar comparison is intended
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ANNOVAR
  role: Transcript-based annotator whose HGVSc/HGVSp and coding-impact output is compared
    against ClinVar; its HGVSc mismatches were dominated by missing (NA) annotations
    (79.19%)
- name: SnpEff
  role: Transcript-based annotator compared against ClinVar; achieved the highest
    HGVSc match (0.988) with 78.00% of HGVSc mismatches attributable to position/variant-type
    differences
- name: Variant Effect Predictor (VEP)
  role: Transcript-based annotator compared against ClinVar; achieved the highest
    HGVSp match (0.977) with 95.47% of HGVSc mismatches attributable to position/variant-type
    differences; supports --per_gene and --coding_only options to streamline relevant
    transcripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:f22da6f88ac1cd10ee52dc27093c38e38822b917273eac21b997725276f858b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transcript-based-variant-annotation

## Summary

Annotate clinical variants with transcript-dependent nomenclature (HGVSc/HGVSp) and coding impact using ANNOVAR, SnpEff, and VEP, then benchmark each tool's output against ClinVar reference annotations for high-review-status (two-star) variants.

## When to use

Use when you need to assess how consistently RNA-transcript-based annotators (ANNOVAR, SnpEff, VEP) reproduce ClinVar HGVSc/HGVSp nomenclature and predicted coding impacts, or to diagnose why tool annotations disagree with ClinVar.

## When NOT to use

- When the goal is ACMG rule impact (e.g. PVS1/LoF downgrading of PLP variants) rather than HGVS nomenclature and coding-impact concordance itself
- When the variant set has no clinical reference annotations (e.g. no ClinVar two-star HGVSc/HGVSp strings) to benchmark against
- When a single tool's output is the required deliverable and no cross-tool validation or ClinVar comparison is intended

## Inputs

- VCF of variants to annotate, phased and normalized for MNVs
- ClinVar reference HGVSc/HGVSp strings for high-review-status (two-star) variants
- Reference genome and transcript annotations identical across all tools (e.g. RefSeq/Ensembl transcripts, MANE Select set)

## Outputs

- Per-tool HGVSc/HGVSp/coding-impact annotations for each variant
- Per-tool per-field match/mismatch classification
- Mismatch-category breakdown (missing/NA, position or variant-type difference, non-preferred/unknown syntax) as percentages, e.g. written to mismatch_categories.csv
- Concordance rates per field and per tool (e.g. SnpEff HGVSc 0.988, VEP HGVSp 0.977)

## How to apply

Load the ClinVar reference HGVSc/HGVSp strings for 164,549 two-star ClinVar variants and run ANNOVAR, SnpEff, and VEP over the same variant set, ensuring all tools use identical reference genomes and transcript annotations so differences reflect annotation logic rather than input skew. For each tool and each field (HGVSc, HGVSp, coding impact), classify each record as match or mismatch, and decompose mismatches into three classes: missing (NA) annotations, position/variant-type differences (shifted or differently typed positions, e.g. left/right shifts in nucleotide repeats), and non-preferred/unknown HGVS syntax that violates HGVS recommendations. Compute per-tool per-field concordance and mismatch-category percentages so that category percentages plus match rate sum to 100% (reference results: 58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact). Restricting transcripts to ClinVar-selected or MANE transcripts improves consistency, but some discrepancies persist, so cross-validate outputs across multiple independent tools rather than trusting a single annotator. Phase and normalize multi-nucleotide variants (MNVs) in the input VCF before annotation, since caller-dependent normalization drives HGVS shifts.

## Related tools

- **ANNOVAR** (Transcript-based annotator whose HGVSc/HGVSp and coding-impact output is compared against ClinVar; its HGVSc mismatches were dominated by missing (NA) annotations (79.19%))
- **SnpEff** (Transcript-based annotator compared against ClinVar; achieved the highest HGVSc match (0.988) with 78.00% of HGVSc mismatches attributable to position/variant-type differences)
- **Variant Effect Predictor (VEP)** (Transcript-based annotator compared against ClinVar; achieved the highest HGVSp match (0.977) with 95.47% of HGVSc mismatches attributable to position/variant-type differences; supports --per_gene and --coding_only options to streamline relevant transcripts)

## Examples

```
vep input.vcf --per_gene --coding_only
```

## Evaluation signals

- Reproduced concordance rates match the reported values: 58.52% for HGVSc, 84.04% for HGVSp, and 85.58% for coding impact against ClinVar two-star variants
- Per-tool mismatch-category percentages reproduce the reported breakdown: ANNOVAR HGVSc 79.19% missing (NA), SnpEff 78.00% position/variant-type, VEP HGVSc 95.47% position/variant-type
- For every tool and HGVS field, the mismatch-category percentages plus the match rate sum to 100%
- Tool-level extrema reproduce: SnpEff highest HGVSc match (0.988) and VEP highest HGVSp match (0.977)
- Restricting to ClinVar-selected or MANE transcripts measurably improves HGVSc, HGVSp, and coding-impact consistency, though residual discrepancies remain

## Limitations

- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations (Table S3 and Table S4), so a mismatch may reflect ClinVar-side non-preferred syntax rather than a tool error
- VCFs generated by different callers produce inconsistent HGVS notation, especially for nucleotide repeats shifted left or right relative to the gene or transcript
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones
- No software versions are reported for the compared annotation tools, and inconsistencies exist between ClinVar web pages and its VCF

## Evidence

- [abstract] Concordance rates for key fields: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [abstract] Tool-level match extrema: "SnpEff showed the highest match for HGVSc (0.988), while VEP bettered for HGVSp (0.977)."
- [intro] Rationale: annotators depend on transcript selection: "Annotation tools like ANNOVAR14, SnpEff15, and VEP16 utilize RNA transcripts to annotate variants by using overlapping genomic characteristics."
- [discussion] Normalization prerequisite for input VCF: "Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF."
- [discussion] Cross-tool validation recommendation: "Implement cross-validation by comparing outputs from multiple independent tools for the same variants to identify discrepancies and determine the most accurate annotations."
- [discussion] ClinVar-side syntax caveat: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [intro] Caller-dependent HGVS shifts in repeats: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation. In cases of nucleotide repeats, shifts to the left or right concerning the gene or transcript"
- [discussion] VEP options to reduce transcript complexity: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
