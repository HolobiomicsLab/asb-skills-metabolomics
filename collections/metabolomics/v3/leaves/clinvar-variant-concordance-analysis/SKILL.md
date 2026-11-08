---
name: clinvar-variant-concordance-analysis
description: Use when evaluating whether a variant annotation pipeline (ANNOVAR, SnpEff,
  VEP) produces HGVS nomenclature and coding-impact calls consistent with ClinVar
  high-review-status (two-star) variants, and when quantifying which mismatch classes
  (NA, position/variant-type, non-preferred syntax) dominate.
when_to_use_negative:
- When the goal is metabolomics-genomics linkage or natural product mining — the README
  context (NPLinker, GNPS/AntiSMASH data) is unrelated to variant annotation concordance.
- When the variant set is not ClinVar high-review-status (two-star); applying this
  skill to lower-review-status or non-ClinVar variants will not reproduce the reported
  concordance thresholds.
- When the three tools cannot be run on identical reference genomes and transcript
  annotations; tool-version or transcript-set divergence inflates apparent mismatches
  beyond what the method can diagnose.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0089
tools:
- name: ANNOVAR
  role: Annotation tool whose HGVSc/HGVSp output and coding-impact calls are compared
    against ClinVar reference annotations; also source of dominant NA-mismatch category
    (79.19% of HGVSc inconsistencies).
- name: SnpEff
  role: Annotation tool benchmarked for HGVSc/HGVSp and coding impact; achieved highest
    HGVSc match (0.988) and 78.00% position/variant-type mismatch class for HGVSc.
- name: Variant Effect Predictor (VEP)
  role: Annotation tool benchmarked for HGVSc/HGVSp and coding impact; achieved highest
    HGVSp match (0.977) and 95.47% position/variant-type mismatch class for HGVSc;
    supports --per_gene and --coding_only flags to streamline relevant transcripts.
- name: ClinVar
  role: Reference clinical database supplying the 164,549 two-star variant benchmark
    set with reference HGVSc/HGVSp strings and reported coding impacts.
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:baed3da2df3daceff56fd0aec7d17562976138e733fa020428cc5e96c7fa19dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - ClinVar
  - SnpEff
  - VEP
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# clinvar-variant-concordance-analysis

## Summary

Benchmark ANNOVAR, SnpEff, and VEP HGVSc/HGVSp nomenclature and coding-impact calls against ClinVar two-star variants (164,549 high review-status records), computing per-tool concordance (e.g. 58.52% HGVSc, 84.04% HGVSp, 85.58% coding impact) and diagnosing mismatch categories such as missing NA annotations, position/variant-type differences, and non-preferred HGVS syntax.

## When to use

Use when evaluating whether a variant annotation pipeline (ANNOVAR, SnpEff, VEP) produces HGVS nomenclature and coding-impact calls consistent with ClinVar high-review-status (two-star) variants, and when quantifying which mismatch classes (NA, position/variant-type, non-preferred syntax) dominate per tool.

## When NOT to use

- When the goal is metabolomics-genomics linkage or natural product mining — the README context (NPLinker, GNPS/AntiSMASH data) is unrelated to variant annotation concordance.
- When the variant set is not ClinVar high-review-status (two-star); applying this skill to lower-review-status or non-ClinVar variants will not reproduce the reported concordance thresholds.
- When the three tools cannot be run on identical reference genomes and transcript annotations; tool-version or transcript-set divergence inflates apparent mismatches beyond what the method can diagnose.

## Inputs

- ClinVar two-star (high review status) variant set, 164,549 records with reference HGVSc/HGVSp strings and reported coding impacts
- Tool-generated HGVSc and HGVSp annotation output from ANNOVAR
- Tool-generated HGVSc and HGVSp annotation output from SnpEff
- Tool-generated HGVSc and HGVSp annotation output from VEP (optionally with --per_gene and --coding_only to streamline relevant transcripts)
- Normalized coding-impact functional categories per tool (SO vocabulary) including the Loss-of-Function category over 31,640 variants

## Outputs

- concordance_table.csv with per-tool overall coding-impact agreement and LoF category concordance values
- mismatch_categories.csv with per-tool, per-field (HGVSc/HGVSp), per-category (NA, position/variant-type, non-preferred/unknown) mismatch percentages
- Per-tool overall HGVS concordance rates (58.52% HGVSc, 84.04% HGVSp) and best-per-tool match values (SnpEff HGVSc 0.988, VEP HGVSp 0.977)

## How to apply

