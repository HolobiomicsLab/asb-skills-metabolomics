---
name: prediction-strategy-comparison-across-tools
description: Use when chemical structures are known for a set of compounds with EI-MS
  or ESI-MS/MS positive-mode spectra and sensory bitter/nonbitter labels, and both
  a structure-based and a spectrum-based predictor must be compared.
when_to_use_negative:
- When chemical structures are unknown or unannotated — in that 'dark metabolome'
  setting the article recommends direct prediction from the mass spectrum with BitterMasS
  rather than a structure-based route
- When spectra were acquired in negative ion mode — BitterMasS's current implementation
  only makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive
  ion mode
- When the goal is internal test-set performance of a single tool rather than a cross-tool
  strategy comparison — use the tool-specific internal benchmarks instead (e.g. BitterMasS
  precision = 0.83, recall = 0.90 on the internal test set)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: BitterMasS
  role: Spectrum-based Random Forest classifier (trained on 5414 experimental mass
    spectra) run directly on EI-MS / ESI-MS/MS spectra to predict bitter/nonbitter
    without needing structures
- name: BitterPredict
  role: Structure-based classifier applied to the in-domain compounds; its applicability
    domain (-3 <= A log P <= 7, MW <= 700) gates which compounds it can cover
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:39272f65131a02f080a50860f606c678b4e060eab206a920fd014cb02967eb7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# prediction-strategy-comparison-across-tools

## Summary

Benchmarks a structure-based bitterness classifier (BitterPredict) against a spectrum-based classifier (BitterMasS) on the same external mass-spectral test sets, quantifying the precision/recall trade-off and the coverage loss caused by BitterPredict's applicability domain.

## When to use

Use when chemical structures are known for a set of compounds with EI-MS or ESI-MS/MS positive-mode spectra and sensory bitter/nonbitter labels, and both a structure-based and a spectrum-based predictor must be compared.

## When NOT to use

- When chemical structures are unknown or unannotated — in that 'dark metabolome' setting the article recommends direct prediction from the mass spectrum with BitterMasS rather than a structure-based route
- When spectra were acquired in negative ion mode — BitterMasS's current implementation only makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode
- When the goal is internal test-set performance of a single tool rather than a cross-tool strategy comparison — use the tool-specific internal benchmarks instead (e.g. BitterMasS precision = 0.83, recall = 0.90 on the internal test set)

## Inputs

- External test set of literature-extracted EI-MS and ESI-MS/MS positive-mode mass spectra with sensory-derived bitter/nonbitter labels (e.g. 106 compounds in external test set 1)
- Known chemical structures for each test compound (SMILES or equivalent descriptors)
- Molecular descriptors for applicability-domain filtering: A log P and molecular weight

## Outputs

- Side-by-side precision, recall, balanced accuracy and F1-score table for BitterPredict vs BitterMasS on the same compounds (e.g. bitterpredict_vs_bittermass_external1.csv)
- Coverage report listing compounds excluded by BitterPredict's applicability domain (A log P outside [-3, 7] or MW > 700)
- Decision note stating which predictor to prefer when the structure is known vs unknown

## How to apply

Load the external test set (e.g. external test set 1: 106 literature-extracted EI-MS / ESI-MS/MS positive-mode spectra with sensory-derived labels) together with the known chemical structures. Compute A log P and molecular weight for each compound and retain only compounds inside BitterPredict's applicability domain (-3 <= A log P <= 7 and MW <= 700); record which compounds are excluded, since this restriction reduces prediction coverage. Run BitterPredict on the in-domain compounds and run BitterMasS (a Random Forest trained on 5414 experimental mass spectra) directly on the mass spectra of the same compounds, requiring no structures. Compute precision and recall (and balanced accuracy and F1) against the sensory labels; on external set 1 the reference values are precision = 0.67 / recall = 0.93 for BitterMasS and precision = 0.81 / recall = 0.81 for BitterPredict. Conclude that BitterPredict is preferable when the structure is known (higher BA and F1), while BitterMasS is the recommended direct route when the structure is unknown, because annotating the structure from the spectrum is the most problematic stage.

## Related tools

- **BitterMasS** (Spectrum-based Random Forest classifier (trained on 5414 experimental mass spectra) run directly on EI-MS / ESI-MS/MS spectra to predict bitter/nonbitter without needing structures)
- **BitterPredict** (Structure-based classifier applied to the in-domain compounds; its applicability domain (-3 <= A log P <= 7, MW <= 700) gates which compounds it can cover) — https://github.com/Niv-Lab/BitterPredict1

## Examples

```
python bitterpredict.py --input external_set1.csv --output bitterpredict_vs_bittermass_external1.csv
```

## Evaluation signals

- BitterMasS metrics on external test set 1 should reproduce precision = 0.67 and recall = 0.93; BitterPredict should show precision = 0.81 and recall = 0.81
- BitterPredict should achieve higher balanced accuracy and F1 than BitterMasS on compounds whose structures are known
- Applicability-domain filter must exclude compounds with A log P outside [-3, 7] or MW > 700, and the excluded count should be reported as a coverage loss
- Both tools must be evaluated on the same sensory-labelled compounds; metrics must be computed against the sensory labels, not against each other's predictions
- Output table should contain both tools' precision and recall side by side and be saved (e.g. as bitterpredict_vs_bittermass_external1.csv)

## Limitations

- BitterPredict's applicability domain excludes some compounds, so its prediction coverage is reduced relative to BitterMasS
- BitterMasS's current implementation only handles EI-MS or ESI-MS/MS spectra obtained in positive ion mode
- No software/code availability or hosting location is provided for BitterMasS; no version number is stated for either tool
- The combined spectral set outperformed individual EI-MS and ESI-MS/MS sets, and nonbitter compound spectral representation in the training set is insufficient
- Only about 30% of compounds in the initial bitter and nonbitter training sets had at least one mass spectrum identified

## Evidence

- [abstract] BitterMasS external set 1 benchmark values: "For these external test cases, BitterMasS exhibited 67% precision and 93% recall for the first"
- [discussion] Applicability domain rationale: "passing the applicability domain of BitterPredict, and finally the prediction itself"
- [discussion] Direct spectrum route recommended when structure is unknown: "the direct prediction of bitterness from the mass spectrum is recommended when the chemical structure of a compound is not yet unknown"
- [discussion] Positive ion mode limitation: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
- [readme] BitterPredict input usage from README: "BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules, and calucautes a predictions if each molecule is bitter or not."
