---
name: classification-precision-recall-evaluation
description: Use when two classifiers must be compared on compounds that carry sensory
  bitter/nonbitter ground-truth labels, and precision, recall, balanced accuracy,
  and F1 must be tabulated to decide which tool is preferable.
when_to_use_negative:
- When chemical structures are unknown and the goal is direct spectrum-to-bitterness
  prediction — the article recommends BitterMasS direct prediction in that setting,
  so a structure-based comparison is moot
- When no sensory-derived bitter/nonbitter ground-truth labels exist for the test
  compounds, since precision, recall, BA, and F1 cannot be computed
- When the target output is bitterness intensity rather than a binary bitter/nonbitter
  class assignment
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: BitterMasS
  role: Random Forest classifier trained on 5414 experimental mass spectra; predicts
    bitter/nonbitter directly from EI-MS or ESI-MS/MS positive-mode spectra and is
    scored against sensory labels in the evaluation
- name: BitterPredict
  role: Structure-based classifier applied to in-domain compounds (−3 ≤ A log P ≤
    7, MW ≤ 700) to provide the comparator predictions for the precision/recall benchmark
  repo: Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:e568ca91deb9e6db84161ceed10e5d82e4b5d4dde8bf6a62963d055832bf58e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# classification-precision-recall-evaluation

## Summary

Side-by-side evaluation of binary bitter/nonbitter classifiers (structure-based BitterPredict vs spectrum-based BitterMasS) using precision, recall, balanced accuracy, and F1 computed against sensory-derived ground-truth labels on external test sets.

## When to use

Use when two classifiers must be compared on compounds that carry sensory bitter/nonbitter ground-truth labels, and precision, recall, balanced accuracy, and F1 must be tabulated to decide which tool is preferable.

## When NOT to use

- When chemical structures are unknown and the goal is direct spectrum-to-bitterness prediction — the article recommends BitterMasS direct prediction in that setting, so a structure-based comparison is moot
- When no sensory-derived bitter/nonbitter ground-truth labels exist for the test compounds, since precision, recall, BA, and F1 cannot be computed
- When the target output is bitterness intensity rather than a binary bitter/nonbitter class assignment

## Inputs

- External test set of literature-extracted EI-MS / ESI-MS/MS positive-mode mass spectra with sensory-derived bitter/nonbitter labels (e.g., 106 spectra for external set 1)
- Known chemical structures for the same compounds, with computed A log P and molecular weight for applicability-domain filtering
- Binary bitter/nonbitter predictions from BitterMasS (spectrum input) and from BitterPredict (structure input, in-domain compounds only)

## Outputs

- Side-by-side metrics table (CSV, e.g., bitterpredict_vs_bittermass_external1.csv) with precision, recall, balanced accuracy, and F1 per tool
- Coverage/counts of compounds retained after BitterPredict applicability-domain filtering versus total test-set size
- Decision record: which classifier is preferable when structures are known

## How to apply

Run both classifiers on the identical external test set: apply BitterMasS directly to the EI-MS / ESI-MS/MS positive-mode spectra, and apply BitterPredict only to compounds passing its applicability domain (−3 ≤ A log P ≤ 7, MW ≤ 700). Score each tool's bitter/nonbitter predictions against the sensory-derived labels, computing precision and recall per tool, then balanced accuracy and F1 for the head-to-head comparison. On external set 1 (106 literature-extracted spectra), target the reported reference values: BitterMasS precision = 0.67, recall = 0.93 and BitterPredict precision = 0.81, recall = 0.81; also record that BitterPredict achieves higher BA and F1 when the structure is known. Track prediction coverage separately, because the applicability domain filter removes some compounds from BitterPredict's evaluation. Tabulate both tools' metrics side by side and save the result (e.g., bitterpredict_vs_bittermass_external1.csv).

## Related tools

- **BitterMasS** (Random Forest classifier trained on 5414 experimental mass spectra; predicts bitter/nonbitter directly from EI-MS or ESI-MS/MS positive-mode spectra and is scored against sensory labels in the evaluation)
- **BitterPredict** (Structure-based classifier applied to in-domain compounds (−3 ≤ A log P ≤ 7, MW ≤ 700) to provide the comparator predictions for the precision/recall benchmark) — Niv-Lab/BitterPredict1

## Examples

```
BitterPredict('descriptors.csv')
```

## Evaluation signals

- External set 1 metrics reproduce reported values: BitterMasS precision ≈ 0.67 and recall ≈ 0.93; BitterPredict precision ≈ 0.81 and recall ≈ 0.81
- Internal test set reference: BitterMasS precision = 0.83 and recall = 0.90
- External set 2 reference: BitterMasS ≈ 58% accuracy and 99% recall on 26 compounds
- BitterPredict yields higher balanced accuracy and F1 than BitterMasS on the same in-domain subset when structures are known
- The number of compounds scored with BitterPredict equals the number passing the applicability domain filter, not the full test-set size

## Limitations

- BitterPredict's applicability domain (−3 ≤ A log P ≤ 7, MW ≤ 700) excludes some compounds, reducing its prediction coverage relative to BitterMasS
- BitterMasS can trade accuracy for recall: on external set 2 it reached only 58% accuracy but 99% recall
- Prediction on individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set, so metrics should be computed on the combined spectra
- Only about 30% of training compounds had at least one identified mass spectrum, limiting the spectrum-based model's evaluation context

## Evidence

- [abstract] Internal test set precision/recall reference for BitterMasS: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [abstract] External set 1 precision/recall for BitterMasS: "For these external test cases, BitterMasS exhibited 67% precision and 93% recall for the first"
- [abstract] External set 2 accuracy/recall for BitterMasS: "58% accuracy and 99% recall for the second"
- [abstract] Strategy comparison motivating the head-to-head evaluation: "The spectrum–bitterness prediction strategy was more effective than the spectrum–structure–bitterness prediction strategy and covered more compounds"
- [readme] BitterPredict role as structure-based classifier comparator: "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
- [readme] Workflow step source for running BitterPredict on descriptor files: "BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules"