Build a benchmark set of 164,549 two-star ClinVar variants and annotate the identical variant set with ANNOVAR, SnpEff, and VEP using identical reference genomes and transcript annotations across all three tools. Normalize each tool's predicted coding-impact terms to a common Sequence Ontology vocabulary so tool-specific labels become comparable, prioritizing severe/high-impact classes when resolving ambiguous mappings. For each tool and each HGVS field (HGVSc, HGVSp), compare tool output against the ClinVar reference string and classify each record as match, missing (NA), position/variant-type difference, or non-preferred/unknown HGVS syntax, then compute per-tool per-field category percentages that sum with the match rate to 100%. Compare normalized coding-impact calls against ClinVar-reported impact per variant to obtain per-tool overall agreement (target ANNOVAR 0.990, SnpEff 0.992, VEP 0.994) and restrict to the Loss-of-Function category to compute LoF concordance (target ANNOVAR 0.983, SnpEff 0.975, VEP 0.972). Note that SnpEff achieves the highest HGVSc match (0.988) while VEP leads for HGVSp (0.977); restricting transcripts to ClinVar-selected ones improves consistency but discrepancies persist. Write per-tool agreement values to concordance_table.csv and mismatch-category percentages to mismatch_categories.csv.

## Related tools

- **ANNOVAR** (Annotation tool whose HGVSc/HGVSp output and coding-impact calls are compared against ClinVar reference annotations; also source of dominant NA-mismatch category (79.19% of HGVSc inconsistencies).)
- **SnpEff** (Annotation tool benchmarked for HGVSc/HGVSp and coding impact; achieved highest HGVSc match (0.988) and 78.00% position/variant-type mismatch class for HGVSc.)
- **Variant Effect Predictor (VEP)** (Annotation tool benchmarked for HGVSc/HGVSp and coding impact; achieved highest HGVSp match (0.977) and 95.47% position/variant-type mismatch class for HGVSc; supports --per_gene and --coding_only flags to streamline relevant transcripts.)
- **ClinVar** (Reference clinical database supplying the 164,549 two-star variant benchmark set with reference HGVSc/HGVSp strings and reported coding impacts.)

## Examples

```
vep --input_file clinvar_2star.vcf --output_file out.vcf --per_gene --coding_only --offline --cache; python concordance.py --tool vep --ref clinvar_hgvs.tsv --out mismatch_categories.csv
```

## Evaluation signals

- Reproduced per-tool coding-impact agreement in concordance_table.csv matches reported values within rounding tolerance: ANNOVAR 0.990, SnpEff 0.992, VEP 0.994; LoF concordance ANNOVAR 0.983, SnpEff 0.975, VEP 0.972 over 31,640 LoF variants.
- Reproduced overall HGVS match rates are consistent with the reported 58.52% HGVSc and 84.04% HGVSp concordance.
- Per-tool per-field mismatch-category percentages in mismatch_categories.csv reproduce the reported values: ANNOVAR HGVSc 79.19% NA (remaining 19.67% position/variant-type, 1.14% unknown), SnpEff 78.00% position/variant-type, VEP HGVSc 95.47% position/variant-type.
- For each tool and each HGVS field, category percentages plus match rate sum to 100%.
- Restricting transcripts to ClinVar-selected ones should improve consistency in HGVSc, HGVSp, and coding impact annotations, though residual discrepancies with ClinVar persist.

## Limitations

- Some HGVS syntax in ClinVar itself does not adhere to HGVS recommendations (Table S3 and Table S4), contributing to non-preferred/unknown mismatch class.
- Inconsistencies exist between ClinVar web pages and its VCF, which can inflate apparent tool-ClinVar mismatches.
- No software versions are reported for the compared annotation tools (ANNOVAR, SnpEff, VEP), complicating exact reproduction.
- Discrepancies with ClinVar persisted even after restricting transcripts to ClinVar-selected ones.
- Variability in transcript selection and HGVS nomenclature (e.g. left/right shifts in nucleotide repeats from different VCF callers, different MANE selections) complicates variant retrieval and increases risk of oversight during clinical genetic testing.
- Incorrect PVS1 interpretations affected final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives of clinically relevant variants.

## Evidence

- [abstract] Two-star ClinVar variants used as benchmark: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [abstract] Overall concordance rates reported: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [abstract] Best-per-tool HGVS match values: "SnpEff showed the highest match for HGVSc (0.988), while VEP bettered for HGVSp (0.977)."
- [abstract] PVS1 mismatch causing LoF downgrade: "Incorrect PVS1 interpretations affected the final pathogenicity and downgraded PLP variants (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%), risking false negatives of clinically relevant variants in"
- [discussion] Standardization workflow step: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
- [discussion] ClinVar syntax non-compliance noted: "Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (see Table S3 and Table S4)"
- [discussion] Transcript restriction still leaves discrepancies: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
- [discussion] VEP flags to streamline transcripts: "using plug-in function --per_gene and --coding_only in VEP can streamline relevant transcripts, reducing complexity and potential errors."
- [results] Mismatch-category breakdown method: [evidence span withheld — anchor status: not_found]
- [results] Reproducible per-tool concordance values: [evidence span withheld — anchor status: not_found]
