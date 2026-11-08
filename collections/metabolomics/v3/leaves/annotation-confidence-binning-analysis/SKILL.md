---
name: annotation-confidence-binning-analysis
description: Use when ranked metabolite annotation candidates exist with per-query
  cosine scores and correctness labels, and the goal is to quantify how annotation
  reliability grows with the Δcos gap between top-1 and top-2 candidates.
when_to_use_negative:
- Annotation results contain only a single ranked candidate per query, so no top-1/top-2
  cosine gap can be computed
- Correctness labels are unavailable for queries, preventing stratification of top-1
  accuracy by gap bin
- The downstream question is absolute similarity-score filtering (e.g. a fixed cosine
  cutoff) rather than gap-based ranking confidence
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SingleFrag
  role: Produces the predicted MS/MS spectra and in silico library used to generate
    the ranked candidate annotations whose cosine gaps are binned
  repo: github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds input SMILES into the 300-dimensional vectors fed to SingleFrag's ANN
    models, underpinning the spectral predictions used for candidate ranking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:5106571aacb53c613f5c1b4065375f579822412d07a0ef2ce133d7d318d1daa7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# annotation-confidence-binning-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Bins metabolite annotations by a confidence metric — the cosine-similarity gap Δcos between the top- and second-ranked candidate spectra — and computes per-bin top-1 correctness to show when the top candidate can be trusted.

## When to use

Use when ranked metabolite annotation candidates exist with per-query cosine scores and correctness labels, and the goal is to quantify how annotation reliability grows with the Δcos gap between top-1 and top-2 candidates.

## When NOT to use

- Annotation results contain only a single ranked candidate per query, so no top-1/top-2 cosine gap can be computed
- Correctness labels are unavailable for queries, preventing stratification of top-1 accuracy by gap bin
- The downstream question is absolute similarity-score filtering (e.g. a fixed cosine cutoff) rather than gap-based ranking confidence

## Inputs

- Ranked candidate annotation lists from an upstream annotation run (SingleFrag in silico MS/MS library), with per-query cosine scores for the top two candidates
- Top-1 correctness labels for each query spectrum

## Outputs

- gap_stratified_accuracy.csv — per-bin table of Δcos gap bins and fraction of queries whose top candidate is correct
- gap_stratified_accuracy.png — bar/line plot of top-1 correctness versus Δcos gap bins, mirroring Fig. 4d–f

## How to apply

Load the ranked candidate lists produced by the SingleFrag annotation run, retaining for each query the cosine scores of the top two candidates plus the top-1 correctness label. Compute Δcos = cos(top-1) − cos(top-2) with numpy for every query spectrum. Bin queries by Δcos using pandas — equal-width bins spanning the score range, with the 0.2 gap threshold highlighted as the key cutoff — and compute the fraction of queries whose top-ranked candidate is correct in each bin. Plot top-1 correctness versus Δcos gap bins with matplotlib, mirroring the panel layout of Fig. 4d–f, and export the per-bin accuracy table. The rationale is that when the second-ranked candidate's spectral similarity falls sufficiently behind the top hit, the top candidate becomes the majority-correct metabolite; the 0.2 gap is the article's decision threshold.

## Related tools

- **SingleFrag** (Produces the predicted MS/MS spectra and in silico library used to generate the ranked candidate annotations whose cosine gaps are binned) — github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds input SMILES into the 300-dimensional vectors fed to SingleFrag's ANN models, underpinning the spectral predictions used for candidate ranking)

## Examples

```
df['dcos'] = df['cos_top1'] - df['cos_top2']; binned = df.groupby(pd.cut(df['dcos'], bins=np.arange(0, 1, 0.05)))['top1_correct'].mean().to_csv('results/gap_stratified_accuracy.csv')
```

## Evaluation signals

- Bins with Δcos exceeding 0.2 show top-1 correctness above 0.5 (top candidate correct in the majority of cases), consistent with Fig. 4d–f
- Per-bin table contains a row for every bin across the Δcos score range, each with a non-degenerate query count
- Plotted accuracy values fall in [0, 1] and vary monotonically or near-monotonically with increasing gap in the article's trend
- Recomputed Δcos for a sampled query equals cos(top-1) − cos(top-2) to floating-point tolerance
- Output file gap_stratified_accuracy.csv is written to the results folder with bin boundaries and accuracy columns

## Limitations

- Correctness is majority (> 0.5), not certainty: at Δcos > 0.2 the top candidate is correct in the majority of cases, leaving a nontrivial error rate
- The analysis assumes an upstream ranked-candidate annotation run with reliable top-2 cosine scores; annotation reliability itself is described as the standing roadblock for metabolomics
- Bins depend on the chosen binning scheme; the 0.2 threshold is the article-specific cutoff and may need re-estimation for other spectral libraries or instruments

## Evidence

- [full_text] 0.2 gap threshold stratification: [evidence span withheld — anchor status: not_found]
- [intro] tool roles: "The first model embeds the molecule using mol2vec20 and uses this embedding as input to a multilayer, feedforward neural network."
- [discussion] scope of annotation problem: "which is the standing roadblock for the development of the full potential of metabolomics."
