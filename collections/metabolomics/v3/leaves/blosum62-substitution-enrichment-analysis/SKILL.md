---
name: blosum62-substitution-enrichment-analysis
description: Use when you have de novo peptide assignments (e.g. from Casanovo on
  clustered unidentified MS/MS spectra) and want to test whether discordant or mismatched
  assignments are enriched for variants explainable by single-nucleotide substitutions.
when_to_use_negative:
- Peptides were identified by standard database searching with no de novo predictions,
  so there are no mismatched assignments to test for substitution consistency
- The input spectra have not been clustered (e.g. no GLEAMS cluster identifiers),
  so per-spectrum predictions cannot be aggregated into cluster-level consensus peptides
  before mismatch testing
- Predicted and reference peptides differ by multiple amino acids, which cannot be
  attributed to a single-nucleotide substitution and fall outside this single-mismatch
  analysis
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Casanovo
  role: Produces the per-spectrum de novo peptide predictions (beam search decoding,
    mzTab export) whose single-mismatch substitutions are tested for single-nucleotide
    consistency and BLOSUM62 enrichment
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:3babe54c9e2a09deb9f12a8f5997dc5711f4e71c33e4a58c0ee49f0c631bc238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# BLOSUM62 substitution enrichment analysis

## Summary

Tests whether amino-acid mismatches between de novo peptide predictions (from Casanovo) and database peptide matches are consistent with single-nucleotide substitutions, and quantifies this with a BLOSUM62 enrichment statistic — the analysis that showed 83.4% of Casanovo's predicted substitutions on GLEAMS dark-proteome clusters correspond to potential single-nucleotide substitutions versus 38.6% expected.

## When to use

Use when you have de novo peptide assignments (e.g. from Casanovo on clustered unidentified MS/MS spectra) and want to test whether discordant or mismatched assignments are enriched for variants explainable by single-nucleotide substitutions.

## When NOT to use

- Peptides were identified by standard database searching with no de novo predictions, so there are no mismatched assignments to test for substitution consistency
- The input spectra have not been clustered (e.g. no GLEAMS cluster identifiers), so per-spectrum predictions cannot be aggregated into cluster-level consensus peptides before mismatch testing
- Predicted and reference peptides differ by multiple amino acids, which cannot be attributed to a single-nucleotide substitution and fall outside this single-mismatch analysis

## Inputs

- Per-spectrum de novo peptide predictions and confidence scores from Casanovo (mzTab format), grouped by GLEAMS cluster identifier
- Cluster-level consensus peptides derived by plurality vote from per-spectrum predictions
- Predicted proteomes (e.g. human proteome) for mapping consensus peptides and identifying single-mismatch assignments

## Outputs

- Fraction of predicted substitutions consistent with single-nucleotide substitutions (observed vs expected, e.g. 83.4% vs 38.6%)
- BLOSUM62 enrichment statistic for mismatched peptide assignments
- Cluster-assignment counts and assigned-spectra totals written to gleams_cluster_assignments_summary.tsv

## How to apply

Start from per-spectrum Casanovo predictions (beam search decoding, exported to mzTab) grouped by GLEAMS cluster identifier, with a cluster-level consensus peptide derived by plurality vote. For clusters with discordant per-spectrum assignments, align the predicted peptides against their database-matched reference peptides and test whether each amino-acid mismatch is consistent with exactly one nucleotide substitution (single-mismatch assignments). Compute the fraction of predicted substitutions consistent with a single-nucleotide substitution and compare it against the expectation under a background substitution model; in the GLEAMS dark-proteome analysis this yielded 83.4% observed versus 38.6% expected. Additionally compute the BLOSUM62 enrichment of the mismatched assignments — high-scoring BLOSUM62 substitutions indicate conservative, biochemically plausible swaps — and write the cluster-assignment counts, assigned-spectra totals, and enrichment statistics to a summary table (gleams_cluster_assignments_summary.tsv).

## Related tools

- **Casanovo** (Produces the per-spectrum de novo peptide predictions (beam search decoding, mzTab export) whose single-mismatch substitutions are tested for single-nucleotide consistency and BLOSUM62 enrichment) — https://github.com/Noble-Lab/casanovo

## Evaluation signals

- Observed fraction of single-nucleotide-consistent substitutions substantially exceeds the expected background fraction (paper: 83.4% observed vs 38.6% expected)
- Each tested mismatch differs from the reference peptide at exactly one amino-acid position
- BLOSUM62 enrichment of mismatched assignments is positive/quantifiable and reproducible across the clustered spectra
- Cluster-level counts are consistent: 52,523 assigned clusters (9%) and 196,724 assigned spectra out of 3.4 million clustered unidentified spectra
- Summary table gleams_cluster_assignments_summary.tsv contains all three reported quantities: cluster counts, spectra totals, and enrichment statistics

## Limitations

- The analysis rests on de novo predictions without a rigorous confidence-estimation method — identified in the paper as one of the key bottlenecks in the field
- Single-mismatch consistency is an indirect signal for genetic variants; the paper frames it as enrichment ('potential single-nucleotide substitutions') rather than definitive variant calling
- Results depend on the quality of the GLEAMS clustering and the plurality-vote consensus; discordant clusters drive the mismatch statistics

## Evidence

- [results] single-nucleotide consistency test and BLOSUM62 enrichment are core workflow steps: [evidence span withheld — anchor status: not_found]
- [results] quantitative finding on substitution enrichment: [evidence span withheld — anchor status: not_found]
- [readme] README grounds the de novo prediction source for the mismatches: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset"
- [discussion] article discussion flags the confidence-estimation bottleneck underlying this analysis: "one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing."
