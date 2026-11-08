---
name: ground-truth-reference-peak-extraction
description: Use when preparing a user-defined ground-truth reference feature list
  (peaks, walls, backgrounds) for training a PeakBot CNN model on LC-HRMS profile-mode
  chromatograms, and you need accurate RT/mz/borders/mz-deviation values matched to
  real chromatographic peaks.
when_to_use_negative:
- The data is not profile-mode LC-HRMS chromatograms (e.g. already centroided feature
  tables or other data types).
- Co-eluting compounds (isomers) are present in the reference chromatograms, which
  PeakBot cannot disentangle during gradient-descend matching.
- Peak-picking has already been done and the ground-truth list only needs to be consumed,
  not refined.
tools:
- name: PeakBot
  role: Provides the smoothing and gradient-descend peak pre-detection that estimates
    borders/centers and matches them to the user-defined reference list; also offers
    a function to estimate LC-HRMS parameters automatically.
  repo: https://github.com/christophuv/PeakBot
- name: PeakBot_Example
  role: Provides quickExample_GPU.py / quickExample_CPU.py, template TSV reference
    lists (_template_Peaks.tsv, _template_Walls.tsv, _template_Backgrounds.tsv), and
    estimateParameters.py for guiding ground-truth specification.
  repo: https://github.com/christophuv/PeakBot_Example
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:1e66274c572ca1c02071a11a34992dc59ff15107845a2f3dfdb3bd85914db35b
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
  - CUDA toolkit
  - PeakBot
  - PeakBot_Example
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ground-truth-reference-peak-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract ground-truth reference features for PeakBot's CNN peak-picking training by pre-detecting chromatographic peaks with a smoothing and gradient-descend algorithm, matching them to a user-defined reference list, and extending the list with isotopologs of the same compound.

## When to use

Use when preparing a user-defined ground-truth reference feature list (peaks, walls, backgrounds) for training a PeakBot CNN model on LC-HRMS profile-mode chromatograms, and you need accurate RT/mz/borders/mz-deviation values matched to real chromatographic peaks.

## When NOT to use

- The data is not profile-mode LC-HRMS chromatograms (e.g. already centroided feature tables or other data types).
- Co-eluting compounds (isomers) are present in the reference chromatograms, which PeakBot cannot disentangle during gradient-descend matching.
- Peak-picking has already been done and the ground-truth list only needs to be consumed, not refined.

## Inputs

- LC-HRMS profile-mode training chromatograms
- User-defined reference feature list in TSV format with columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation
- Wall background list in TSV format with columns MZ, Rtstart, Rtend, MZdeviation
- Other background list in TSV format with columns RTStart, RTEnd, MZLow, MZHigh
- Minimal PeakBot LC-HRMS settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM)

## Outputs

- Updated ground-truth reference feature list with peaks' borders and centers estimated by the gradient-descend algorithm
- Extended reference list including isotopologs of the same compound
- Matched reference features ready for training-instance generation

## How to apply

Start by pre-detecting chromatographic peaks in the training chromatograms with PeakBot's smoothing and gradient-descend algorithm, which also estimates the peaks' borders and centers. Match these detected peaks with the user-defined reference list (the ground-truth; isolated single chromatographic peaks) with the aim of using the same chromatographic peak but from different samples; the properties of the reference features (RT, mz, LeftRTBorder, RightRTBorder, MZDeviation) are updated to best fit the chromatographic peaks from the reference chromatograms. The pre-detection also allows extending the reference list with isotopologs of the same compound, which is how the recommended minimum of 100 reference features is reached. If working with PeakBot_Example scripts, place references in tab-separated .tsv files (template_Peaks.tsv, template_Walls.tsv, template_Backgrounds.tsv) and use the gradient-descend approach to automatically compute the actual values in each chromatogram; take care that no co-eluting compounds (isomers) are present.

## Related tools

- **PeakBot** (Provides the smoothing and gradient-descend peak pre-detection that estimates borders/centers and matches them to the user-defined reference list; also offers a function to estimate LC-HRMS parameters automatically.) — https://github.com/christophuv/PeakBot
- **PeakBot_Example** (Provides quickExample_GPU.py / quickExample_CPU.py, template TSV reference lists (_template_Peaks.tsv, _template_Walls.tsv, _template_Backgrounds.tsv), and estimateParameters.py for guiding ground-truth specification.) — https://github.com/christophuv/PeakBot_Example

## Examples

```
python quickExample_GPU.py  # uses _template_Peaks.tsv, _template_Walls.tsv, _template_Backgrounds.tsv and PeakBot's gradient-descend pre-detection to build the ground-truth reference list
```

## Evaluation signals

- The updated reference feature list parses as a TSV with the required columns (RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation for peaks; MZ, Rtstart, Rtend, MZdeviation for walls; RTStart, RTEnd, MZLow, MZHigh for backgrounds).
- Reference feature RT/mz values differ per chromatogram as expected, since actual values are calculated automatically with the gradient-descend approach.
- The reference list contains at least 100 different reference features (possibly isotopologs of the same compound) as recommended.
- No co-eluting compounds (isomers) remain in the matched reference chromatograms.
- All minimal PeakBot settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM) have been specified for the chromatograms.

## Limitations

- Care must be taken that no co-eluting compounds (isomers) are present, because the gradient-descend approach will not separate them.
- A minimum of 100 reference features is needed to train the machine-learning model to achieve high-quality peak-picking results; fewer references degrade performance.
- The two ppm parameters (intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM) are difficult to evaluate and typically need the same values, estimated via TOPPView or the estimateParameters.py helper.
- Different chromatographic peak shapes (fronting, tailing, etc.) must be considered so the new PeakBot model can recognize them correctly.

## Evidence

- [readme] The format to specify the list of reference features is a simple tab-separated values (tsv) file with the following columns: RT | MZ | LeftRTBorder | RightRTBorder | MZDeviation: [evidence span withheld — anchor status: not_found]
- [readme] The actual values of the reference features and backgrounds in the different reference chromatograms are automatically calculated using a gradient-descend appraoch. Thus, these determined reference feature values can vary from chromatogram to chromatogram: [evidence span withheld — anchor status: external_doc]
- [abstract] A minimum of 100 reference features is needed to train the machine-learning model to achieve high-quality peak-picking results: [evidence span withheld — anchor status: not_found]
- [readme] For training, different reference features must be specified. A comparison has shown that at least 100 different reference features should be provided. However, this can also be different isotopologs of the same feature: [evidence span withheld — anchor status: external_doc]
