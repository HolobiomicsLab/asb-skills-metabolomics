---
name: rank-distribution-statistics
description: Use when you have ranked candidate lists (suspect rankings) from BAM
  for anchor-suspect pairs and need to measure ranking accuracy — e.g. average rank
  of the correct structure and the proportion ranked at position 1–5 — to benchmark
  GNN-SOM's site-of-metabolism ranking.
when_to_use_negative:
- When the goal is to generate or re-rank the candidate derivatives themselves (use
  PROXIMAL2 + GNN-SOM, not this statistics skill)
- When the evaluation target is molecular formula annotation or SIRIUS-based structure
  annotation accuracy rather than ranking accuracy
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BAM
  role: Produces the ranked candidate lists (suspect rankings) whose rank distribution
    is being evaluated
  repo: https://github.com/HassounLab/BAM
- name: GNN-SOM
  role: 'Source of the ranking signal: predicts site-of-metabolism likelihood to order
    putative derivatives'
  repo: https://github.com/HassounLab/GNN-SOM
- name: PROXIMAL2
  role: Generates the derivative candidates upstream that get ranked and then evaluated
  repo: https://github.com/HassounLab/PROXIMAL2
- name: Python 3.9
  role: Implementation environment for the statistics computation, tie resolution,
    and figure generation
- name: Matplotlib
  role: Renders the rank-distribution figure (histogram/bar chart) as PNG
- name: Seaborn
  role: Styling for the rank-distribution figure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:1b869790fb017247e0f52298d5916e7723c3772192c68d48e18f0894ac8762ad
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rank-distribution-statistics

## Summary

Computes summary statistics (average rank, top-k proportions) over BAM's ranked derivative candidate lists for anchor-suspect pairs, and visualizes the correct-suspect rank distribution to quantify how well GNN-SOM ranking places the true derivative near the top.

## When to use

Use when you have ranked candidate lists (suspect rankings) from BAM for anchor-suspect pairs and need to measure ranking accuracy — e.g. average rank of the correct structure and the proportion ranked at position 1–5 — to benchmark GNN-SOM's site-of-metabolism ranking.

## When NOT to use

- When the goal is to generate or re-rank the candidate derivatives themselves (use PROXIMAL2 + GNN-SOM, not this statistics skill)
- When the evaluation target is molecular formula annotation or SIRIUS-based structure annotation accuracy rather than ranking accuracy

## Inputs

- Ranked candidate lists (suspect rankings) per anchor-suspect pair from BAM, with the correct derivative identified
- CSV table of anchor–suspect pair records (identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor)

## Outputs

- Summary statistics CSV: overall average rank and proportion of correct-suspect ranks at position k (k = 1–5 and beyond)
- Rank-distribution figure (histogram or bar chart) saved as PNG
- Intermediate ranked-candidate records used in the computation

## How to apply

Load the ranked candidate lists (suspect rankings) that BAM produced for anchor-suspect pairs derived from the global molecular network (454,091 annotated nodes; 60,368 total anchor–suspect pairs). Resolve rank ties using the average-rank rule: sum all ranks including duplicates and divide by the number of tied candidates, as specified in the methods. Compute the overall average rank of the correct derivative and the proportion of cases where the true suspect appears at position k (k = 1 to 5), then compare against the paper's reported values (average rank 2.1; top-1 hit rate 57.7%). Plot the rank distribution as a histogram/bar chart of ranks 1–5 and beyond using Matplotlib/Seaborn, and save summary statistics to CSV alongside intermediate ranked-candidate records. The rationale is that GNN-SOM ranks derivatives by the likelihood of each atom being the site of metabolism, so a low average rank and high top-1 proportion validate the ranking stage of the BAM pipeline.

## Related tools

- **BAM** (Produces the ranked candidate lists (suspect rankings) whose rank distribution is being evaluated) — https://github.com/HassounLab/BAM
- **GNN-SOM** (Source of the ranking signal: predicts site-of-metabolism likelihood to order putative derivatives) — https://github.com/HassounLab/GNN-SOM
- **PROXIMAL2** (Generates the derivative candidates upstream that get ranked and then evaluated) — https://github.com/HassounLab/PROXIMAL2
- **Python 3.9** (Implementation environment for the statistics computation, tie resolution, and figure generation)
- **Matplotlib** (Renders the rank-distribution figure (histogram/bar chart) as PNG)
- **Seaborn** (Styling for the rank-distribution figure)

## Examples

```
python analyze_rank_accuracy.py --rankings suspect_rankings.csv --truth true_derivatives.csv --tie-rule average --out-summary rank_summary.csv --out-fig rank_distribution.png
```

## Evaluation signals

- Overall average rank of the correct derivative matches the paper's reported value of 2.1
- Proportion of cases where the correct suspect is ranked first matches the reported 57.7%
- Rank distribution histogram covers ranks 1–5 and beyond, with ties resolved by average rank before binning
- Summary statistics CSV and rank-distribution PNG are saved alongside the intermediate ranked-candidate records
- Every ranked candidate list contains the correct derivative label so top-k proportions are well-defined

## Limitations

- Rank ties must be resolved by the average-rank rule (sum all ranks including duplicates, divide by number of tied candidates) — failing to do so will skew the average-rank metric
- Top-1 rate of 57.7% leaves ~42% of cases where the correct derivative is not first, so ranking accuracy is not absolute
- The evaluation depends on the anchor–suspect pair set derived from the global molecular network (60,368 pairs); results are not transferable to other pair sources without re-evaluation

## Evidence

- [methods] Tie resolution: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [intro] Ranking basis: "the graph neural network-based GNN-SOM tool is used to predict the site-of-metabolism and rank these putative derivatives"
- [results] Evaluation set: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
- [readme] Input spec: "molecules_of_interest = csv file of list of queries. It must have an identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor."
- [methods] Ranking workflow: "BAM uses GNN-SOM to rank the generated derivatives based on the likelihood of their corresponding site-of-metabolism"
