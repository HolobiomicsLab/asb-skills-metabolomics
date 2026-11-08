---
name: suspect-ranking-evaluation
description: Use when ranked candidate lists (e.g. BAM suspect rankings) must be scored
  for how accurately the correct structure is placed, or when a rank-distribution
  histogram and top-k hit rates need to be computed and visualized.
when_to_use_negative:
- Input is a set of spectral similarity scores or cosine scores rather than ordinal
  candidate rankings — this skill evaluates rank positions, not score distributions.
- The task is to evaluate molecular formula annotation accuracy (SIRIUS-based) rather
  than suspect structure ranking; formula annotation precedes ranking and has its
  own accuracy metric (24.2% correct assignments).
- Candidate lists are unranked (all candidates treated as equally likely); the average-rank
  tie rule presupposes an ordered candidate list.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Python 3.9
  role: Scripting environment used to analyze PROXIMAL2 output and compute rank statistics
- name: NumPy
  role: Numerical computation for rank aggregation and tie handling
- name: Pandas
  role: Loading ranked candidate CSVs and writing summary statistics
- name: Matplotlib
  role: Rendering the rank-distribution figure
- name: Seaborn
  role: Styling the rank-distribution histogram/bar chart
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:be60bd2eadcf1df4faa57b1f4f7470abcea1d7b689282896635adb7707a6bde3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matplotlib
  - NumPy
  - PROXIMAL2
  - Pandas
  - Python 3.9
  - Seaborn
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# suspect-ranking-evaluation

## Summary

Evaluates the ranking accuracy of BAM-generated derivative candidates for anchor-suspect pairs by computing average ranks and rank-distribution statistics, using the average-rank tie-resolution rule. Quantifies how often the correct suspect structure appears at the top of the ranked candidate list.

## When to use

Use when ranked candidate lists (e.g. BAM suspect rankings) must be scored for how accurately the correct structure is placed, or when a rank-distribution histogram and top-k hit rates need to be computed and visualized.

## When NOT to use

- Input is a set of spectral similarity scores or cosine scores rather than ordinal candidate rankings — this skill evaluates rank positions, not score distributions.
- The task is to evaluate molecular formula annotation accuracy (SIRIUS-based) rather than suspect structure ranking; formula annotation precedes ranking and has its own accuracy metric (24.2% correct assignments).
- Candidate lists are unranked (all candidates treated as equally likely); the average-rank tie rule presupposes an ordered candidate list.

## Inputs

- Ranked candidate lists (suspect rankings) per anchor-suspect pair, e.g. BAM output where derivatives are ordered by GNN-SOM site-of-metabolism likelihood
- Ground-truth identities of the correct suspect structure for each anchor-suspect pair
- Anchor-suspect pair metadata (suspect identifier, suspect mass, anchor identifier, anchor SMILES) as in molecules_of_interest.csv

## Outputs

- Summary statistics table (CSV) with overall average rank and top-1..top-5 proportions
- Rank-distribution figure (PNG) via Matplotlib/Seaborn
- Intermediate ranked-candidate records used in the computation

## How to apply

Load the ranked candidate lists (suspect rankings) produced by BAM for anchor-suspect pairs derived from the global molecular network. Resolve rank ties using the average-rank rule: sum all tied ranks, including duplicates, and divide by the number of tied candidates. Compute the overall average rank and the proportion of cases where the true suspect identity is ranked at position k for k = 1 to 5. The article reports an average rank of 2.1 and 57.7% top-1 recovery across the 60,368 anchor-suspect pairs (30,184 structurally unique spectrum pairs from 454,091 annotated nodes in the global molecular network). Generate a rank-distribution histogram/bar chart (ranks 1-5 and beyond) using Matplotlib or Seaborn, and save summary statistics to CSV alongside intermediate ranked-candidate records.

## Related tools

- **Python 3.9** (Scripting environment used to analyze PROXIMAL2 output and compute rank statistics)
- **NumPy** (Numerical computation for rank aggregation and tie handling)
- **Pandas** (Loading ranked candidate CSVs and writing summary statistics)
- **Matplotlib** (Rendering the rank-distribution figure)
- **Seaborn** (Styling the rank-distribution histogram/bar chart)

## Examples

```
python evaluate_ranks.py --suspect_rankings suspect_rankings.csv --ground_truth pairs.csv --summary summary_stats.csv --fig rank_dist.png
```

## Evaluation signals

- Overall average rank of the correct suspect should be reproducible against the published value (2.1 for the anchor-suspect evaluation set).
- Proportion of cases where the correct suspect is ranked first should match the reported 57.7%.
- Rank-distribution histogram should show monotonically decreasing hit rates from rank 1 to rank 5 and beyond.
- Tie handling must follow the average-rank rule (sum all tied ranks including duplicates, divide by number of tied candidates); applying a min-rank or max-rank rule instead will shift statistics.
- Every suspect in the evaluation set must have a resolvable ground-truth rank — pairs where the correct derivative is absent from the candidate list should be excluded or reported separately.

## Limitations

- Ties in the ranked list inflate the average rank unless resolved with the specified averaging rule; the choice of tie rule materially changes reported metrics.
- Only suspects whose correct derivative was generated by the biotransformation rules can be scored — cases where no matching rule fired are not captured in the rank distribution.
- The evaluation is specific to anchor-suspect pairs derived from a global molecular network (454,091 annotated nodes; 30,184 structurally unique pairs); transferring statistics to other networks is untested.

## Evidence

- [methods] average rank tie resolution: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [methods] ranking candidate structures: "BAM uses GNN-SOM to rank the generated derivatives based on the likelihood of their corresponding site-of-metabolism"
- [results] rank evaluation workflow: "BAM identifies relevant mass differences between the suspects and the anchor, selects appropriate biotransformation rules corresponding to these mass differences (Figure 2b), and applies them to"
- [results] evaluation set scale: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
- [readme] BAM usage via README: "molecules_of_interest = csv file of list of queries. It must have an identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor."
