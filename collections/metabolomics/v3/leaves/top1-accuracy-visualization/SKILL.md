---
name: top1-accuracy-visualization
description: Use when you have ranked metabolite annotation candidates with per-query
  cosine scores for the top two hits plus top-1 correctness labels, and need to test
  whether a larger Δcos gap increases annotation reliability.
when_to_use_negative:
- Candidate lists contain only a single hit per query, so Δcos = cos(top-1) − cos(top-2)
  cannot be computed
- No ground-truth correctness labels are available, so top-1 accuracy per Δcos bin
  cannot be estimated
- The goal is to produce the ranked annotations themselves (spectra prediction and
  candidate ranking) rather than evaluate the reliability of an existing ranking
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SingleFrag
  role: Generates the upstream in silico MS/MS library and the ranked candidate annotations
    whose cosine scores and correctness labels are stratified by Δcos
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Produces the 300-dimensional molecular embedding fed to SingleFrag ANN models,
    which underlies the predicted spectra used for candidate ranking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:0fc360ff5549512bf02538011a223116754274e7af937b7201f1d7c14e150e6a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# top1-accuracy-visualization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Stratifies top-1 metabolite annotation correctness by the cosine-similarity gap (Δcos = cos(top-1) − cos(top-2)) between the two highest-ranked candidates, and visualizes binned correctness to test whether a larger gap (e.g. Δcos > 0.2) indicates the top candidate is the true metabolite.

## When to use

Use when you have ranked metabolite annotation candidates with per-query cosine scores for the top two hits plus top-1 correctness labels, and need to test whether a larger Δcos gap increases annotation reliability.

## When NOT to use

- Candidate lists contain only a single hit per query, so Δcos = cos(top-1) − cos(top-2) cannot be computed
- No ground-truth correctness labels are available, so top-1 accuracy per Δcos bin cannot be estimated
- The goal is to produce the ranked annotations themselves (spectra prediction and candidate ranking) rather than evaluate the reliability of an existing ranking

## Inputs

- Ranked candidate annotation lists from the upstream SingleFrag annotation run, with per-query cosine scores for the top-1 and top-2 candidates
- Top-1 correctness labels per query (whether the top-ranked candidate is the true metabolite)

## Outputs

- gap_stratified_accuracy.png: binned plot of top-1 correctness versus Δcos gap, mirroring the Fig. 4d-f panel layout
- gap_stratified_accuracy.csv: per-bin top-1 accuracy table across Δcos bins

## How to apply

Load the ranked candidate lists produced by the upstream SingleFrag annotation run, retaining per-query cosine scores for the top two candidates and top-1 correctness labels. Compute Δcos = cos(top-1) − cos(top-2) for each query spectrum with numpy, then bin queries by Δcos using pandas equal-width bins spanning the score range, with the 0.2 gap threshold highlighted as the key cutoff (mirroring the Fig. 4d-f panel layout). Compute the fraction of queries whose top-ranked candidate is correct in each bin, plot correctness versus Δcos bin with matplotlib, and save gap_stratified_accuracy.png plus the per-bin table as gap_stratified_accuracy.csv in the results folder. Validate that bins with Δcos exceeding 0.2 show top-candidate correctness above 0.5, consistent with the reported signal: the top candidate is the correct molecule in the majority of cases once the gap exceeds 0.2.

## Related tools

- **SingleFrag** (Generates the upstream in silico MS/MS library and the ranked candidate annotations whose cosine scores and correctness labels are stratified by Δcos) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Produces the 300-dimensional molecular embedding fed to SingleFrag ANN models, which underlies the predicted spectra used for candidate ranking)

## Examples

```
df['dcos'] = df['cos_top1'] - df['cos_top2']; acc = df.groupby(pd.cut(df['dcos'], bins=np.arange(0, 1.0, 0.05)))['correct'].mean()
```

## Evaluation signals

- Bins with Δcos exceeding 0.2 show top-1 candidate correctness above 0.5 (top candidate correct in the majority of cases)
- gap_stratified_accuracy.csv contains one accuracy value per equal-width Δcos bin covering the full score range, with the 0.2 cutoff highlighted
- Every computed Δcos = cos(top-1) − cos(top-2) is non-negative, since candidates are ranked by cosine similarity
- The plotted figure mirrors the panel layout of Fig. 4d-f (correctness versus Δcos gap bins)

## Limitations

- The 0.2 Δcos threshold is an empirical cutoff observed in the Fig. 4d-f stratification; bins below this gap do not guarantee majority-correct top candidates
- Correctness stratification requires labeled true metabolites, so it is only applicable where ground truth (e.g. curated test spectra) exists
- Upstream ranking quality depends on the annotation model; the ANN models outperform GNN and COM models, so gap-stratified reliability reflects the ANN-based ranking behavior

## Evidence

- [results] Δcos stratification finding: [evidence span withheld — anchor status: not_found]
