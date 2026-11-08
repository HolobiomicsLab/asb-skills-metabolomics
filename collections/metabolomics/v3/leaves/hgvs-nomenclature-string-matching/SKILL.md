---
name: hgvs-nomenclature-string-matching
description: Use when you must quantify how concordant HGVSc/HGVSp nomenclature strings
  from variant annotation tools (ANNOVAR, SnpEff, VEP) are against a curated clinical
  reference such as ClinVar two-star variants.
when_to_use_negative:
- The variant set lacks a curated reference of deposited HGVS strings (e.g., no ClinVar
  high-confidence subset) to match against, making string matching undefined.
- The goal is functional impact or pathogenicity prediction (e.g., ACMG PVS1 assessment),
  not nomenclature concordance benchmarking.
- The VCF has not been normalized and MNVs phased, so HGVS differences reflect caller
  artifacts rather than true annotation discrepancies.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Annotates the standardized ClinVar VCF to produce HGVSc/HGVSp strings for
    matching (lowest HGVSc match fraction, 0.753)
- name: SnpEff
  role: Annotates the standardized ClinVar VCF to produce HGVSc/HGVSp strings for
    matching (highest HGVSc match fraction, 0.988)
- name: VEP
  role: Annotates the standardized ClinVar VCF with --per_gene and --coding_only to
    produce HGVSc/HGVSp strings for matching (highest HGVSp match fraction, 0.991)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:0f3ddec9a63ce79ae6f82ffdeffed4bf2ce342a30cc909a2eaf0766ab15cddd8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - VEP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hgvs-nomenclature-string-matching

## Summary

Benchmark skill that string-matches tool-reported HGVSc/HGVSp nomenclature from ANNOVAR, SnpEff, and VEP against ClinVar-deposited HGVS strings to quantify annotation concordance on high-review-status variants. Rationale: transcript selection and nomenclature variability across annotators complicates variant retrieval and risks clinical oversight.

## When to use

Use when you must quantify how concordant HGVSc/HGVSp nomenclature strings from variant annotation tools (ANNOVAR, SnpEff, VEP) are against a curated clinical reference such as ClinVar two-star variants.

## When NOT to use

- The variant set lacks a curated reference of deposited HGVS strings (e.g., no ClinVar high-confidence subset) to match against, making string matching undefined.
- The goal is functional impact or pathogenicity prediction (e.g., ACMG PVS1 assessment), not nomenclature concordance benchmarking.
- The VCF has not been normalized and MNVs phased, so HGVS differences reflect caller artifacts rather than true annotation discrepancies.

## Inputs

- ClinVar VCF filtered to 164,549 two-star (high review status) variants
- ANNOVAR, SnpEff, and VEP annotation outputs (parsed HGVSc, HGVSp, coding impact strings per variant)
- ClinVar-deposited HGVS nomenclature strings (HGVSc and HGVSp)
- Shared reference genome and transcript annotations used identically across all tools

## Outputs

- Per-variant match/mismatch tallies of tool-reported HGVSc and HGVSp vs. ClinVar HGVS strings
- Overall concordance rates (reference values: 58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact)
- Per-tool match fractions (reference values: SnpEff 0.988 HGVSc, VEP 0.991 HGVSp, ANNOVAR 0.753 HGVSc)

## How to apply

Download and filter the ClinVar VCF to the 164,549 two-star (high review status) variants, then run ANNOVAR, SnpEff, and VEP on the standardized VCF using identical reference genomes and transcript annotations across tools; for VEP, apply the --per_gene and --coding_only options to streamline relevant transcripts. Phase and normalize multi-nucleotide variants (MNVs) in the VCF before annotation, since nucleotide-repeat shifts left or right concerning the gene or transcript change HGVS notation. Parse each tool's output to extract HGVSc, HGVSp, and coding impact strings per variant, and string-match them against the ClinVar-deposited HGVS strings, tallying matches and mismatches. Compute overall concordance rates and per-tool match fractions, restricting to ClinVar-selected transcripts to test whether consistency improves. Interpret residual mismatches carefully: some ClinVar HGVS syntax itself does not adhere to HGVS recommendations.

## Related tools

- **ANNOVAR** (Annotates the standardized ClinVar VCF to produce HGVSc/HGVSp strings for matching (lowest HGVSc match fraction, 0.753))
- **SnpEff** (Annotates the standardized ClinVar VCF to produce HGVSc/HGVSp strings for matching (highest HGVSc match fraction, 0.988))
- **VEP** (Annotates the standardized ClinVar VCF with --per_gene and --coding_only to produce HGVSc/HGVSp strings for matching (highest HGVSp match fraction, 0.991))

## Evaluation signals

- Computed overall HGVSc agreement must equal 58.52% and HGVSp agreement must equal 84.04% within rounding tolerance.
- Computed coding-impact agreement must equal 85.58% within rounding tolerance.
- Per-tool HGVSc match fractions must reproduce SnpEff 0.988, VEP 0.977, ANNOVAR 0.753.
- Per-tool HGVSp match fractions must reproduce VEP 0.991 (highest for HGVSp).
- Match counts must be tallied over exactly the 164,549 two-star ClinVar variants with identical reference genome and transcript annotations across all three tools.

## Limitations

- Discrepancies with ClinVar persisted even after restricting transcripts to those selected by ClinVar, so perfect concordance is not achievable.
- Some HGVS syntax deposited in ClinVar does not adhere to HGVS recommendations (documented in Table S3 and Table S4), inflating apparent tool mismatch.
- VCFs generated by different callers yield inconsistent HGVS notation for nucleotide repeats shifted left or right relative to the gene or transcript, contaminating string matches.
- No software versions are reported for the compared annotation tools, limiting reproducibility of exact match fractions.

## Evidence

- [abstract] overall concordance rates: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [abstract] per-tool match fractions: "SnpEff showed the highest match for HGVSc (0.988), while VEP bettered for HGVSp (0.977)."
- [discussion] persistence of discrepancies: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
- [discussion] ClinVar HGVS non-compliance: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [intro] caller-dependent HGVS shifts: "the Variant Call Format (VCF) generated by different callers may result in inconsistent HGVS notation."
