---
name: vep-transcript-streamlining-parameterization
description: Use when annotating ClinVar or other pathogenic-variant VCFs with VEP
  and per-transcript HGVS output (HGVSc/HGVSp) diverges from a reference annotation
  (e.g. ClinVar strings), or when reducing transcript-multiplicity noise before cross-validating
  VEP output against ANNOVAR/SnpEff.
when_to_use_negative:
- Input variants are non-coding or intronic, since --coding_only suppresses output
  outside coding regions and the paper's concordance figures target coding impacts
  only
- Only one annotation tool is being used and no cross-tool comparison is planned;
  transcript-streamlining still requires a downstream concordance check against ClinVar
  to be meaningful
- Reference genome or transcript annotation set differs between tools, because non-uniform
  reference resources make the parameterization ineffective at resolving HGVS discrepancies
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: Variant Effect Predictor (VEP)
  role: Run with --per_gene and --coding_only to streamline transcript selection and
    produce coding-only HGVS annotations for cross-tool concordance benchmarking.
  repo: https://github.com/Ensembl/ensembl-vep
- name: ANNOVAR
  role: Companion annotator whose HGVSc/HGVSp outputs are cross-validated against
    VEP's transcript-streamlined output to expose per-transcript discrepancies.
- name: SnpEff
  role: Companion annotator whose HGVSc match fraction (0.988) on ClinVar-selected
    transcripts sets a baseline against which VEP's streamlining gains are judged.
- name: ClinVar
  role: Ground-truth HGVS strings (two-star, high-review-status subset) used to measure
    HGVSc/HGVSp concordance after VEP parameterization.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:823393958550181717e20c75dac1d107f17f99e5d36dbd3ccb0775e1f46c806d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ensembl/ensembl-vep
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# VEP transcript streamlining parameterization

## Summary

Apply VEP's --per_gene and --coding_only plug-in parameters to restrict the Variant Effect Predictor's transcript selection to one transcript per gene and coding-relevant variants only, reducing transcript-complexity-driven HGVS discrepancies. This paper's cross-tool benchmark against 164,549 two-star ClinVar variants identified non-uniform transcript selection as a principal source of HGVSc/HGVSp/coding-impact discordance, so constraining VEP's candidate transcripts is a concrete parameterization lever for improving concordance.

## When to use

Use when annotating ClinVar or other pathogenic-variant VCFs with VEP and per-transcript HGVS output (HGVSc/HGVSp) diverges from a reference annotation (e.g. ClinVar strings), or when reducing transcript-multiplicity noise before cross-validating VEP output against ANNOVAR/SnpEff.

## When NOT to use

- Input variants are non-coding or intronic, since --coding_only suppresses output outside coding regions and the paper's concordance figures target coding impacts only
- Only one annotation tool is being used and no cross-tool comparison is planned; transcript-streamlining still requires a downstream concordance check against ClinVar to be meaningful
- Reference genome or transcript annotation set differs between tools, because non-uniform reference resources make the parameterization ineffective at resolving HGVS discrepancies

## Inputs

- Standardized VCF of variants (e.g. the 164,549 two-star ClinVar high-review-status subset) with identical reference genome and transcript annotations
- Phase- and normalized multi-nucleotide variants (MNVs) in the VCF
- ClinVar HGVS reference strings (HGVSc and HGVSp) per variant for concordance checking

## Outputs

- VEP annotation table with per-gene, coding-only HGVSc, HGVSp, and coding-impact strings
- Per-variant match/mismatch tallies against ClinVar HGVS strings
- Concordance metrics (overall HGVSc 58.52%, HGVSp 84.04%, coding impact 85.58%; VEP HGVSp match fraction 0.991)

## How to apply

Run VEP with --per_gene and --coding_only so it selects a single relevant transcript per gene and emits only coding-impact annotations, 'streamlining relevant transcripts, reducing complexity and potential errors'. This directly addresses the paper's observation that 'different MANE selections still impact the final annotation' and that 'variability in transcript selection and HGVS nomenclature complicates variant retrieval and increases the risk of oversight during clinical genetic testing'. Pair the parameterization with an identical reference genome and transcript annotation set across ANNOVAR, SnpEff, and VEP, and phase/normalize multi-nucleotide variants in the VCF beforehand, since VCF caller differences shift nucleotide repeats left or right and alter HGVS notation. After annotation, string-match VEP's HGVSc/HGVSp against ClinVar-deposited HGVS per variant; on ClinVar-selected transcripts, VEP achieved the highest HGVSp match fraction (0.991), so transcript-streamlined runs should be re-checked against those baselines. Cross-validate outputs from multiple independent tools to identify residual discrepancies and record tool versions, since the paper notes 'no software versions are reported' as an unreproducibility gap.

## Related tools

- **Variant Effect Predictor (VEP)** (Run with --per_gene and --coding_only to streamline transcript selection and produce coding-only HGVS annotations for cross-tool concordance benchmarking.) — https://github.com/Ensembl/ensembl-vep
- **ANNOVAR** (Companion annotator whose HGVSc/HGVSp outputs are cross-validated against VEP's transcript-streamlined output to expose per-transcript discrepancies.)
- **SnpEff** (Companion annotator whose HGVSc match fraction (0.988) on ClinVar-selected transcripts sets a baseline against which VEP's streamlining gains are judged.)
- **ClinVar** (Ground-truth HGVS strings (two-star, high-review-status subset) used to measure HGVSc/HGVSp concordance after VEP parameterization.)

## Examples

```
vep --input_file clinvar_2star.vcf --output_file out.tsv --per_gene --coding_only --hgvs --cache --ref_genome GRCh38
```

## Evaluation signals

- Overall HGVSc agreement should equal 58.52% and HGVSp agreement should equal 84.04% within rounding tolerance when the full ClinVar two-star set is re-annotated on ClinVar-selected transcripts
- VEP's per-variant HGVSp match fraction should approach 0.991 on ClinVar-selected transcripts; if lower, transcript-streamlining was not effective
- Every variant in the output should have exactly one transcript per gene (the --per_gene contract) and only coding-impact entries (the --coding_only contract)
- The input VCF must be MNV-phased and normalized; residual left/right nucleotide-repeat shifts should surface as HGVS mismatches relative to ClinVar strings
- Tool versions and reference genome/transcript annotation IDs should be logged; the paper flags 'no software versions are reported' as a reproducibility failure to avoid

## Limitations

- Restricting transcripts to ClinVar-selected ones improved consistency, but discrepancies with ClinVar persisted; some ClinVar HGVS syntax does not adhere to HGVS recommendations (Tables S3 and S4)
- Inconsistencies exist between ClinVar web pages and its VCF, so the reference HGVS strings are themselves imperfect
- Loss-of-Function (LoF) category showed substantial discrepancies, and incorrect PVS1 interpretations downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives in clinical reports
- Even with MANE transcript implementation, different MANE selections still impact final variant annotation

## Evidence

- [discussion] using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors.: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
- [abstract] The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact.: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [intro] different MANE selections still impact the final annotation.: "different MANE selections still impact the final annotation."
- [discussion] Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF.: "Multi-nucleotide variants (MNVs) must be phased and normalized in the VCF."
- [discussion] Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted.: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
