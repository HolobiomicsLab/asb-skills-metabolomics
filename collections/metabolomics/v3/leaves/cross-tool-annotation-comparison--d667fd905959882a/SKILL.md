---
name: cross-tool-annotation-comparison
description: Use when the same variant set has been annotated by multiple annotation
  tools (ANNOVAR, SnpEff, VEP) and you need to quantify agreement on HGVSc, HGVSp,
  coding impact, or transcript accessions against a clinical reference set.
when_to_use_negative:
- Only one annotation tool's output is available — cross-tool concordance requires
  at least two independent annotators on the same variants
- The data are not variant annotations (e.g., microbial genomics/metabolomics linkage
  data such as NPLinker GNPS/AntiSMASH inputs) — this skill is specific to variant
  annotation comparison
- The ground-truth variants lack high review status; low-review ClinVar entries are
  unreliable references for concordance benchmarking
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: ANNOVAR
  role: Annotates the ClinVar benchmark VCF to produce HGVSc, HGVSp, and coding-impact
    calls for pairwise concordance comparison
- name: SnpEff
  role: Annotates the same variant set with the same reference/transcript setup; showed
    the highest HGVSc match (0.988) in the benchmark
- name: Variant Effect Predictor (VEP)
  role: Annotates the same variant set (with --per_gene and --coding_only to streamline
    transcripts); showed the highest HGVSp match (0.977) and the most available transcripts
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:06ce65df70e704851a74d6226522a9db2e901176c9a712811fc313e54e0d57f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - VEP
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# cross-tool-annotation-comparison

## Summary

Benchmarks HGVS nomenclature and predicted coding-impact concordance across ANNOVAR, SnpEff, and VEP against a ClinVar high-review-status ground truth, exposing annotation discrepancies that affect ACMG interpretation.

## When to use

Use when the same variant set has been annotated by multiple annotation tools (ANNOVAR, SnpEff, VEP) and you need to quantify agreement on HGVSc, HGVSp, coding impact, or transcript accessions against a clinical reference set.

## When NOT to use

- Only one annotation tool's output is available — cross-tool concordance requires at least two independent annotators on the same variants
- The data are not variant annotations (e.g., microbial genomics/metabolomics linkage data such as NPLinker GNPS/AntiSMASH inputs) — this skill is specific to variant annotation comparison
- The ground-truth variants lack high review status; low-review ClinVar entries are unreliable references for concordance benchmarking

## Inputs

- ClinVar two-star (high review status) variant benchmark set (164,549 variants, e.g., ClinVar VCF)
- VCF of the same variants annotated by ANNOVAR, SnpEff, and VEP with identical reference genome and transcript annotations
- Per-tool annotation outputs containing HGVSc, HGVSp, normalized coding-impact terms, and transcript accessions (NM/NR, ENST)

## Outputs

- concordance_table.csv with per-tool coding-impact agreement rates (targets: ANNOVAR 0.990, SnpEff 0.992, VEP 0.994) and LoF concordance (0.983/0.975/0.972)
- transcript_concordance.tsv with per-tool unique NM/NR and ENST transcript counts and cross-tool concordance percentages (22.68% RefSeq, 8.53% Ensembl)
- Cross-tool discrepancy list of variants whose annotations disagree with ClinVar (e.g., HGVSc 58.52%, HGVSp 84.04% agreement)
- Per-tool fractions of PLP variants downgraded by incorrect PVS1 interpretation (ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3%)

## How to apply

Start from a ClinVar two-star benchmark set (164,549 high-review-status variants) and annotate it with ANNOVAR, SnpEff, and VEP using identical reference genomes and transcript annotations across all three tools, per the article's standardization recommendation. Normalize tool-specific predicted coding-impact terms to a common Sequence Ontology vocabulary, prioritizing severe/high-impact classes when resolving ambiguous mappings, and restrict each tool's transcripts where needed (e.g., VEP --per_gene and --coding_only). Compare each tool's normalized HGVSc, HGVSp, and coding-impact call against the ClinVar-reported value per variant and compute per-tool agreement rates, then restrict to the Loss-of-Function category to compute per-tool LoF concordance over 31,640 LoF variants. Separately, extract NM/NR and ENST transcript identifier sets from each tool's outputs and compute cross-tool intersection concordance normalized by the shared transcript pool. Validate against the article's targets: overall coding-impact agreement 85.58%, per-tool LoF concordance ANNOVAR 0.983 / SnpEff 0.975 / VEP 0.972, and transcript concordance 22.68% (RefSeq) and 8.53% (Ensembl).

## Related tools

- **ANNOVAR** (Annotates the ClinVar benchmark VCF to produce HGVSc, HGVSp, and coding-impact calls for pairwise concordance comparison)
- **SnpEff** (Annotates the same variant set with the same reference/transcript setup; showed the highest HGVSc match (0.988) in the benchmark)
- **Variant Effect Predictor (VEP)** (Annotates the same variant set (with --per_gene and --coding_only to streamline transcripts); showed the highest HGVSp match (0.977) and the most available transcripts)

## Examples

```
variant_effect_predictor.pl --per_gene --coding_only -i clinvar_two_star.vcf --format vcf
```

## Evaluation signals

- Per-tool coding-impact agreement in concordance_table.csv must match reported values within rounding tolerance (ANNOVAR 0.990, SnpEff 0.992, VEP 0.994; three-tool level 85.58%)
- Per-tool LoF concordance over 31,640 LoF variants must equal 0.983 (ANNOVAR), 0.975 (SnpEff), 0.972 (VEP)
- Transcript concordance in transcript_concordance.tsv must be 22.68% (RefSeq, n=5,776) and 8.53% (Ensembl, n=5,845)
- Per-tool transcript counts must satisfy VEP most (25,381 RefSeq NM/NR; 67,695 Ensembl ENST) and SnpEff fewest (5,848/6,183)
- PVS1-driven PLP downgrade fractions should fall near ANNOVAR 55.9%, SnpEff 66.5%, VEP 67.3% when ACMG impact is evaluated

## Limitations

- Discrepancies with ClinVar persisted even after restricting transcripts to those selected by ClinVar
- Some HGVS syntax in ClinVar does not adhere to HGVS recommendations (Table S3 and Table S4), and inconsistencies exist between ClinVar web pages and its VCF, biasing the ground truth
- VCFs from different callers can produce inconsistent HGVS notation for nucleotide repeats shifted left or right; MNVs must be phased and normalized before comparison
- No software versions were reported for ANNOVAR, SnpEff, or VEP, limiting reproducibility of exact concordance values

## Evidence

- [abstract] Abstract: three-tool benchmark on ClinVar two-star variants: "This study evaluated the annotation concordance between three tools-- ANNOVAR, SnpEff, and Variant Effect Predictor (VEP)--using 164,549 two-star variants from ClinVar"
- [abstract] Abstract: overall concordance rates: "The analysis revealed variable concordance rates, with 58.52% agreement for HGVSc, 84.04% for HGVSp, and 85.58% for the coding impact."
- [discussion] Discussion: identical reference inputs across tools: "Ensure uniformity in input by recalibrating data formats to utilize identical reference genomes and transcript annotations across all tools."
- [discussion] Discussion: transcript restriction still leaves discrepancies: "Restricting transcripts to those selected by ClinVar improved consistency in HGVSc, HGVSp, and coding impact annotations; however, discrepancies with ClinVar persisted."
- [discussion] Discussion: prioritize high-impact variants: "Prioritize high-impact variants, as they are often more clinically relevant."
