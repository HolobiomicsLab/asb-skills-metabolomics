---
name: cross-dataset-training-set-size-analysis
description: Use when you must justify a database's suitability as an ML training
  resource by comparing its labelled instance count against the training-set sizes
  of existing predictors in the same domain.
when_to_use_negative:
- The question is about predictor accuracy or benchmarking predictions on held-out
  data — this skill only quantifies labelled data volume, not performance.
- The database entries are minimal/low-quality or lack experimentally validated substrate
  specificity labels, since unlabelled annotations cannot serve as training data points.
- Comparing training sets from different prediction domains (e.g. PKS AT-domains vs
  NRPS A-domains) where fold-ratios are not meaningful.
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: SANDPUMA
  role: Reference A-domain selectivity predictor whose published training-set size
    is used as the comparison baseline
- name: NRPSPredictor2
  role: Reference A-domain selectivity predictor whose published training-set size
    is used as the comparison baseline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:413339ff028735c1b68289288d811a9395e080f76fe4084783e067a97d439eba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NRPSPredictor2
  - SANDPUMA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Cross-dataset training-set size analysis

## Summary

Quantifies how many labelled data points a curated database (e.g. MIBiG 3.0's 2020 annotated NRPS A-domain substrates) provides relative to the training sets of established predictors (SANDPUMA, NRPSPredictor2), by counting substrate annotations from source JSON entries and computing fold-ratios against published training-set sizes.

## When to use

Use when you must justify a database's suitability as an ML training resource by comparing its labelled instance count against the training-set sizes of existing predictors in the same domain.

## When NOT to use

- The question is about predictor accuracy or benchmarking predictions on held-out data — this skill only quantifies labelled data volume, not performance.
- The database entries are minimal/low-quality or lack experimentally validated substrate specificity labels, since unlabelled annotations cannot serve as training data points.
- Comparing training sets from different prediction domains (e.g. PKS AT-domains vs NRPS A-domains) where fold-ratios are not meaningful.

## Inputs

- MIBiG 3.0 JSON entries from the mibig-secmet/mibig-json repository `data` directory (NRPS A-domain annotations with experimentally validated substrate specificity labels)
- Published training-set sizes of SANDPUMA and NRPSPredictor2 from their original publications

## Outputs

- a_domain_training_set_comparison.csv — three-row table (MIBiG 3.0, SANDPUMA, NRPSPredictor2) of labelled data-point counts plus fold-ratio columns
- Unique substrate count (274 unique substrates from 2775 A-domain annotations)

## How to apply

Load the target database's structured records — for MIBiG 3.0 this means parsing the JSON entries in the mibig-secmet/mibig-json `data` directory with Python's json module — and count unique data points carrying an experimentally validated label (here, NRPS A-domain annotations, expected to reconcile with the 2020 A-domain substrates reported in MIBiG 3.0, covering 274 unique substrates). Retrieve the published training-set sizes of the reference predictors from their original papers (references 23 and 24 in the article: SANDPUMA and NRPSPredictor2). Compute the fold-ratio of the database count against each predictor's training-set size, then export a comparison table (one row per dataset plus ratio columns) with pandas. Rationale: A-domain substrate prediction in NRPS systems requires substantially more data than AT-domain prediction in PKS systems because A-domains recognise over 500 known substrates, so demonstrating a 3–4x data-volume advantage substantiates claims that MIBiG is the largest A-domain substrate resource.

## Related tools

- **SANDPUMA** (Reference A-domain selectivity predictor whose published training-set size is used as the comparison baseline)
- **NRPSPredictor2** (Reference A-domain selectivity predictor whose published training-set size is used as the comparison baseline)

## Examples

```
import json, glob, pandas as pd; ads = [a for f in glob.glob('data/*.json') for a in parse_a_domains(json.load(open(f))) if a.get('validated')]; pd.DataFrame(rows).to_csv('a_domain_training_set_comparison.csv')
```

## Evaluation signals

- The counted MIBiG 3.0 A-domain data points must match the reported figure of 2020 annotated A-domain substrates.
- The counted annotations must reconcile with 2775 A-domain annotations covering 274 unique substrates.
- The computed fold-ratios against SANDPUMA and NRPSPredictor2 training sets must fall in the reported 3–4x range.
- The output CSV must contain exactly three dataset rows (MIBiG 3.0, SANDPUMA, NRPSPredictor2) plus two ratio columns.

## Limitations

- The count is limited to A-domain annotations carrying an experimentally validated substrate specificity label; unvalidated annotations in the JSON entries must be excluded or the count will overshoot.
- Fold-ratio comparisons depend on training-set sizes reported in the predictors' original publications (references 23 and 24), which may use differing definitions of a 'labelled data point'.
- MIBiG 3.0 removed 69 duplicate and low-quality entries, so counts from older MIBiG versions (e.g. MIBiG 2.0's 2021 entries) will not reconcile.

## Evidence

- [intro] A-domain substrate prediction is harder and more data-hungry than PKS AT-domain prediction: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"
- [intro] SANDPUMA and NRPSPredictor2 are the reference training sets for comparison: "the training sets used for the A-domain selectivity predictors SANDPUMA (23) and NRPSPredictor2 (24)"
- [intro] MIBiG 3.0 validated and annotated 2020 protein domain substrates of NRPSs: "the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020"
- [readme] The MIBiG JSON repository is the data source for counting A-domain annotations: "This repository tracks the current [MIBiG] annotations in JSON format."
- [readme] The repository layout (data vs genbanks directories) matters for locating parseable entries: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
