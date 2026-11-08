---
name: clinvar-clinical-significance-classification
description: Use when you need a curated, review-status-filtered ClinVar variant set
  (two-star variants) with clinical-significance labels to benchmark annotation tools
  like ANNOVAR, SnpEff, or VEP.
when_to_use_negative:
- When the task requires variants of low or unspecified review status (the pipeline
  is restricted to two-star ClinVar records)
- When analyzing mitochondrial variants or large indels (≥50 bp) — these are explicitly
  excluded by the pipeline
- When the goal is discovery of variants of uncertain significance (VUS) rather than
  a labeled benchmark set
edam_operation: http://edamontology.org/operation_0366
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3234
tools:
- name: bcftools
  role: Left-aligns (normalizes) variants so repeat-region indels are represented
    consistently regardless of the original variant caller
- name: ANNOVAR
  role: One of the annotation tools applied downstream to the filtered 164,549-variant
    set for HGVS and coding-impact concordance evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:cec2bece54b010100095462e1a1df6e1fd9eecc4354052399f10bc4a7dfd8d11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - bcftools
  edam_operation: http://edamontology.org/operation_0366
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3234
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# clinvar-clinical-significance-classification

## Summary

Builds a high-confidence ClinVar test set of 164,549 two-star variants with defined clinical-significance labels (Pathogenic, Likely Pathogenic, Benign, Likely Benign) by downloading the January 7, 2024 ClinVar VCF and applying a normalization-plus-exclusion filtering chain. It serves as the gold-standard input for benchmarking variant annotation tools.

## When to use

Use when you need a curated, review-status-filtered ClinVar variant set (two-star variants) with clinical-significance labels to benchmark annotation tools like ANNOVAR, SnpEff, or VEP.

## When NOT to use

- When the task requires variants of low or unspecified review status (the pipeline is restricted to two-star ClinVar records)
- When analyzing mitochondrial variants or large indels (≥50 bp) — these are explicitly excluded by the pipeline
- When the goal is discovery of variants of uncertain significance (VUS) rather than a labeled benchmark set

## Inputs

- ClinVar VCF release dated January 7, 2024 (NCBI)
- Reference genome FASTA for bcftools left-alignment/normalization
- Molecular Consequence (MC) annotations from ClinVar records

## Outputs

- Filtered VCF containing exactly 164,549 two-star variants
- Clinical-significance class assignment per variant: 38,278 Pathogenic, 6,073 Likely Pathogenic, 57,081 Benign, 63,117 Likely Benign
- Normalized (left-aligned) variant records suitable for cross-tool annotation comparison

## How to apply

Download the ClinVar VCF release dated January 7, 2024 from NCBI. Left-align (normalize) all variants with bcftools so repeat-region indels are represented consistently regardless of the original variant caller — this matters because VCFs from different callers yield inconsistent HGVS notation for nucleotide repeats. Remove duplicate records and records with degenerate (ambiguous) bases, then drop variants meeting the exclusion criteria: indels of 50 bp or larger, VUS entries, mitochondrial (MT) variants, and variants with missing or multiple Molecular Consequence (MC) annotations. Validate that the output contains exactly 164,549 variants with the reported composition of 38,278 Pathogenic, 6,073 Likely Pathogenic, 57,081 Benign, and 63,117 Likely Benign variants. Optionally restrict transcripts to those selected by ClinVar when annotating, though discrepancies with ClinVar annotations still persist.

## Related tools

- **bcftools** (Left-aligns (normalizes) variants so repeat-region indels are represented consistently regardless of the original variant caller)
- **ANNOVAR** (One of the annotation tools applied downstream to the filtered 164,549-variant set for HGVS and coding-impact concordance evaluation)

## Examples

```
bcftools norm -f reference.fa clinvar_20240107.vcf.gz | bcftools view -e 'INDEL AND (LEN>=50)' -Oz -o clinvar_filtered.vcf.gz
```

## Evaluation signals

- Final filtered VCF must contain exactly 164,549 variants
- Per-class counts must match the reported composition: 38,278 Pathogenic / 6,073 Likely Pathogenic / 57,081 Benign / 63,117 Likely Benign
- No VUS, mitochondrial (MT), or missing/multi-Molecular-Consequence records remain in the output
- No duplicate records or degenerate (ambiguous) bases remain after deduplication
- All indels of 50 bp or larger have been dropped from the final set

## Limitations

- Restricting transcripts to those selected by ClinVar improved consistency but discrepancies with ClinVar persisted
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations, and inconsistencies exist between ClinVar web pages and its VCF
- Different MANE selections and left/right shifts in nucleotide repeats can impact final variant annotation

## Evidence

- [abstract] two-star ClinVar test set size: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [intro] rationale for left-alignment normalization: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation"
- [full_text] filtering chain yields 164,549 variants with reported composition: [evidence span withheld — anchor status: not_found]
- [discussion] ClinVar internal inconsistencies limit perfect benchmarking: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
