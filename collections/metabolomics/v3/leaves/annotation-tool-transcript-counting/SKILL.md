---
name: annotation-tool-transcript-counting
description: Use when comparing variant annotation tools (ANNOVAR, SnpEff, VEP) and
  you need to measure how many NM/NR or ENST transcripts each tool exposes on RefSeq/Ensembl
  and how much their transcript-accession sets overlap, to explain downstream annotation
  discordance.
when_to_use_negative:
- You already have per-variant HGVS or coding-impact outputs and need variant-level
  concordance rates (58.52%/84.04%/85.58%) — use a variant-annotation comparison skill
  instead, not transcript-set counting
- The tools have been configured to use identical, restricted transcript sets (e.g.,
  ClinVar-selected or MANE transcripts), so the transcript-availability gap you intend
  to measure has been eliminated by design
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0085
tools:
- name: Variant Effect Predictor (VEP)
  role: Source of the largest transcript identifier set (25,381 RefSeq NM/NR; 67,695
    Ensembl ENST) used as one arm of the concordance intersection
- name: SnpEff
  role: Source of the smallest transcript identifier set (5,848 NM/NR; 6,183 ENST),
    defining the lower bound of the intersection
- name: ANNOVAR
  role: Third annotation tool whose transcript availability completes the three-way
    pairwise intersection on both RefSeq and Ensembl reference sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:e5a43aa1ca9265d2be83fac286a370fa2725f1c60ac4874ba4f3f145dad3d652
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# annotation-tool-transcript-counting

## Summary

Quantifies the transcript availability (NM/NR and ENST) of ANNOVAR, SnpEff, and VEP on RefSeq and Ensembl reference sets and computes the cross-tool transcript-accession concordance (22.68% RefSeq, 8.53% Ensembl), exposing the low overlap that drives downstream HGVS/impact discrepancies.

## When to use

Use when comparing variant annotation tools (ANNOVAR, SnpEff, VEP) and you need to measure how many NM/NR or ENST transcripts each tool exposes on RefSeq/Ensembl and how much their transcript-accession sets overlap, to explain downstream annotation discordance.

## When NOT to use

- You already have per-variant HGVS or coding-impact outputs and need variant-level concordance rates (58.52%/84.04%/85.58%) — use a variant-annotation comparison skill instead, not transcript-set counting
- The tools have been configured to use identical, restricted transcript sets (e.g., ClinVar-selected or MANE transcripts), so the transcript-availability gap you intend to measure has been eliminated by design

## Inputs

- Transcript identifier sets (NM/NR accessions) extracted from ANNOVAR, SnpEff, and VEP annotation outputs on RefSeq transcripts
- Transcript identifier sets (ENST accessions) extracted from ANNOVAR, SnpEff, and VEP annotation outputs on Ensembl transcripts

## Outputs

- Per-tool unique transcript counts (e.g., VEP: 25,381 NM/NR and 67,695 ENST; SnpEff: 5,848 NM/NR and 6,183 ENST)
- Cross-tool transcript concordance percentages per reference database, written to transcript_concordance.tsv (RefSeq: 22.68%, n=5,776; Ensembl: 8.53%, n=5,845)

## How to apply

Extract the transcript identifiers each tool actually uses for annotation from its output files — VEP yields the largest set (25,381 RefSeq NM/NR; 67,695 Ensembl ENST), ANNOVAR is intermediate, and SnpEff the smallest (5,848 NM/NR; 6,183 ENST). For each tool and each reference database, count unique NM/NR (RefSeq) and ENST (Ensembl) accessions, then intersect the per-tool identifier sets pairwise to obtain cross-tool concordance, normalizing as a percentage of the shared transcript pool per database. Write per-tool counts and concordance percentages to transcript_concordance.tsv, separating the RefSeq and Ensembl analyses because the two reference databases yield very different overlap (22.68% vs 8.53%, n=5,776 and n=5,845 respectively). The rationale is that low transcript-set overlap is the root cause of HGVS nomenclature (58.52% concordance) and coding-impact (85.58% concordance) disagreements across the three tools, so measuring the intersection quantifies the pre-annotation alignment bottleneck before any variant-level comparison.

## Related tools

- **Variant Effect Predictor (VEP)** (Source of the largest transcript identifier set (25,381 RefSeq NM/NR; 67,695 Ensembl ENST) used as one arm of the concordance intersection)
- **SnpEff** (Source of the smallest transcript identifier set (5,848 NM/NR; 6,183 ENST), defining the lower bound of the intersection)
- **ANNOVAR** (Third annotation tool whose transcript availability completes the three-way pairwise intersection on both RefSeq and Ensembl reference sets)

## Evaluation signals

- Per-tool transcript counts match published values exactly: VEP 25,381 NM/NR and 67,695 ENST; SnpEff 5,848 NM/NR and 6,183 ENST — deviations beyond rounding indicate an error in transcript-set extraction
- Cross-tool concordance reproduces 22.68% (n=5,776) on RefSeq and 8.53% (n=5,845) on Ensembl when intersecting all three tools per database
- transcript_concordance.tsv contains both per-tool counts and normalized concordance percentages, separated by reference database (RefSeq vs Ensembl)
- RefSeq concordance is substantially higher than Ensembl concordance (22.68% vs 8.53%), reflecting the smaller, curated NM/NR pool versus the larger ENST set — a large inversion of this ordering signals a wrong transcript-extraction step
- Identifiers are correctly partitioned by accession prefix (NM/NR for RefSeq, ENST for Ensembl); mixing prefixes within a database column invalidates the intersection

## Limitations

- Cross-tool concordance is low even on the curated RefSeq set (22.68%), so transcript-count overlap alone cannot guarantee consistent variant annotation — downstream HGVS and coding-impact concordance (58.52%/85.58%) must be assessed separately
- Restricting transcripts to those selected by ClinVar improves HGVSc, HGVSp, and coding-impact consistency but discrepancies with ClinVar persist, indicating transcript-set alignment is necessary but not sufficient
- Different MANE selections still impact final variant annotation, so a high transcript-count overlap does not eliminate annotation variability if MANE or isoform selection differs

## Evidence

- [full_text] Cross-tool concordance of transcript accessions: [evidence span withheld — anchor status: not_found]
- [intro] Transcript-set availability determines annotation divergence: "Annotation tools like ANNOVAR14, SnpEff15, and VEP16 utilize RNA transcripts to annotate variants by using overlapping genomic characteristics."
- [abstract] Standardizing transcript sets is the remedy: "To enhance the reliability of genetic variant interpretation in clinical practice, standardizing transcript sets and systematically cross-validating results across multiple annotation tools is"
