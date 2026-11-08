---
name: summary-statistics-table-generation
description: Use when a ranked candidate list has been produced for each anchor–suspect
  pair and you need to quantify ranking accuracy via average correct-suspect rank
  and the proportion of cases where the true suspect is ranked first.
when_to_use_negative:
- When the input is not a per-pair ranked candidate list but raw MS/MS spectra or
  formula annotations — this skill operates on post-ranking output only.
- When evaluating molecular formula annotation correctness rather than derivative
  ranking — BAM performs formula annotation first, which requires different metrics.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: BAM
  role: Produces the ranked derivative/suspect candidate lists whose ranks are summarized
  repo: https://github.com/HassounLab/BAM
- name: GNN-SOM
  role: Supplies the site-of-metabolism likelihoods used to rank candidate structures
    within BAM
  repo: https://github.com/HassounLab/GNN-SOM
- name: Python 3.9
  role: Environment for computing summary statistics and generating figures
- name: Pandas
  role: Tabular aggregation of per-pair ranks into summary statistics
- name: NumPy
  role: Numerical computation of average ranks under ties
- name: Matplotlib
  role: Render the rank-distribution figure
- name: Seaborn
  role: Styled visualization of the rank distribution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:1c6a56a920895d7409be5151f7d3f32a88457e1869682a3994f9a8efc60d3dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - GNN-SOM
  - Matplotlib
  - NumPy
  - PROXIMAL2
  - Pandas
  - Python 3.9
  - Seaborn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# summary-statistics-table-generation

## Summary

Compute rank-based summary statistics (average suspect rank and top-k hit proportions) for BAM's ranked candidate lists and persist them as a CSV table with an accompanying rank-distribution figure.

## When to use

Use when a ranked candidate list has been produced for each anchor–suspect pair and you need to quantify ranking accuracy via average correct-suspect rank and the proportion of cases where the true suspect is ranked first.

## When NOT to use

- When the input is not a per-pair ranked candidate list but raw MS/MS spectra or formula annotations — this skill operates on post-ranking output only.
- When evaluating molecular formula annotation correctness rather than derivative ranking — BAM performs formula annotation first, which requires different metrics.

## Inputs

- Ranked candidate (suspect) lists per anchor–suspect pair produced by BAM's GNN-SOM ranking step
- Anchor–suspect pair metadata (suspect identifier, anchor identifier, anchor SMILES, masses) from the molecular-network-derived pair set

## Outputs

- CSV table of summary statistics (average correct-suspect rank, proportion ranked at positions 1–5)
- PNG rank-distribution figure (histogram/bar chart of ranks 1–5 and beyond)

## How to apply

Load the ranked suspect lists BAM produced for the anchor-suspect pairs derived from the global molecular network (30,184 structurally unique pairs yielding 60,368 anchor–suspect pairs). For each pair, locate the rank of the correct derivative in the candidate list; where multiple candidates tie, apply the average-rank rule — sum all tied ranks including duplicates and divide by the number of tied candidates. Compute the overall average rank and the proportion of true suspects at ranks 1–5, then write the summary statistics to CSV and plot a rank-distribution histogram (ranks 1–5 and beyond) with Matplotlib/Seaborn. Validate against the paper's reported values of an average rank of 2.1 and 57.7% top-1 proportion.

## Related tools

- **BAM** (Produces the ranked derivative/suspect candidate lists whose ranks are summarized) — https://github.com/HassounLab/BAM
- **GNN-SOM** (Supplies the site-of-metabolism likelihoods used to rank candidate structures within BAM) — https://github.com/HassounLab/GNN-SOM
- **Python 3.9** (Environment for computing summary statistics and generating figures)
- **Pandas** (Tabular aggregation of per-pair ranks into summary statistics)
- **NumPy** (Numerical computation of average ranks under ties)
- **Matplotlib** (Render the rank-distribution figure)
- **Seaborn** (Styled visualization of the rank distribution)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('suspect_rankings.csv'); avg = df['correct_rank'].mean(); top1 = (df['correct_rank']==1).mean(); df.groupby('correct_rank').size().to_csv('rank_summary.csv')
```

## Evaluation signals

- Computed average correct-suspect rank matches the paper's reported 2.1
- Proportion of cases where the correct suspect is ranked first matches the reported 57.7%
- Average-rank tie handling (sum of tied ranks including duplicates divided by number of tied candidates) is demonstrably applied
- Rank-distribution figure covers ranks 1 through 5 and aggregates remaining ranks, matching the paper's presentation
- CSV contains one summary row per evaluated statistic with reproducible counts of evaluated pairs

## Limitations

- Tied ranks must be resolved via the average-rank rule; ignoring ties inflates or deflates the average rank.
- Ranking accuracy depends on GNN-SOM site-of-metabolism predictions; errors upstream in PROXIMAL2 operator generation propagate into the ranks being summarized.

## Evidence

- [methods] Average rank tie resolution: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [results] BAM top-1 and average rank: [evidence span withheld — anchor status: not_found]
- [methods] Ranking step context: "BAM uses GNN-SOM to rank the generated derivatives based on the likelihood of their corresponding site-of-metabolism"
- [results] Pair set size: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
