---
name: binary-classification-metric-evaluation
description: Use when a binary bitter/nonbitter classifier (e.g. BitterMasS Random
  Forest on positive-ion-mode EI-MS or ESI-MS/MS spectra) must be scored on held-out
  or literature-derived test compounds.
when_to_use_negative:
- Predicting bitterness from chemical structure — use BitterPredict instead; this
  skill evaluates mass-spectrum-based (BitterMasS) predictions only.
- Spectra measured in negative ion mode — BitterMasS's current implementation is restricted
  to positive ion mode predictions.
- Multi-class or graded bitterness-intensity assessment (e.g. BitterIntense-style
  intense-bitterness prediction) — this skill is strictly binary bitter/nonbitter
  scoring.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: BitterMasS
  role: Random Forest classifier whose bitter/nonbitter predictions are aggregated
    per compound and scored with precision/recall against sensory labels
  repo: https://github.com/Niv-Lab/BitterPredict1
- name: MassBank
  role: Source of the experimental positive-ion-mode EI-MS and ESI-MS/MS spectra on
    which per-spectrum predictions are generated before evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:f4ec1158ef438fad11a8c5058cba34324061e621c66844fab2f32b001f36c64d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - MassBank
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# binary-classification-metric-evaluation

## Summary

Evaluates a binary (bitter/nonbitter) classifier by per-spectrum predictions aggregated to per-compound labels, then computing precision and recall (or accuracy and recall) against sensory ground truth on internal and external test sets.

## When to use

Use when a binary bitter/nonbitter classifier (e.g. BitterMasS Random Forest on positive-ion-mode EI-MS or ESI-MS/MS spectra) must be scored on held-out or literature-derived test compounds.

## When NOT to use

- Predicting bitterness from chemical structure — use BitterPredict instead; this skill evaluates mass-spectrum-based (BitterMasS) predictions only.
- Spectra measured in negative ion mode — BitterMasS's current implementation is restricted to positive ion mode predictions.
- Multi-class or graded bitterness-intensity assessment (e.g. BitterIntense-style intense-bitterness prediction) — this skill is strictly binary bitter/nonbitter scoring.

## Inputs

- Per-spectrum bitter/nonbitter predictions from the BitterMasS Random Forest classifier (positive-ion-mode EI-MS and ESI-MS/MS spectra)
- Per-compound sensory taste-panel bitterness ground-truth labels from the 18 cited publications and laboratory measurements
- Test-set compound lists (internal test set; external test set 1 of 106 literature-extracted compounds; external test set 2 of 26 compounds)

## Outputs

- Per-compound binary prediction labels (majority vote across multiple spectra)
- Precision and recall values per test set (internal: precision = 0.83, recall = 0.90; external test set 1: precision = 0.67, recall = 0.93)
- Accuracy and recall values per test set (external test set 2: 58% accuracy, 99% recall)
- Comparison of combined EI-MS + ESI-MS/MS performance versus individual spectral-mode subsets

## How to apply

Run the trained BitterMasS Random Forest classifier on each retrieved experimental spectrum to obtain per-spectrum bitter/nonbitter predictions. When a compound has multiple spectra (as in the 759 MassBank/DrugBank spectra for 106 literature-extracted compounds), combine per-spectrum predictions by majority vote to yield one label per compound. Compute precision and recall of the per-compound labels against sensory taste-panel ground truth, or accuracy and recall when class balance favors one class (as for the 26-compound external set). Report metrics for the internal test set as a baseline: precision = 0.83 and recall = 0.90; for external test set 1: precision = 0.67 and recall = 0.93; for external test set 2: 58% accuracy and 99% recall. Compare combined-set performance against individual EI-MS or ESI-MS/MS subsets, since prediction on individual sets showed poor results compared with the combined set.

## Related tools

- **BitterMasS** (Random Forest classifier whose bitter/nonbitter predictions are aggregated per compound and scored with precision/recall against sensory labels) — https://github.com/Niv-Lab/BitterPredict1
- **MassBank** (Source of the experimental positive-ion-mode EI-MS and ESI-MS/MS spectra on which per-spectrum predictions are generated before evaluation)

## Examples

```
sklearn.metrics.precision_score(y_true, y_pred, pos_label='bitter'); sklearn.metrics.recall_score(y_true, y_pred, pos_label='bitter')
```

## Evaluation signals

- Internal test set must reproduce precision = 0.83 and recall = 0.90
- External test set 1 (759 spectra, 106 compounds, majority-voted) must reproduce precision = 0.67 and recall = 0.93
- External test set 2 (26 compounds) must reproduce 58% accuracy and 99% recall
- Per-compound label counts must match the number of unique compounds, confirming majority-vote aggregation (e.g. 106 compounds from 759 spectra in external set 1)
- Combined EI-MS + ESI-MS/MS evaluation must outperform each individual spectral-mode subset evaluated alone

## Limitations

- External validation set assembled from 18 publications and laboratory measurements is not described with a hosting location or accession, making exact reproduction of ground-truth labels difficult
- Reference citations for the external sensory studies are garbled in the article text (e.g. 'studies51−5253606168'), preventing retrieval of the original sources
- Only ~30% of training compounds had at least one mass spectrum identified, and nonbitter compound spectral representation is insufficient, which may limit generalization of the evaluated metrics
- Most articles reporting experimental mass spectra do not include the actual spectra, so independent external evaluation is constrained by available standardized spectral deposition

## Evidence

- [abstract] BitterMasS achieved precision = 0.83 and recall = 0.90 on an internal test set: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [abstract] BitterMasS exhibited 67% precision and 93% recall on 106 literature-extracted spectra: "For these external test cases, BitterMasS exhibited 67% precision and 93% recall for the first"
- [abstract] BitterMasS exhibited 58% accuracy and 99% recall on spectra measured for 26 compounds: "58% accuracy and 99% recall for the second"
- [discussion] Prediction on individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [other] Per-spectrum predictions are combined by majority vote within each compound before computing per-compound metrics: [evidence span withheld — anchor status: not_found]
