---
name: lc-hrms-reference-feature-extraction
description: 'Use when preparing training data for a PeakBot CNN model: you have LC-HRMS
  profile-mode chromatograms, a user-defined ground-truth list of isolated reference
  peaks, and need at least 100 reference features (incl. isotopologs) to learn peak
  characteristics.'
when_to_use_negative:
- Your goal is peak detection in new chromatograms with an already-trained model —
  use peakbot.runPeakBot instead of reference-feature extraction
- Your reference chromatograms contain co-eluting compounds (isomers), which the gradient-descend
  estimation of reference-feature values cannot handle reliably
- You have fewer than 100 reference features (including isotopologs) available — the
  article states a minimum of 100 is needed for high-quality peak-picking results
edam_operation: http://edamontology.org/operation_3633
tools:
- name: PeakBot
  role: 'Provides the reference-feature extraction functionality: smoothing and gradient-descend
    peak pre-detection with border/center estimation, matching to the user-defined
    ground-truth list, and isotopolog extension'
  repo: https://github.com/christophuv/PeakBot
- name: PeakBot_Example
  role: Reference scripts and template TSV files (_template_Peaks.tsv, _template_Walls.tsv,
    _template_Backgrounds.tsv) for specifying reference features and settings
  repo: https://github.com/christophuv/PeakBot_Example
- name: Python 3.8
  role: Runtime environment in which the PeakBot package and its reference-feature
    extraction run
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:619c847f0f2b13cefa864cc52288a78aaf3e584406bb648f2ace1184b4720e5a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/christophuv/PeakBot
  repo_url: https://github.com/christophuv/PeakBot
  tools:
  - Anaconda
  - CUDA toolkit
  - PeakBot
  - PeakBot_Example
  - Python
  - Python 3.8
  - cuDNN
  edam_operation: http://edamontology.org/operation_3633
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# lc-hrms-reference-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts and curates reference features (ground-truth chromatographic peaks) from LC-HRMS profile-mode training chromatograms with PeakBot, using a smoothing and gradient-descend peak pre-detection to estimate peak borders and centers, match peaks to a user-defined reference list across samples, and optionally extend the list with isotopologs — the prerequisite step for generating training instances for the PeakBot CNN.

## When to use

Use when preparing training data for a PeakBot CNN model: you have LC-HRMS profile-mode chromatograms, a user-defined ground-truth list of isolated reference peaks, and need at least 100 reference features (incl. isotopologs) to learn peak characteristics.

## When NOT to use

- Your goal is peak detection in new chromatograms with an already-trained model — use peakbot.runPeakBot instead of reference-feature extraction
- Your reference chromatograms contain co-eluting compounds (isomers), which the gradient-descend estimation of reference-feature values cannot handle reliably
- You have fewer than 100 reference features (including isotopologs) available — the article states a minimum of 100 is needed for high-quality peak-picking results

## Inputs

- LC-HRMS profile-mode training chromatograms
- User-defined ground-truth reference list (_template_Peaks.tsv format: RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- Optional wall background list (MZ, Rtstart, Rtend, MZdeviation)
- Optional other-background list (RTStart, RTEnd, MZLow, MZHigh)
- PeakBot settings: noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM

## Outputs

- Matched, updated reference features with estimated borders, centers, and RT/mz properties per chromatogram
- Extended reference list including isotopologs of the same compound (>=100 reference features)
- Reference-feature basis used to generate augmented training instances for the PeakBot CNN

## How to apply

Supply LC-HRMS chromatograms with a user-defined reference list of isolated single chromatographic peaks in tab-separated format with columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation. Run PeakBot's reference-feature extraction: first search for chromatographic peaks with a smoothing and gradient-descend algorithm, estimating each peak's borders and center; then match the detected peaks with the ground-truth reference list across samples so the same chromatographic peak from different samples is used, updating the reference-feature properties to best fit the observed peaks. Use this pre-detection to extend the reference list with isotopologs of the same compound. Provide at least 100 different reference features (isotopologs count separately) and ensure the different peak shapes present (fronting, tailing, etc.) are represented. The actual RT/MZ values of reference features are automatically re-determined per chromatogram by the gradient-descend approach, so care must be taken that no co-eluting compounds (isomers) are present in the reference chromatograms.

## Related tools

- **PeakBot** (Provides the reference-feature extraction functionality: smoothing and gradient-descend peak pre-detection with border/center estimation, matching to the user-defined ground-truth list, and isotopolog extension) — https://github.com/christophuv/PeakBot
- **PeakBot_Example** (Reference scripts and template TSV files (_template_Peaks.tsv, _template_Walls.tsv, _template_Backgrounds.tsv) for specifying reference features and settings) — https://github.com/christophuv/PeakBot_Example
- **Python 3.8** (Runtime environment in which the PeakBot package and its reference-feature extraction run) — https://github.com/christophuv/PeakBot

## Examples

```
pip install git+https://github.com/christophuv/PeakBot
```

## Evaluation signals

- The reference-feature list parses as TSV with exactly the required columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation
- At least 100 distinct reference features (isotopologs may count separately) are extracted before training-instance generation
- Estimated LeftRTBorder < RT (apex) < RightRTBorder for each matched reference feature
- Reference-feature properties are updated per chromatogram by the gradient-descend fit without errors from co-eluting isomers
- Downstream check: CNN trained on these features reaches peakType_ACCPeakNoPeak > 0.99 on the independent validation set (iV) with <5% train/validation metric gap

## Limitations

- A minimum of 100 different reference features is required to achieve high-quality peak-picking results from the trained model
- Reference-feature values are automatically calculated with a gradient-descend approach and can vary from chromatogram to chromatogram; no co-eluting compounds (isomers) may be present in the reference chromatograms
- Care must be taken to select and specify the many different chromatographic peak forms (fronting, tailing, etc.) present in the data so the model can recognize them correctly

## Evidence

- [readme] TSV format for the reference-feature list: [evidence span withheld — anchor status: external_doc]
- [readme] Gradient-descend recalculation and isomer caveat: [evidence span withheld — anchor status: external_doc]
