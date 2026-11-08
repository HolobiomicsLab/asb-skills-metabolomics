---
name: tie-resolution-average-rank-calculation
description: Use when evaluating how accurately a ranked candidate list (e.g. BAM/GNN-SOM
  derivative rankings for anchor–suspect pairs) places the true suspect, and multiple
  candidates share the same rank score requiring average-rank tie resolution.
when_to_use_negative:
- Candidate lists contain no tied scores and each candidate has a unique rank — plain
  rank counting suffices.
- You are comparing two different ranking algorithms against each other on raw likelihood
  scores rather than summarizing a single ranking's accuracy — a paired statistical
  comparison is more appropriate.
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: BAM
  role: Produces the ranked derivative candidate lists whose tied ranks are resolved
    by average-rank calculation for evaluating suspect-ranking accuracy.
  repo: https://github.com/HassounLab/BAM
- name: GNN-SOM
  role: Generates the site-of-metabolism likelihood scores per atom that induce ties
    among derivative candidates needing average-rank resolution.
  repo: https://github.com/HassounLab/GNN-SOM
- name: Python 3.9
  role: Used (with NumPy/Pandas/Matplotlib/Seaborn) to compute average-rank ties,
    rank distributions, and summary statistics/figures.
- name: Pandas
  role: Used for loading and aggregating the ranked candidate records before computing
    the tie-resolved rank statistics.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:b729b535d1bc17f5132ba99ca8de795d9aa05b7fe1557b488b5f5a16358d26dc
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
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# tie-resolution-average-rank-calculation

## Summary

Resolves ties among equally-scored derivative candidates in BAM's GNN-SOM rankings by computing the average rank (summing all tied ranks, including duplicates, divided by the number of tied candidates), then summarizing the distribution of correct-suspect ranks.

## When to use

Use when evaluating how accurately a ranked candidate list (e.g. BAM/GNN-SOM derivative rankings for anchor–suspect pairs) places the true suspect, and multiple candidates share the same rank score requiring average-rank tie resolution.

## When NOT to use

- Candidate lists contain no tied scores and each candidate has a unique rank — plain rank counting suffices.
- You are comparing two different ranking algorithms against each other on raw likelihood scores rather than summarizing a single ranking's accuracy — a paired statistical comparison is more appropriate.

## Inputs

- Ranked candidate list per anchor–suspect pair (derivative structures ordered by GNN-SOM site-of-metabolism likelihood)
- Flag/identifier marking which candidate is the correct suspect structure for each pair
- Anchor–suspect pair metadata (suspect identifier/mass, anchor identifier/SMILES) from the global molecular network

## Outputs

- Average rank of the correct derivative across all evaluated pairs
- Proportion of true suspect identities ranked at positions 1–5 (rank histogram / cumulative top-k)
- CSV summary table of rank statistics and a PNG figure of the rank distribution
- Tie-resolved intermediate record of ranked-candidate ranks per pair

## How to apply

Load BAM's ranked candidate lists for the anchor–suspect pairs derived from the global molecular network (30,184 structurally unique, annotated spectrum pairs yielding 60,368 total pairs). For each true suspect that shares a rank score with one or more other derivative candidates, resolve the tie by summing all tied ranks (including duplicates) and dividing by the number of tied candidates — e.g. two candidates tied at rank 3 each receive average rank 3.5. With all ranks resolved, compute the overall average rank of the correct derivative (reported as 2.1) and the proportion of true suspect identities ranked at position k for k = 1..5 (reported 57.7% at top-1). The tie rule is required because GNN-SOM assigns a site-of-metabolism likelihood per atom, so multiple derivatives can receive identical likelihood scores and must not be broken arbitrarily. Summarize results as a rank-distribution table/CSV and a histogram figure of ranks 1–5 and beyond.

## Related tools

- **BAM** (Produces the ranked derivative candidate lists whose tied ranks are resolved by average-rank calculation for evaluating suspect-ranking accuracy.) — https://github.com/HassounLab/BAM
- **GNN-SOM** (Generates the site-of-metabolism likelihood scores per atom that induce ties among derivative candidates needing average-rank resolution.) — https://github.com/HassounLab/GNN-SOM
- **Python 3.9** (Used (with NumPy/Pandas/Matplotlib/Seaborn) to compute average-rank ties, rank distributions, and summary statistics/figures.)
- **Pandas** (Used for loading and aggregating the ranked candidate records before computing the tie-resolved rank statistics.)

## Examples

```
python eval_rank_accuracy.py --pairs anchor_suspect_pairs.csv --ranked-candidates bam_ranked_candidates.csv --tie-rule average --out-rank-stats rank_stats.csv --out-figure rank_distribution.png
```

## Evaluation signals

- Computed overall average rank of the correct derivative matches the paper's reported value of 2.1.
- Computed proportion of true suspects ranked at position 1 matches 57.7%.
- Tie resolution is applied consistently: sum of tied ranks divided by count of tied candidates, with duplicates counted.
- Rank distribution figure covers ranks 1–5 and beyond, with proportions summing to 1 (or 100%) across the evaluated pairs.
- Intermediate ranked-candidate records are saved alongside the summary CSV and PNG figure for reproducibility.

## Limitations

- Average-rank ties produce fractional ranks (e.g. 3.5), so downstream top-k percentages must be interpreted with that in mind.
- Evaluation is only as accurate as the anchor–suspect pair set derived from the global molecular network; mis-annotated anchors propagate into rank statistics.
- Only a small subset of the 60,368 anchor–suspect pairs had ranked candidate lists suitable for this accuracy evaluation.

## Evidence

- [methods] Average rank tie resolution rule: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [results] Reported average rank of correct suspect: [evidence span withheld — anchor status: not_found]
- [results] Reported top-1 proportion: [evidence span withheld — anchor status: not_found]
- [results] Anchor-suspect pair set size: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
- [methods] GNN-SOM ranking source causing ties: "BAM uses GNN-SOM to rank the generated derivatives based on the likelihood of their corresponding site-of-metabolism"
- [methods] Tooling used for analysis: "PROXIMAL2 was applied and its results were analyzed using Python 3.9, PubChemPy (version 1.0.4)"
