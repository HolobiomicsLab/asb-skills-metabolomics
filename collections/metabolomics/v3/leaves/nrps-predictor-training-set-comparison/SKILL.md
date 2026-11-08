---
name: nrps-predictor-training-set-comparison
description: Use when you need to compare labelled A-domain substrate resources (MIBiG
  3.0 vs SANDPUMA/NRPSPredictor2 training sets) to justify dataset choice for NRPS
  specificity ML models.
when_to_use_negative:
- When you need substrate specificity predictions themselves (use SANDPUMA or NRPSPredictor2
  directly) rather than a data-volume comparison of their training sets.
- When analysing AT-domain substrate annotations in PKS systems — the article motivates
  this skill specifically by the greater difficulty of A-domain substrate prediction.
- When the target resource is not MIBiG JSON-formatted curation data (e.g. raw GenBank
  files); the genbanks directory is not the labelled A-domain source.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: SANDPUMA
  role: A-domain selectivity predictor whose published training-set size is used as
    a comparison baseline for the MIBiG 3.0 labelled A-domain count
- name: NRPSPredictor2
  role: A-domain selectivity predictor whose published training-set size is used as
    a second comparison baseline for the MIBiG 3.0 labelled A-domain count
- name: mibig-secmet/mibig-json
  role: Source of MIBiG 3.0 annotations in JSON format, parsed with Python's json
    module to count labelled A-domain data points
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:63457fea705847939334f302ed14bfb78ad4bc179e2509e3e31d69b22d0be83e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NRPSPredictor2
  - SANDPUMA
  - mibig-secmet/mibig-json
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# NRPS predictor training-set comparison

## Summary

Quantifies the labelled NRPS A-domain substrate data available in MIBiG 3.0 (2775 A-domain annotations over 274 unique substrates) and computes fold-ratios against the training sets of SANDPUMA and NRPSPredictor2 to benchmark data-volume advantage for selectivity modelling.

## When to use

Use when you need to compare labelled A-domain substrate resources (MIBiG 3.0 vs SANDPUMA/NRPSPredictor2 training sets) to justify dataset choice for NRPS specificity ML models.

## When NOT to use

- When you need substrate specificity predictions themselves (use SANDPUMA or NRPSPredictor2 directly) rather than a data-volume comparison of their training sets.
- When analysing AT-domain substrate annotations in PKS systems — the article motivates this skill specifically by the greater difficulty of A-domain substrate prediction.
- When the target resource is not MIBiG JSON-formatted curation data (e.g. raw GenBank files); the genbanks directory is not the labelled A-domain source.

## Inputs

- MIBiG 3.0 entries in JSON format from the mibig-secmet/mibig-json repository (data directory, entry status tracked via cluster.status field)
- Published training-set sizes of SANDPUMA and NRPSPredictor2 from their original publications (refs 23 and 24)

## Outputs

- a_domain_training_set_comparison.csv — three-row table (MIBiG 3.0, SANDPUMA, NRPSPredictor2) of labelled data-point counts plus two fold-ratio columns

## How to apply

Load MIBiG 3.0 entries in JSON format from the mibig-secmet/mibig-json repository (data directory) with Python's json module and parse each entry's NRPS A-domain annotations, counting only data points carrying an experimentally validated substrate specificity label; the count must reconcile with the 2020 A-domain substrates reported for MIBiG 3.0. Retrieve the published training-set sizes of SANDPUMA and NRPSPredictor2 from their original publications (references 23 and 24 in the MIBiG 3.0 article). Compute the fold-ratio of the MIBiG 3.0 labelled A-domain count against each predictor's training-set size with pandas, then write a three-row table (MIBiG 3.0, SANDPUMA, NRPSPredictor2) with labelled data-point counts plus two ratio columns to a_domain_training_set_comparison.csv. The rationale is that A-domains recognise over 500 known substrates, so substrate prediction needs substantially more data than AT-domain prediction in PKS systems — MIBiG 3.0 provides 3–4x the labelled points of either predictor's training set.

## Related tools

- **SANDPUMA** (A-domain selectivity predictor whose published training-set size is used as a comparison baseline for the MIBiG 3.0 labelled A-domain count)
- **NRPSPredictor2** (A-domain selectivity predictor whose published training-set size is used as a second comparison baseline for the MIBiG 3.0 labelled A-domain count)
- **mibig-secmet/mibig-json** (Source of MIBiG 3.0 annotations in JSON format, parsed with Python's json module to count labelled A-domain data points) — https://github.com/mibig-secmet

## Examples

```
import json, glob, pandas as pd; n = sum(1 for f in glob.glob('mibig-json/data/*.json') for d in json.load(open(f)).get('a_domains', []) if d.get('substrate')); pd.DataFrame(...).to_csv('a_domain_training_set_comparison.csv')
```

## Evaluation signals

- Counted MIBiG 3.0 A-domain data points must match the reported figure of 2020 annotated (validated) A-domain substrates.
- The unique-substrate tally should reconcile with 274 unique substrates across 2775 A-domain annotations.
- Both computed fold-ratios (vs SANDPUMA and vs NRPSPredictor2) must fall in the reported 3–4x range.
- The output CSV must have exactly three rows (MIBiG 3.0, SANDPUMA, NRPSPredictor2) and five columns (counts plus two ratio columns).
- Only entries with experimentally validated substrate specificity labels may be counted; unvalidated annotations must be excluded.

## Limitations

- Fold-ratio comparison assumes published training-set sizes from refs 23 and 24 are directly comparable to MIBiG annotation counts; overlap between datasets is not accounted for.
- MIBiG 3.0 removed 69 duplicate and low-quality entries in the update, so counts from older MIBiG JSON snapshots will not reconcile.
- Labelled A-domain annotations are only a subset of the >500 known A-domain substrates; coverage is partial and biased toward experimentally characterised BGCs.

## Evidence

- [intro] SANDPUMA/NRPSPredictor2 training-set comparison: "the training sets used for the A-domain selectivity predictors SANDPUMA (23) and NRPSPredictor2 (24)"
- [intro] A-domain data challenge motivation: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"
- [intro] 2020 validated A-domain substrates: [evidence span withheld — anchor status: not_found]
- [readme] MIBiG JSON source: [evidence span withheld — anchor status: not_found]
