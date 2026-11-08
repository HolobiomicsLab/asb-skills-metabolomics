---
name: bitterness-classification-prediction
description: Use when you need bitter/nonbitter labels for small molecules and have
  either known chemical structures (with descriptors) or experimental EI-MS / ESI-MS/MS
  positive-mode mass spectra, e.g. for metabolome screening where most spectra lack
  assigned structures.
when_to_use_negative:
- Spectra were acquired in negative ion mode or by other ionization modes not covered
  by BitterMasS, which 'makes predictions based on EI-MS or ESI-MS/MS spectra obtained
  in positive ion mode'
- The compound is outside BitterPredict's applicability domain (A log P outside [-3,
  7] or MW > 700), so the structure-based prediction is not applicable
- The goal is predicting bitterness intensity or receptor (TAS2R) associations rather
  than bitter/nonbitter classification; dedicated tools such as BitterIntense exist
  for intense bitterness
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: BitterMasS
  role: Random Forest classifier that predicts bitterness directly from experimental
    EI-MS or ESI-MS/MS positive-mode mass spectra, without requiring chemical structures
- name: BitterPredict
  role: Structure-based ML classifier used after applying its applicability domain
    filter (-3 <= A log P <= 7, MW <= 700); run via the MATLAB script BitterPredict.m
    on descriptor CSV/EXCEL files
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:c5b13f84c18f5f391ad4c4d617e6ea144026e4e6f08195f2412f6def01af5081
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bitterness-classification-prediction

## Summary

Classifies compounds as bitter or nonbitter using either BitterPredict (structure-based ML classifier requiring descriptors such as A log P and MW) or BitterMasS (Random Forest trained on 5414 experimental EI-MS and ESI-MS/MS positive-mode mass spectra), enabling bitterness prediction even when only spectral data are available.

## When to use

Use when you need bitter/nonbitter labels for small molecules and have either known chemical structures (with descriptors) or experimental EI-MS / ESI-MS/MS positive-mode mass spectra, e.g. for metabolome screening where most spectra lack assigned structures.

## When NOT to use

- Spectra were acquired in negative ion mode or by other ionization modes not covered by BitterMasS, which 'makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode'
- The compound is outside BitterPredict's applicability domain (A log P outside [-3, 7] or MW > 700), so the structure-based prediction is not applicable
- The goal is predicting bitterness intensity or receptor (TAS2R) associations rather than bitter/nonbitter classification; dedicated tools such as BitterIntense exist for intense bitterness

## Inputs

- Chemical structures with computed descriptors (A log P, molecular weight) for BitterPredict, supplied as CSV or EXCEL files
- Experimental EI-MS or ESI-MS/MS positive ion mode mass spectra (e.g. extracted from MassBank) for BitterMasS
- Sensory-derived bitter/nonbitter labels for evaluation against external test sets

## Outputs

- Per-compound or per-spectrum binary bitter/nonbitter prediction
- Precision and recall (or balanced accuracy and F1-score) against sensory labels
- Side-by-side comparison table of BitterPredict vs BitterMasS metrics (e.g. bitterpredict_vs_bittermass_external1.csv)

## How to apply

Choose the tool based on available input: if the chemical structure is known, compute A log P and molecular weight and first apply the BitterPredict applicability domain filter (retaining compounds with -3 <= A log P <= 7 and MW <= 700), then run BitterPredict on the retained compounds. If only the mass spectrum is available, run BitterMasS directly on the EI-MS or ESI-MS/MS positive-mode spectrum; BitterMasS is a Random Forest classifier trained on 5414 experimental mass spectra and bypasses the structure-annotation stage entirely. When comparing both tools on the same compounds (external test set 1, 106 literature-extracted spectra), compute precision and recall against sensory-derived bitter/nonbitter labels and tabulate side by side. When the structure is unknown, the article recommends direct prediction from the spectrum, since structure annotation is 'the most problematic stage' and known structures cover only 5-10% of the metabolome. For BitterPredict usage, the MATLAB script BitterPredict.m accepts CSV or EXCEL files containing the required molecular descriptors and outputs a bitter/nonbitter prediction per molecule.

## Related tools

- **BitterMasS** (Random Forest classifier that predicts bitterness directly from experimental EI-MS or ESI-MS/MS positive-mode mass spectra, without requiring chemical structures)
- **BitterPredict** (Structure-based ML classifier used after applying its applicability domain filter (-3 <= A log P <= 7, MW <= 700); run via the MATLAB script BitterPredict.m on descriptor CSV/EXCEL files) — https://github.com/Niv-Lab/BitterPredict1

## Examples

```
matlab -batch "BitterPredict('molecule_descriptors.csv')"
```

## Evaluation signals

- On internal test data, BitterMasS should reproduce approximately precision = 0.83 and recall = 0.90
- On external test set 1 (106 literature-extracted spectra), BitterMasS should reach approximately precision = 0.67 and recall = 0.93, while BitterPredict should reach approximately precision = 0.81 and recall = 0.81 on in-domain compounds
- Every compound submitted to BitterPredict must satisfy the applicability domain: -3 <= A log P <= 7 and MW <= 700; compounds failing this are excluded before prediction
- Output must be a binary bitter/nonbitter class per compound or spectrum, not a continuous intensity score
- When both tools are compared on the same compounds with known structures, BitterPredict should show higher balanced accuracy and F1-score than BitterMasS

## Limitations

- BitterMasS currently only supports EI-MS or ESI-MS/MS spectra obtained in positive ion mode
- Only about 30% of the compounds in the initial bitter and nonbitter training set had at least one mass spectrum identified, and nonbitter compound spectral representation in the training set is insufficient
- Prediction on individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set
- BitterPredict's applicability domain excludes some compounds, limiting its prediction coverage
- Known structures represent only 5-10% of the metabolome, so the spectrum-first strategy is needed for the unassigned 'dark' metabolome; no software version or hosting location for BitterMasS itself is provided in the article

## Evidence

- [abstract] BitterMasS Random Forest training set: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [abstract] External test set 1 performance: "For these external test cases, BitterMasS exhibited 67% precision and 93% recall for the first"
- [discussion] Recommendation when structure unknown: "the direct prediction of bitterness from the mass spectrum is recommended when the chemical structure of a compound is not yet unknown"
- [discussion] BitterPredict definition: "BitterPredict14 is a tool that predicts the bitterness of a compound based on its chemical structure."
- [readme] README input format for BitterPredict: "BitterPredict.m gets as input CSV or EXCEL files with required descriptors of molecules, and calucautes a predictions if each molecule is bitter or not."
- [readme] README description of BitterPredict: "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
- [discussion] Positive ion mode limitation: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
