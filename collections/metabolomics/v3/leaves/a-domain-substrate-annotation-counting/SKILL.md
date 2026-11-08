---
name: a-domain-substrate-annotation-counting
description: Use when you need to quantify how many labelled A-domain substrate data
  points a database (e.g. MIBiG 3.0) provides, or benchmark it against predictor training-set
  sizes (SANDPUMA, NRPSPredictor2) to justify ML training-data choices.
when_to_use_negative:
- The goal is to actually predict A-domain substrate specificity — use SANDPUMA or
  NRPSPredictor2 directly rather than counting training data
- The target domains are PKS AT-domains rather than NRPS A-domains; the article states
  AT-domain prediction is a lesser challenge than A-domain prediction, which 'recognise
  over 500 known substrates'
- Input BGC annotations are not experimentally validated MIBiG entries — counting
  unvalidated annotations would not match the reported labelled data-point figures
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0780
tools:
- name: SANDPUMA
  role: A-domain selectivity predictor whose published training-set size serves as
    the comparison baseline against the MIBiG 3.0 A-domain annotation count
- name: NRPSPredictor2
  role: A-domain selectivity predictor whose published training-set size serves as
    the second comparison baseline against the MIBiG 3.0 A-domain annotation count
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:40221f3d60f1c02a3fd28aee10d9645cffcb408bb319ce75607180bba0dd6464
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NRPSPredictor2
  - SANDPUMA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# a-domain-substrate-annotation-counting

## Summary

A benchmarking skill that counts experimentally validated NRPS A-domain substrate annotations in MIBiG 3.0 JSON data and compares the labelled data-point volume against the training-set sizes of A-domain selectivity predictors such as SANDPUMA and NRPSPredictor2.

## When to use

Use when you need to quantify how many labelled A-domain substrate data points a database (e.g. MIBiG 3.0) provides, or benchmark it against predictor training-set sizes (SANDPUMA, NRPSPredictor2) to justify ML training-data choices.

## When NOT to use

- The goal is to actually predict A-domain substrate specificity — use SANDPUMA or NRPSPredictor2 directly rather than counting training data
- The target domains are PKS AT-domains rather than NRPS A-domains; the article states AT-domain prediction is a lesser challenge than A-domain prediction, which 'recognise over 500 known substrates'
- Input BGC annotations are not experimentally validated MIBiG entries — counting unvalidated annotations would not match the reported labelled data-point figures

## Inputs

- MIBiG 3.0 JSON entry files (from the mibig-secmet/mibig-json repository, data/ directory, with cluster.status field)
- Published training-set sizes of SANDPUMA and NRPSPredictor2 from their original papers (references 23 and 24)

## Outputs

- Count of experimentally validated A-domain substrate annotations (expected ~2775 data points over 274 unique substrates, reconciling with the reported 2020 annotated A-domain substrates)
- Three-row CSV (a_domain_training_set_comparison.csv) of labelled data-point counts for MIBiG 3.0, SANDPUMA, and NRPSPredictor2 plus two fold-ratio columns (3-4x range)

## How to apply

Load the MIBiG 3.0 entries in JSON format from the mibig-secmet/mibig-json repository with Python's json module, parsing each entry's NRPS A-domain annotations from the 'data' directory (entry status tracked via the cluster.status field). Count unique A-domain data points carrying an experimentally validated substrate specificity label, expecting reconciliation with the 2020 A-domain substrates reported in MIBiG 3.0 (covering ~274 unique substrates out of the >500 known A-domain substrates). Retrieve the published training-set sizes of SANDPUMA and NRPSPredictor2 from their original publications and compute fold-ratios of the MIBiG count against each training-set size. The rationale is that A-domain substrate prediction is harder than PKS AT-domain prediction and 'will require substantially more data', so MIBiG's 3-4x data-volume advantage over existing predictor training sets makes it the preferred training resource. Write the three-row comparison table (MIBiG 3.0, SANDPUMA, NRPSPredictor2 counts plus ratio columns) to a CSV with pandas.

## Related tools

- **SANDPUMA** (A-domain selectivity predictor whose published training-set size serves as the comparison baseline against the MIBiG 3.0 A-domain annotation count)
- **NRPSPredictor2** (A-domain selectivity predictor whose published training-set size serves as the second comparison baseline against the MIBiG 3.0 A-domain annotation count)

## Examples

```
import json, glob; ads = [a for f in glob.glob('data/*.json') for e in [json.load(open(f))] for a in e['cluster'].get('nrps', {}).get('a_domains', [])]; print(len(ads))
```

## Evaluation signals

- The counted MIBiG 3.0 A-domain data points must reconcile with the reported figure of 2020 annotated A-domain substrates
- The computed fold-ratios versus SANDPUMA and NRPSPredictor2 training-set sizes must fall in the reported 3-4x range
- The unique-substrate count should be ~274, consistent with 2775 annotations over 274 unique substrates
- The output CSV must contain exactly three data rows (MIBiG 3.0, SANDPUMA, NRPSPredictor2) plus two ratio columns
- All counted A-domains must carry experimentally validated substrate specificity labels, not inferred or predicted ones

## Limitations

- MIBiG 3.0 covers only ~274 unique substrates of the >500 known A-domain substrates, so counting annotations overstates substrate-space coverage
- Entry status is tracked via the cluster.status field and the database removed 69 duplicate and low-quality entries, so filtering on validated status is required before counting
- MIBiG 3.0 includes 708 annotations transferred from the Walker and Clardy dataset, so provenance of annotations is mixed

## Evidence

- [intro] A-domain annotation count: "the validation and annotation of 2020"
- [intro] A-domain prediction challenge: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"
- [intro] training-set comparison: "the training sets used for the A-domain selectivity predictors SANDPUMA (23) and NRPSPredictor2 (24)"
- [readme] repository structure: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] data quality caveat: "we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
