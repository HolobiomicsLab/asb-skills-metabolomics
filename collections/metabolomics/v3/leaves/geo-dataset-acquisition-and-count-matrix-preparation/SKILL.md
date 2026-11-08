---
name: geo-dataset-acquisition-and-count-matrix-preparation
description: Use when starting a reproducibility analysis of RNA-seq replicate experiments
  and the raw multi-lab SEQC data must be retrieved from GEO and converted into per-experiment
  transcript count vectors/rank lists before computing the maximum rank statistic.
when_to_use_negative:
- The count matrix and the six SEQC comparisons are already prepared and the task
  is MaRR estimation itself (k-hat, mFDR-hat, pi1-hat), not data acquisition
- The experiment has no replicate structure or the data are not transcript-level read
  counts from RNA-seq, since the comparison formation (biological replicate pairs,
  per-lab count sums) is specific to the SEQC design
- The analysis requires a model-based copula mixture estimate; this skill only stages
  data for the nonparametric rank-based MaRR procedure
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3168
tools:
- name: Python (NumPy/pandas)
  role: Loads the GSE47774 read counts, sums counts over replicates per lab for the
    between-lab comparisons, and assigns read-count ranks with random tie-breaking
    over 43,919 transcripts
- name: maximum rank reproducibility (MaRR) procedure
  role: Downstream consumer of the prepared rank lists; its requirement that each
    comparison be a paired rank list over the same transcript set dictates the matrix
    preparation choices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:3c083e47d7509baa687fe8f8807b6b8a68a3bf974e9d87b43d53738238eb0873
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  - Python (NumPy/pandas)
  - maximum rank reproducibility (MaRR) procedure
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# geo-dataset-acquisition-and-count-matrix-preparation

## Summary

Acquires the SEQC/MAQC-III Universal Human Reference RNA-seq dataset (GEO accession GSE47774) and prepares the ranked count-matrix inputs — 43,919 transcripts across within-lab biological replicate pairs (Mayo Clinic, Penn State, NYU Medical Center) and three between-lab same-platform comparisons — required by the MaRR reproducibility analysis.

## When to use

Use when starting a reproducibility analysis of RNA-seq replicate experiments and the raw multi-lab SEQC data must be retrieved from GEO and converted into per-experiment transcript count vectors/rank lists before computing the maximum rank statistic.

## When NOT to use

- The count matrix and the six SEQC comparisons are already prepared and the task is MaRR estimation itself (k-hat, mFDR-hat, pi1-hat), not data acquisition
- The experiment has no replicate structure or the data are not transcript-level read counts from RNA-seq, since the comparison formation (biological replicate pairs, per-lab count sums) is specific to the SEQC design
- The analysis requires a model-based copula mixture estimate; this skill only stages data for the nonparametric rank-based MaRR procedure

## Inputs

- SEQC/MAQC-III GEO accession GSE47774 (13 laboratories, Illumina HiSeq 2000 / SOLiD 5500 / Roche 454 GS FLX)
- Per-transcript read counts mapped to the common set of 43,919 transcripts
- Laboratory and replicate metadata identifying Mayo Clinic, Penn State, and NYU Medical Center within-lab replicate pairs and per-lab grouping for between-lab comparisons

## Outputs

- Transcript-by-experiment read-count matrix over 43,919 transcripts (NumPy/pandas)
- Six replicate comparison count vectors: three within-lab pairs and three between-lab same-platform comparisons (per-lab summed counts)
- Per-experiment rank vectors (rank 1 = most reads, ties randomly assigned) ready for maximum rank statistic computation

## How to apply

Download the SEQC/MAQC-III Universal Human Reference RNA sequencing data under GEO accession GSE47774, comprising 13 laboratories sequencing the same samples on Illumina HiSeq 2000, Life Technologies SOLiD 5500, and Roche 454 GS FLX platforms, with all reads mapped to the same set of 43,919 transcripts. Assemble a transcript-by-experiment read-count matrix, keeping only the common 43,919 transcripts so every experiment is directly comparable. Form six replicate comparisons: within-lab biological replicate pairs from Mayo Clinic, Penn State, and NYU Medical Center, plus three between-lab same-platform comparisons obtained by summing read counts over all replicates per lab (Illumina, Life Technologies, Roche 454). Within each experiment, rank transcripts by read count with rank 1 for the most-read transcript, breaking ranking ties by random assignment so that the maximum rank statistic Mg = max{Rgx, Rgy} is well defined downstream.

## Related tools

- **Python (NumPy/pandas)** (Loads the GSE47774 read counts, sums counts over replicates per lab for the between-lab comparisons, and assigns read-count ranks with random tie-breaking over 43,919 transcripts)
- **maximum rank reproducibility (MaRR) procedure** (Downstream consumer of the prepared rank lists; its requirement that each comparison be a paired rank list over the same transcript set dictates the matrix preparation choices)

## Examples

```
counts = pd.read_csv('seqc_gse47774_counts.csv', index_col=0); ranks = counts.loc[np.random.permutation(counts.index)].rank(ascending=False, method='first')
```

## Evaluation signals

- The prepared matrix contains exactly 43,919 transcripts for every experiment, matching the common mapping set
- Six comparisons are formed: within-lab pairs from Mayo Clinic, Penn State, and NYU Medical Center plus three same-platform between-lab comparisons (Illumina, Life Technologies, Roche 454)
- Between-lab comparison counts equal the sum of read counts over all replicates within each lab
- Every rank vector is a permutation of 1..43,919 with rank 1 assigned to the transcript with the most reads and no duplicate ranks within an experiment
- Downstream MaRR estimates on the prepared inputs reproduce pi1-hat = 0.851, 0.847, 0.900, 0.898 for comparisons 1, 2, 4, 5 and the Table 5 / Figure 9 counts of reproducible transcripts at alpha = 0.01

## Limitations

- Ranking ties are broken by random assignment, so prepared rank lists and downstream MaRR results can vary between runs unless a fixed random seed is used
- Between-lab comparisons rely on summing read counts over all replicates per lab, which is specific to the SEQC design and may not generalize to other replicate structures
- Only transcripts mapped to the common set of 43,919 transcripts can be used; experiments not mapped to this shared transcript set are not directly comparable

## Evidence

- [methods] SEQC workflow step 1: [evidence span withheld — anchor status: not_found]
- [methods] SEQC workflow step 2-3: [evidence span withheld — anchor status: not_found]
