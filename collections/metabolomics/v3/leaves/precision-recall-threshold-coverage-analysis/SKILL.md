---
name: precision-recall-threshold-coverage-analysis
description: Use when comparing a probabilistic classifier (e.g. spectrum-to-property
  predictions on 17k test spectra) against a baseline with hard labels, and you need
  precision-recall trade-offs plus how much data remains usable at a precision floor.
when_to_use_negative:
- The downstream application requires every spectrum to receive a prediction with
  no confidence abstention — in that case precision–recall–coverage analysis overstates
  practical utility versus a fixed-threshold classifier
- The test set is small or not structurally disjoint from training molecules (the
  article uses Murcko-histogram-disjoint splits on 17k spectra; metrics on small or
  overlapping splits are not comparable)
- The baseline model outputs no calibrated score, so only a single operating point
  exists and a threshold sweep cannot be constructed
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DreaMS
  role: Supplies the probability scores whose threshold sweep yields the precision-recall-coverage
    metrics
  repo: https://github.com/pluskal-lab/DreaMS
- name: SIRIUS
  role: Hard-label baseline classifier whose fixed precision (≤ 0.51) is compared
    against the DreaMS curve
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:39229145aac46077b02c41a1b12e90711ed43028438d4eed568c0dc2d7f4aa82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - MoNA
  - NIST20
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# precision-recall-threshold-coverage-analysis

## Summary

Sweep the decision threshold of a binary classifier's probability scores to compute precision, recall, and coverage (fraction of predictions above a target precision floor), enabling fair comparison of uncertain-output classifiers such as DreaMS fluorine detection against deterministic baselines like SIRIUS.

## When to use

Use when comparing a probabilistic classifier (e.g. spectrum-to-property predictions on 17k test spectra) against a baseline with hard labels, and you need precision-recall trade-offs plus how much data remains usable at a precision floor.

## When NOT to use

- The downstream application requires every spectrum to receive a prediction with no confidence abstention — in that case precision–recall–coverage analysis overstates practical utility versus a fixed-threshold classifier
- The test set is small or not structurally disjoint from training molecules (the article uses Murcko-histogram-disjoint splits on 17k spectra; metrics on small or overlapping splits are not comparable)
- The baseline model outputs no calibrated score, so only a single operating point exists and a threshold sweep cannot be constructed

## Inputs

- Per-spectrum probability scores from a fine-tuned binary classifier (e.g. DreaMS fluorine-detection head with focal binary cross-entropy loss α=0.8, γ=0.5)
- Held-out, previously unreported test set of labeled MS/MS spectra (e.g. 17,052 spectra from MassIVE MSV000094528)
- Hard-label baseline predictions from SIRIUS 5.6.3 for the same test set

## Outputs

- Precision-recall curve obtained by sweeping the decision threshold over probability scores
- Operating-point metrics (precision 0.91, recall 0.57 for DreaMS fluorine detection)
- Coverage at a fixed precision floor (e.g. 95% of predictions classified as certain at 90% precision)
- Per-model comparison table (fluorine_detection_results.csv) with per-threshold metrics

## How to apply

Run the fine-tuned DreaMS checkpoint (initialized from the pre-trained 116M-parameter transformer, Zenodo record 10997887) in inference mode over the held-out test set — e.g. the 17,052-spectrum fluorine test set from MassIVE accession MSV000094528 — collecting the model's probability score for each spectrum. Sweep the decision threshold across these scores to build the precision–recall curve rather than fixing a single operating point, because classifiers with well-calibrated uncertainty let you trade recall for confidence. To compare against a hard-label baseline such as SIRIUS 5.6.3, report the baseline's precision alongside the curve (SIRIUS does not exceed precision 0.51, while DreaMS reaches 0.91 at recall 0.57). Additionally select a threshold at a target precision floor (e.g. 90%) and report the resulting recall and the coverage — the fraction of predictions classified as certain (95% of DreaMS predictions pass the 90% precision threshold). Save per-threshold metrics to a table such as fluorine_detection_results.csv for reproducibility.

## Related tools

- **DreaMS** (Supplies the probability scores whose threshold sweep yields the precision-recall-coverage metrics) — https://github.com/pluskal-lab/DreaMS
- **SIRIUS** (Hard-label baseline classifier whose fixed precision (≤ 0.51) is compared against the DreaMS curve)

## Examples

```
python scripts/sweep_thresholds.py --scores fluorine_probs.csv --labels test_labels.csv --precision-floor 0.90 --coverage 0.95 --out fluorine_detection_results.csv
```

## Evaluation signals

- The reported precision at the chosen operating point matches the paper's headline value (0.91 precision at 0.57 recall for DreaMS fluorine detection)
- The baseline's precision does not exceed 0.51 on the same test set, reproducing the SIRIUS comparison
- At the 90% precision threshold, the coverage (fraction of predictions above threshold) is approximately 95%
- Per-threshold metrics table is complete: every threshold on the probability-score sweep has a paired precision and recall value
- Test-set size and structural disjointness are verifiable: 17,052 spectra in-house test set with no shared Murcko scaffolds to the training split

## Limitations

- Coverage depends on calibration of the probability scores; if the fluorine-detection head is a proof-of-concept fine-tuning rather than a production model, the 90%-precision thresholds may not generalize
- Recall at the precision floor (0.57) is modest — nearly half of true fluorine-containing spectra are discarded when only confident predictions are kept
- The article frames the fingerprint fine-tuning as proof of concept; the same caution applies to threshold-derived uncertainty classification for other chemical properties

## Evidence

- [abstract] DreaMS achieves a precision of 0.91 with a recall of 0.57: "Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57"
- [abstract] 95% of DreaMS predictions can be classified as certain using 90% precision thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] The dataset is available under MassIVE accession number MSV000094528: "The dataset is available under MassIVE accession number MSV000094528"
- [methods] Source card workflow: sweep decision threshold on test-set probability scores to compute precision, recall, and coverage at 90% precision: [evidence span withheld — anchor status: not_found]
- [methods] Source card workflow: construct Murcko-histogram-disjoint train/val/test splits: [evidence span withheld — anchor status: not_found]
