---
name: variant-left-alignment-normalization
description: Use when building a variant benchmark from a ClinVar VCF for cross-tool
  (ANNOVAR/SnpEff/VEP) annotation comparison, where caller-dependent left/right shifts
  of nucleotide-repeat indels would produce inconsistent HGVS notation.
when_to_use_negative:
- The input VCF is already left-aligned and normalized against the same reference
  genome and transcript annotation as the downstream annotation tools.
- The analysis uses a single annotation tool and does not compare HGVS nomenclature
  or coding-impact predictions across callers, so caller-dependent repeat-shift ambiguity
  does not affect results.
- The task involves natural-products data mining (NPLinker/GNPS/AntiSMASH workflows)
  rather than germline variant normalization.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3268
tools:
- name: bcftools
  role: Left-aligns (normalizes) all VCF records so repeat-region indels are represented
    consistently regardless of the original variant caller
- name: ANNOVAR
  role: Downstream annotation tool that consumes the normalized variant set for HGVS/coding-impact
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:5f2cce01a43fd6ab72c839fd75cdbd02198e8d72876a32381292fc9d80913be7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - bcftools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3268
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# variant-left-alignment-normalization

## Summary

Left-aligns (normalizes) VCF records with bcftools so that repeat-region indels are represented identically regardless of the original variant caller, then applies a filtering chain (dedup, degenerate-base removal, indel/record exclusions) to produce a clean ClinVar-derived test set for cross-tool annotation comparison.

## When to use

Use when building a variant benchmark from a ClinVar VCF for cross-tool (ANNOVAR/SnpEff/VEP) annotation comparison, where caller-dependent left/right shifts of nucleotide-repeat indels would produce inconsistent HGVS notation.

## When NOT to use

- The input VCF is already left-aligned and normalized against the same reference genome and transcript annotation as the downstream annotation tools.
- The analysis uses a single annotation tool and does not compare HGVS nomenclature or coding-impact predictions across callers, so caller-dependent repeat-shift ambiguity does not affect results.
- The task involves natural-products data mining (NPLinker/GNPS/AntiSMASH workflows) rather than germline variant normalization.

## Inputs

- ClinVar VCF release dated January 7, 2024 (NCBI download)
- Reference genome FASTA for bcftools left alignment

## Outputs

- Normalized, filtered VCF containing exactly 164,549 two-star ClinVar variants
- Clinical-significance composition counts: 38,278 Pathogenic / 6,073 Likely Pathogenic / 57,081 Benign / 63,117 Likely Benign

## How to apply

Download the ClinVar VCF release dated January 7, 2024 from NCBI and left-align (normalize) all records with bcftools so repeat-region indels are represented consistently regardless of the original variant caller. Remove duplicate records and records containing degenerate (ambiguous) bases. Drop variants matching the exclusion criteria: indels of 50 bp or larger, VUS entries, mitochondrial (MT) variants, and variants with missing or multiple Molecular Consequence (MC) annotations. Normalize multi-nucleotide variants and ensure all downstream tools use identical reference genomes and transcript annotations, since different MANE selections and caller-specific shifts change the final HGVS annotation. Validate the output by counting variants and per-class clinical-significance composition before handing the set to annotation tools.

## Related tools

- **bcftools** (Left-aligns (normalizes) all VCF records so repeat-region indels are represented consistently regardless of the original variant caller)
- **ANNOVAR** (Downstream annotation tool that consumes the normalized variant set for HGVS/coding-impact comparison)

## Examples

```
bcftools norm -f reference.fa clinvar_20240107.vcf.gz -Oz -o clinvar_left_aligned.vcf.gz
```

## Evaluation signals

- The filtered file must contain exactly 164,549 variants.
- Per-class counts must match the reported composition: 38,278 Pathogenic, 6,073 Likely Pathogenic, 57,081 Benign, 63,117 Likely Benign.
- No remaining records contain degenerate (ambiguous) bases or duplicate entries.
- No remaining variants are indels of 50 bp or larger, VUS entries, MT variants, or records with missing/multiple MC annotations.
- Repeat-region indels produce identical HGVS representation when re-annotated by independent tools (ANNOVAR, SnpEff, VEP).

## Limitations

- Even after left alignment, discrepancies with ClinVar persisted once annotations were compared — restricting transcripts to ClinVar-selected ones improved consistency but did not eliminate differences.
- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations, and inconsistencies exist between ClinVar web pages and its VCF, limiting perfect normalization.
- Different MANE transcript selections still impact the final annotation even after normalization.

## Evidence

- [intro] Caller-dependent HGVS shifts motivate normalization: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation."
- [discussion] Normalization requirement for MNVs: "Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF."
- [discussion] Uniform reference and transcript annotations across tools: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
