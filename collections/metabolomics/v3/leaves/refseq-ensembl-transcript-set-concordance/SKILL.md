---
name: refseq-ensembl-transcript-set-concordance
description: 'Decision support for choosing annotation tools or transcript sources:
  when you must know which transcripts ANNOVAR, SnpEff, or VEP actually cover on RefSeq
  vs Ensembl, and how much their transcript identifier sets overlap.'
when_to_use_negative:
- You need variant-level annotation agreement (HGVSc/HGVSp/coding impact concordance)
  rather than transcript-set overlap — that is a different comparison performed on
  annotated variants, not identifier sets.
- Your annotation tools were run against the same standardized transcript reference,
  so their identifier sets are expected to be identical by construction and the concordance
  statistic carries no signal.
- Input annotation outputs do not record transcript accessions at all (e.g. gene-level-only
  annotations), making identifier-set intersection impossible.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0085
tools:
- name: Variant Effect Predictor (VEP)
  role: One of the three annotation tools whose emitted RefSeq NM/NR and Ensembl ENST
    transcript sets are extracted and intersected; had the largest transcript pool
    in the reference study.
- name: SnpEff
  role: Source of the smallest transcript identifier set to be compared; its annotations
    supply the NM/NR and ENST accessions used in the concordance intersection.
- name: ANNOVAR
  role: Third annotation tool contributing transcript accessions to the pairwise concordance
    calculation across RefSeq and Ensembl databases.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.11.617756
    title: 'Toward Automatic Variant Interpretation: Discordant Genetic Interpretation
      Across Variant Annotations for ClinVar Pathogenic Variants'
  source_content_hash: sha256:dcdf9344c1a88ea179f5412f9d185ee83b3a3c3cdad6de534570a65f14f9dd9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - SnpEff
  - Variant Effect Predictor (VEP)
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# refseq-ensembl-transcript-set-concordance

## Summary

Quantifies how many NM/NR (RefSeq) and ENST (Ensembl) transcript accessions each variant annotation tool exposes, then computes pairwise concordance across ANNOVAR, SnpEff, and VEP for the same reference database.

## When to use

Decision support for choosing annotation tools or transcript sources: when you must know which transcripts ANNOVAR, SnpEff, or VEP actually cover on RefSeq vs Ensembl, and how much their transcript identifier sets overlap.

## When NOT to use

- You need variant-level annotation agreement (HGVSc/HGVSp/coding impact concordance) rather than transcript-set overlap — that is a different comparison performed on annotated variants, not identifier sets.
- Your annotation tools were run against the same standardized transcript reference, so their identifier sets are expected to be identical by construction and the concordance statistic carries no signal.
- Input annotation outputs do not record transcript accessions at all (e.g. gene-level-only annotations), making identifier-set intersection impossible.

## Inputs

- Annotation outputs from ANNOVAR, SnpEff, and VEP containing transcript identifiers for RefSeq (NM/NR) and Ensembl (ENST) transcript sets

## Outputs

- transcript_concordance.tsv with per-tool unique transcript counts (RefSeq NM/NR and Ensembl ENST) and pairwise cross-tool concordance percentages per reference database

## How to apply

Extract the transcript identifiers (NM/NR on RefSeq, ENST on Ensembl) that each of VEP, SnpEff, and ANNOVAR emits from their annotation outputs. Count unique accessions per tool per database — in the reference study VEP exposed the most (25,381 RefSeq NM/NR; 67,695 Ensembl ENST) while SnpEff exposed the fewest (5,848 RefSeq; 6,183 Ensembl). Intersect the identifier sets pairwise within each reference database to measure cross-tool transcript concordance, normalizing as a percentage of the shared pool: the study found 22.68% (n=5,776) concordance on RefSeq and 8.53% (n=5,845) on Ensembl. Write per-tool counts and concordance percentages to a TSV (e.g. transcript_concordance.tsv) so downstream tool-selection or transcript-standardization decisions are traceable. Rationale: transcript-availability asymmetry directly drives divergent HGVS nomenclature and predicted coding impacts between tools, so low concordance flags the need to standardize transcript sets before cross-validating annotations.

## Related tools

- **Variant Effect Predictor (VEP)** (One of the three annotation tools whose emitted RefSeq NM/NR and Ensembl ENST transcript sets are extracted and intersected; had the largest transcript pool in the reference study.)
- **SnpEff** (Source of the smallest transcript identifier set to be compared; its annotations supply the NM/NR and ENST accessions used in the concordance intersection.)
- **ANNOVAR** (Third annotation tool contributing transcript accessions to the pairwise concordance calculation across RefSeq and Ensembl databases.)

## Evaluation signals

- Per-tool RefSeq counts match the reference bounds: VEP ~25,381 NM/NR, SnpEff ~5,848, Ensembl ENST: VEP ~67,695, SnpEff ~6,183.
- Cross-tool RefSeq concordance lands near 22.68% (n=5,776) and Ensembl concordance near 8.53% (n=5,845); deviations beyond rounding indicate an error in transcript-set extraction.
- Concordance is computed per reference database (RefSeq vs Ensembl separately) — mixing NM/NR and ENST identifiers in one intersection invalidates the metric.
- transcript_concordance.tsv exists and contains both per-tool counts and normalized concordance percentages, not raw identifier dumps.
- Identifier sets are drawn from actual annotation outputs of all three tools, not from any official database release file, so the numbers reflect tool-internal transcript availability.

## Limitations

- Concordance reflects only what each tool's annotation output exposes; discrepancies may stem from tool-internal transcript filtering, not database content differences.
- The metric is set-level: it does not reveal which specific transcripts are missing or whether missing transcripts cause clinical annotation divergence for particular variants.
- No software versions of the compared annotation tools were reported in the study, so results may vary across releases of ANNOVAR, SnpEff, and VEP.

## Evidence

- [results] cross-tool concordance definition: [evidence span withheld — anchor status: not_found]
- [results] per-tool transcript counts: [evidence span withheld — anchor status: not_found]
- [methods] refseq/ensembl transcript identifiers: [evidence span withheld — anchor status: not_found]
- [methods] output file: [evidence span withheld — anchor status: not_found]
- [results] validation bounds: [evidence span withheld — anchor status: not_found]
- [intro] annotation tools using transcripts: "Annotation tools like ANNOVAR14, SnpEff15, and VEP16 utilize RNA transcripts to annotate variants by using overlapping genomic characteristics"
