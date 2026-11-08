---
name: external-validation-set-sampling
description: Use when a classifier trained on curated spectra (e.g., BitterMasS) must
  be evaluated on literature-extracted or laboratory-measured spectra of compounds
  unseen in training, and per-compound sampling is needed to avoid duplicate-spectrum
  bias.
when_to_use_negative:
- Evaluating on the internal held-out test set split from the 5414-spectrum training
  data — that is internal validation, not external sampling
- Input spectra are computationally predicted (e.g., CFM-ID) rather than experimental
  EI-MS/ESI-MS/MS positive-ion-mode measurements
- Compounds lack ground-truth sensory bitter/non-bitter labels, since recall/precision/F1
  cannot be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: BitterMasS
  role: Random Forest classifier applied to the sampled external spectra to obtain
    direct bitter/non-bitter predictions
- name: BitterPredict
  role: Structure-based classifier applied to MassBank-annotated structures on the
    same sampled set, with applicability-domain coverage tracking
  repo: Niv-Lab/BitterPredict1
- name: MassBank
  role: Source of experimental EI-MS and ESI-MS/MS positive ion mode spectra and reference
    library for structure annotation in the indirect arm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:806be22824c52857f40b78074915ec6f79bde683833e9921ae8e0815cb39f75f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# external-validation-set-sampling

## Summary

Assemble and sample an external validation set of experimental mass spectra (EI-MS and ESI-MS/MS positive ion mode) with ground-truth bitter/non-bitter labels, selecting one spectrum per compound with a fixed random seed to enable fair, reproducible head-to-head comparison of prediction strategies.

## When to use

Use when a classifier trained on curated spectra (e.g., BitterMasS) must be evaluated on literature-extracted or laboratory-measured spectra of compounds unseen in training, and per-compound sampling is needed to avoid duplicate-spectrum bias.

## When NOT to use

- Evaluating on the internal held-out test set split from the 5414-spectrum training data — that is internal validation, not external sampling
- Input spectra are computationally predicted (e.g., CFM-ID) rather than experimental EI-MS/ESI-MS/MS positive-ion-mode measurements
- Compounds lack ground-truth sensory bitter/non-bitter labels, since recall/precision/F1 cannot be computed

## Inputs

- Merged external set of experimental mass spectra (EI-MS and ESI-MS/MS positive ion mode) with ground-truth bitter/non-bitter labels
- MassBank reference spectral library entries for the sampled compounds (for the indirect strategy)
- Compound identity / multiple-spectrum mapping used for one-spectrum-per-compound sampling

## Outputs

- Randomly sampled one-spectrum-per-compound validation subset (fixed seed, reproducible)
- Per-strategy recall, precision, and F1 metrics versus ground-truth labels
- Compound-coverage counts per strategy
- Metrics table such as strategy_comparison_metrics.csv

## How to apply

Merge candidate external spectra (e.g., EI-MS and ESI-MS/MS positive ion mode entries extracted from MassBank and 18 publications with sensory studies plus laboratory measurements) with their ground-truth bitter/non-bitter labels, yielding a set such as 130 'unknown' mass spectra. Because a compound may have multiple spectra, randomly select exactly one spectrum per compound using a fixed random seed so the comparison is reproducible and not weighted by well-spectrated compounds. Run each strategy (e.g., direct BitterMasS spectrum-bitterness vs. indirect MassBank-annotation + BitterPredict) on the same sampled spectra, treating compounds uncovered by the indirect path as missed predictions. Compute recall, precision, and F1 per strategy against the labels and verify against expected values (direct: recall ~0.93, F1 ~0.77; indirect: recall ~0.65, F1 ~0.65). Persist per-strategy metrics and coverage counts (e.g., to strategy_comparison_metrics.csv).

## Related tools

- **BitterMasS** (Random Forest classifier applied to the sampled external spectra to obtain direct bitter/non-bitter predictions)
- **BitterPredict** (Structure-based classifier applied to MassBank-annotated structures on the same sampled set, with applicability-domain coverage tracking) — Niv-Lab/BitterPredict1
- **MassBank** (Source of experimental EI-MS and ESI-MS/MS positive ion mode spectra and reference library for structure annotation in the indirect arm)

## Examples

```
python run_external_validation.py --spectra external_130.tsv --labels bitter_labels.csv --sample-one-per-compound --seed 42 --strategies direct,indirect --out strategy_comparison_metrics.csv
```

## Evaluation signals

- Sampled subset contains exactly one spectrum per compound and the same subset is reused across all compared strategies
- Re-running the sampler with the fixed seed reproduces the identical spectrum selection
- Reproduced direct-strategy metrics fall in line with reported values (recall ~0.93, precision ~0.67, F1 ~0.77)
- Reproduced indirect-strategy metrics match reported values (recall ~0.65, precision ~0.65-0.66, F1 ~0.65) with uncovered compounds counted as missed
- Per-strategy metrics and coverage counts are written to strategy_comparison_metrics.csv with all 130 compounds accounted for

## Limitations

- Only ~30% of compounds in the initial bitter and nonbitter training sets had at least one mass spectrum, limiting the pool of externally validatable compounds
- Nonbitter compound spectral representation is insufficient, which can depress precision in external evaluation
- Prediction on individual EI-MS or ESI-MS/MS sets alone showed poor results compared with the combined set, so sampled spectra should be analyzed by mode-combined models
- Many literature articles reporting mass spectra do not include the actual spectra, hampering assembly of the external set

## Evidence

- [abstract] External validation sampling design: "For these external test cases, BitterMasS exhibited 67% precision and 93% recall for the first"
- [discussion] Spectrum source and mode: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
- [discussion] External set provenance: "an external set was assembled based on 18 publications with sensory studies"
- [discussion] Indirect-strategy bottleneck: "annotating the structure from the spectrum (which is the most problematic stage)"
- [readme] README confirms BitterPredict usage: "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
