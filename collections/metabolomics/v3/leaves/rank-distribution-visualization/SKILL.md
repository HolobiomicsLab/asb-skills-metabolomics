---
name: rank-distribution-visualization
description: Use when BAM has produced ranked candidate derivative lists for anchor–suspect
  pairs from a global molecular network and you need to quantify and visualize where
  the correct suspect lands in each ranking.
when_to_use_negative:
- Input is raw MS/MS spectra or the global molecular network rather than already-computed
  BAM suspect rankings — run BAM (PROXIMAL2 rule application + GNN-SOM ranking) first.
- Evaluating molecular formula or structure annotation success (e.g. SIRIUS-based
  formula/structure assignment) rather than ranking accuracy of correct-suspect placement.
- Anchor-suspect pairs are not tied to a ranked candidate list, so there is no rank
  to distribute or visualize.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3520
tools:
- name: BAM
  role: Generates and ranks suspect candidate derivatives whose ranks are visualized
    and scored by this skill
  repo: https://github.com/HassounLab/BAM
- name: GNN-SOM
  role: Ranks putative derivatives by site-of-metabolism likelihood, producing the
    ranking whose distribution is analyzed
  repo: https://github.com/HassounLab/GNN-SOM
- name: PROXIMAL2
  role: Applies biotransformation rules to the anchor to generate the derivative candidates
    that are subsequently ranked
  repo: https://github.com/HassounLab/PROXIMAL2
- name: Matplotlib
  role: Used to render the rank-distribution histogram/bar chart
- name: Seaborn
  role: Used alongside Matplotlib for the rank-distribution plot styling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:0a8d25ef86f4833a5a5d9a8a1ddd0aa60eec0269dfe5fa04f902bf3e5b7caedc
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
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# rank-distribution-visualization

## Summary

Visualizes and tabulates how accurately BAM ranks the correct suspect molecular structure among its generated derivatives, by computing average rank (with average-rank tie resolution) and the proportion of correct suspects at rank positions 1–5, then plotting the rank distribution with Matplotlib/Seaborn and saving summary statistics to CSV.

## When to use

Use when BAM has produced ranked candidate derivative lists for anchor–suspect pairs from a global molecular network and you need to quantify and visualize where the correct suspect lands in each ranking.

## When NOT to use

- Input is raw MS/MS spectra or the global molecular network rather than already-computed BAM suspect rankings — run BAM (PROXIMAL2 rule application + GNN-SOM ranking) first.
- Evaluating molecular formula or structure annotation success (e.g. SIRIUS-based formula/structure assignment) rather than ranking accuracy of correct-suspect placement.
- Anchor-suspect pairs are not tied to a ranked candidate list, so there is no rank to distribute or visualize.

## Inputs

- Ranked candidate lists (suspect rankings) produced by BAM for anchor-suspect pairs
- CSV file of anchor-suspect pairs with identifier, mass for suspect and anchor, and anchor SMILES
- Python environment (Python 3.9) with NumPy, Pandas, Matplotlib, Seaborn

## Outputs

- Rank-distribution figure (PNG) showing histogram/bar chart of ranks 1-5 and beyond
- CSV summary statistics table with overall average rank and rank-position proportions
- Intermediate ranked-candidate records used in the computation

## How to apply

Load BAM's ranked candidate (suspect ranking) output for anchor–suspect pairs derived from the global molecular network. Resolve rank ties using the average-rank rule: sum all tied ranks including duplicates and divide by the number of tied candidates. Compute summary statistics — overall average rank and the proportion of true suspect identities at each rank position k = 1..5. Compare against the paper's reported benchmarks (average rank 2.1, top-1 proportion 57.7%). Generate a rank-distribution figure (histogram or bar chart of ranks 1–5 and beyond) and save summary statistics to CSV alongside intermediate ranked-candidate records.

## Related tools

- **BAM** (Generates and ranks suspect candidate derivatives whose ranks are visualized and scored by this skill) — https://github.com/HassounLab/BAM
- **GNN-SOM** (Ranks putative derivatives by site-of-metabolism likelihood, producing the ranking whose distribution is analyzed) — https://github.com/HassounLab/GNN-SOM
- **PROXIMAL2** (Applies biotransformation rules to the anchor to generate the derivative candidates that are subsequently ranked) — https://github.com/HassounLab/PROXIMAL2
- **Matplotlib** (Used to render the rank-distribution histogram/bar chart)
- **Seaborn** (Used alongside Matplotlib for the rank-distribution plot styling)

## Examples

```
python analyze_ranking.py --suspect_rankings bam_output/ --pairs molecules_of_interest.csv --out ranks_summary.csv --plot ranks_distribution.png
```

## Evaluation signals

- Overall average rank of the correct suspect ≈ 2.1
- Proportion of cases where the correct suspect is ranked first ≈ 57.7%
- Tie handling matches the average-rank rule: sum all ranks including duplicates, divide by number of tied candidates
- Rank-distribution figure covers ranks 1–5 and a beyond-rank category; CSV summary written alongside intermediate ranked-candidate records
- Recomputed statistics match the paper's reported benchmarks within tolerance

## Limitations

- Ties in candidate rankings must be resolved with the average-rank convention (sum of all tied ranks including duplicates divided by the number of tied candidates), otherwise the average rank and top-1 proportion will not reproduce the paper's values.
- Reported values (average rank 2.1, 57.7% top-1) are specific to the anchor-suspect pairs derived from the global molecular network; other pair sets may yield different distributions.
- The skill summarizes ranking quality only; it does not itself evaluate whether BAM's biotransformation rules or the mass-difference filtering (e.g. unit-resolution matching against 17,254 of 42,307 RetroRules-derived rules) were applied correctly.

## Evidence

- [results] Average rank of correct suspect and top-1 proportion are computed from ranked candidate lists: [evidence span withheld — anchor status: not_found]
- [methods] Tie resolution rule: "To resolve ties, the average rank is computed by summing all ranks, including duplicates, and dividing by the number of tied candidates."
- [methods] Ranking candidates is a distinct BAM step whose output this skill visualizes: "BAM uses GNN-SOM to rank the generated derivatives based on the likelihood of their corresponding site-of-metabolism"
- [results] Anchor-suspect pair set is the input basis for ranking evaluation: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
- [readme] BAM command-line invocation pattern: "Once the code and conda environments are structured as specified in the Installation and Requirements section above, run the runBAM.sh file as it is"
