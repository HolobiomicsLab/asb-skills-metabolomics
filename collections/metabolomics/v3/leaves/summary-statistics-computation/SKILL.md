---
name: summary-statistics-computation
description: Use when a per-compound prediction table (e.g., MAIP scores keyed to
  SMILES) must be split into annotated subgroups and summarized as group sizes, means,
  and observed ranges to test a structure–activity correlation.
when_to_use_negative:
- Input is raw unannotated structures with no prediction scores or subgroup annotation
  — classify and score the compounds first (e.g., via MAIP and NPClassifier) before
  summarizing.
- The analysis goal is per-compound classification labels rather than group-level
  aggregate comparison — use the classification skill directly instead.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: MAIP
  role: Source of the per-compound biosynthetic-likelihood scores that are summarized
    per subgroup
- name: NPClassifier
  role: Confirms pathway/class assignments of the predicted actives via its /classify?smiles=<>
    API before subgroup annotation
  repo: https://github.com/mwang87/NP-Classifier
- name: Python
  role: Scripting language used to filter entries, batch-query MAIP, compute subgroup
    means/ranges, and write the CSV output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:fd6f03ecccf6a91658d84745d009a83b8731163e2184bd17e39ae98d5454eeba
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - MAIP
  - NPClassifier
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# summary-statistics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-subgroup summary statistics (group sizes, mean scores, score ranges) from per-compound prediction outputs, as done when splitting spirotetronate macrolides into decalin-containing vs. non-decalin groups and comparing mean MAIP antimalarial scores.

## When to use

Use when a per-compound prediction table (e.g., MAIP scores keyed to SMILES) must be split into annotated subgroups and summarized as group sizes, means, and observed ranges to test a structure–activity correlation.

## When NOT to use

- Input is raw unannotated structures with no prediction scores or subgroup annotation — classify and score the compounds first (e.g., via MAIP and NPClassifier) before summarizing.
- The analysis goal is per-compound classification labels rather than group-level aggregate comparison — use the classification skill directly instead.

## Inputs

- Per-compound table of SMILES strings with numeric prediction scores (e.g., MAIP scores from the EMBL-EBI MAIP web service)
- Binary subgroup annotation per compound (e.g., decalin-moiety presence/absence for spirotetronate macrolides from the NP Atlas)

## Outputs

- CSV file with per-compound scores and subgroup summary statistics (maip_spirotetronate_analysis.csv)
- Subgroup summary: group sizes (n), mean scores per subgroup, and observed score range

## How to apply

Start from a per-compound table holding SMILES strings and their prediction values (e.g., MAIP biosynthetic-likelihood scores fetched in batch from the EMBL-EBI MAIP web service for the filtered spirotetronate macrolide entries of the NP Atlas table). Annotate each compound with the structural feature of interest (here, presence of the decalin moiety) and split the set into two subgroups. For each subgroup compute the group size (n), the mean prediction score, and the observed minimum–maximum score range. Write both the per-compound scores and the subgroup summary statistics to an output CSV (e.g., maip_spirotetronate_analysis.csv) so downstream validation can check the subgroup means (97.2 for decalin-containing, n=55; 25.1 for non-decalin) against the reported individual score range of 14.2–138.8.

## Related tools

- **MAIP** (Source of the per-compound biosynthetic-likelihood scores that are summarized per subgroup)
- **NPClassifier** (Confirms pathway/class assignments of the predicted actives via its /classify?smiles=<> API before subgroup annotation) — https://github.com/mwang87/NP-Classifier
- **Python** (Scripting language used to filter entries, batch-query MAIP, compute subgroup means/ranges, and write the CSV output)

## Examples

```
python maip_spirotetronate_analysis.py --input np_atlas_macrolides.csv --group decalin --output maip_spirotetronate_analysis.csv
```

## Evaluation signals

- Decalin-containing subgroup yields mean MAIP score of 97.2 with n = 55.
- Non-decalin subgroup yields mean MAIP score of 25.1.
- All individual MAIP scores fall within the reported 14.2–138.8 range.
- Output CSV contains both per-compound scores and subgroup summary statistics.
- Sum of subgroup sizes (55 + non-decalin n) equals the total number of filtered spirotetronate macrolide entries.

## Limitations

- MAIP scores are predicted biosynthetic-likelihood values, not measured antimalarial activities; the decalin correlation is a computational observation.
- Results depend on the upstream NPClassifier filtering of the NP Atlas table to spirotetronate macrolides; errors in classification propagate into subgroup membership.
- NPClassifier logs which structures were classified but not which users queried them; cached query results may be returned via the cached flag, which should be considered when reproducing scores.

## Evidence

- [full_text] MAIP batch scoring and subgroup split produce per-compound scores written to CSV: [evidence span withheld — anchor status: not_found]
- [full_text] Subgroup summary statistics definition with expected means and range: [evidence span withheld — anchor status: not_found]
