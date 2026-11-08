---
name: genomic-variant-qc-validation
description: Use when a raw ClinVar VCF must be cleaned, normalized, and QC-validated
  to a fixed variant count and clinical-significance composition before it is used
  as a benchmark for annotation-tool concordance (ANNOVAR, SnpEff, VEP).
when_to_use_negative:
- The input VCF is already left-aligned, deduplicated, and free of degenerate bases
  — re-applying normalization can shift repeat-region HGVS notation and produce inconsistent
  annotations.
- The goal is to annotate novel variants rather than build a fixed high-review-status
  benchmark set — the strict exclusion of VUS, MT, and indels ≥50 bp would remove
  variants that are needed.
- The variant set contains MNVs that have not been phased and normalized — running
  annotation tools first would yield inconsistent HGVS nomenclature rather than true
  tool disagreement.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0625
tools:
- name: bcftools
  role: Left-aligns (normalizes) all variants so repeat-region indels are represented
    consistently regardless of the original variant caller
- name: ANNOVAR
  role: Downstream annotation tool whose output concordance (58.52% HGVSc, 84.04%
    HGVSp, 85.58% coding impact) is compared against the QC'd benchmark set
- name: SnpEff
  role: Downstream annotation tool compared against ANNOVAR and VEP on the validated
    ClinVar benchmark (highest HGVSc match 0.988; PVS1 downgraded 66.5% of PLP variants)
- name: Variant Effect Predictor (VEP)
  role: Downstream annotation tool compared on the validated set (highest HGVSp match
    0.977); supports --per_gene and --coding_only flags to streamline relevant transcripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:0d5df8e0f4bfa3397078d2750b562820c9ee66e81a76afdbadeb20c4dfb9124d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  - bcftools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# genomic-variant-qc-validation

## Summary

Normalize and filter a ClinVar VCF (left alignment, deduplication, degenerate-base removal, and exclusion of large indels, VUS, mitochondrial, and missing/multi-MC records) and validate the output against the expected 164,549-variant composition (38,278 P / 6,073 LP / 57,081 B / 63,117 LB) before cross-tool annotation comparison.

## When to use

Use when a raw ClinVar VCF must be cleaned, normalized, and QC-validated to a fixed variant count and clinical-significance composition before it is used as a benchmark for annotation-tool concordance (ANNOVAR, SnpEff, VEP).

## When NOT to use

- The input VCF is already left-aligned, deduplicated, and free of degenerate bases — re-applying normalization can shift repeat-region HGVS notation and produce inconsistent annotations.
- The goal is to annotate novel variants rather than build a fixed high-review-status benchmark set — the strict exclusion of VUS, MT, and indels ≥50 bp would remove variants that are needed.
- The variant set contains MNVs that have not been phased and normalized — running annotation tools first would yield inconsistent HGVS nomenclature rather than true tool disagreement.

## Inputs

- ClinVar VCF release dated January 7, 2024 (NCBI)
- Reference genome and transcript annotations (e.g., RefSeq/Ensembl/MANE) shared across all annotation tools

## Outputs

- Normalized, deduplicated VCF containing 164,549 two-star ClinVar variants
- QC report confirming clinical-significance composition: 38,278 P, 6,073 LP, 57,081 B, 63,117 LB

## How to apply

Download the ClinVar VCF release dated January 7, 2024 from NCBI as the raw input, then left-align (normalize) all variants with bcftools so repeat-region indels are represented consistently regardless of the original variant caller. Remove duplicate records and records containing degenerate (ambiguous) bases, then drop variants matching the article's exclusion criteria: indels of 50 bp or larger, VUS entries, mitochondrial (MT) variants, and variants with missing or multiple Molecular Consequence (MC) annotations. Validate the output against the reported composition of exactly 164,549 variants split as 38,278 Pathogenic, 6,073 Likely Pathogenic, 57,081 Benign, and 63,117 Likely Benign; any mismatch indicates a filtering or normalization error. Multi-nucleotide variants must also be phased and normalized in the VCF, and uniformity of reference genomes and transcript annotations across downstream tools must be ensured so annotation discrepancies are attributable to the tools rather than input inconsistencies.

## Related tools

- **bcftools** (Left-aligns (normalizes) all variants so repeat-region indels are represented consistently regardless of the original variant caller)
- **ANNOVAR** (Downstream annotation tool whose output concordance (58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact) is compared against the QC'd benchmark set)
- **SnpEff** (Downstream annotation tool compared against ANNOVAR and VEP on the validated ClinVar benchmark (highest HGVSc match 0.988; PVS1 downgraded 66.5% of PLP variants))
- **Variant Effect Predictor (VEP)** (Downstream annotation tool compared on the validated set (highest HGVSp match 0.977); supports --per_gene and --coding_only flags to streamline relevant transcripts)

## Examples

```
bcftools norm -f GRCh38.fa -c s clinvar_20240107.vcf.gz | bcftools view -f 'PASS' -e 'GT="het"' -i 'QUAL>=30 && INFO/MC==1' | grep -v -E 'MT|VUS|indel>=50' > clinvar_164549.vcf
```

## Evaluation signals

- After left alignment and deduplication, the variant count must be exactly 164,549 records.
- The clinical-significance composition must match 38,278 Pathogenic, 6,073 Likely Pathogenic, 57,081 Benign, and 63,117 Likely Benign variants.
- No remaining records should contain degenerate (ambiguous) bases.
- No remaining records should be indels of 50 bp or larger.
- All variants in the final set should have exactly one Molecular Consequence (MC) annotation and no mitochondrial (MT) or VUS entries.

## Limitations

- Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation, especially for nucleotide repeats shifted left or right relative to the gene or transcript.
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4), so benchmark consistency is bounded by ClinVar's own annotation quality.
- Inconsistencies exist between ClinVar web pages and its VCF, which can affect the composition of the benchmark set.
- No software versions were reported for the compared annotation tools, so exact reproducibility of the concordance numbers (58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact) is not guaranteed.
- Different MANE selections still impact the final variant annotation even after transcript restriction.

## Evidence

- [methods] ClinVar filtering chain and final count: [evidence span withheld — anchor status: not_found]
- [intro] Normalization rationale for repeat regions: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation. In cases of nucleotide repeats, shifts to the left or right concerning the gene or transcript"
- [discussion] MNV phasing and normalization requirement: "Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF."
- [abstract] ClinVar benchmark set used for cross-tool comparison: "using 164,549 two-star variants from ClinVar"
- [discussion] Transcript standardization across tools: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
