---
name: spike-detection-in-elemental-imaging
description: Use when raw LA-ICP-MS elemental images (Agilent MassHunter batches,
  Perkin Elmer Elan .
when_to_use_negative:
- The image is already drift-corrected and spike-free — the filter would replace genuine
  high-signal features at real spike positions only, but applying it unnecessarily
  risks altering valid pixels if the threshold is set too low.
- The input is spot-wise peak-integrated calibration data rather than a line-by-line
  image; Pew2's filtering tool is designed for image data where local window statistics
  are meaningful.
- Large contiguous regions of high or low signal (not isolated spikes) need removal
  — the rolling median filter targets isolated spikes at single positions, not broad
  regional artifacts that are better handled by segmentation or drift correction.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0610
tools:
- name: Pew2
  role: GUI application providing the Filtering tool with the Rolling Median option
    used to detect and replace spikes
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for importing LA-ICP-MS data and enabling scripted batch application
    of the rolling median filter
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Data container for elemental images; filtered images are exported as NumPy
    archives
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:628f3944184c30ac3d268dee64cc1a710b8d716194c2b16c3926fa52c77c3fce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spike-detection-in-elemental-imaging

## Summary

Rolling median spike-removal filtering in Pew2 replaces instrument-noise spikes in LA-ICP-MS elemental images with the local median value, using a user-defined threshold expressed in medians from the local median over a rolling window. It preserves valid pixel values while selectively cleaning isolated extreme-signal artifacts.

## When to use

Use when raw LA-ICP-MS elemental images (Agilent MassHunter batches, Perkin Elmer Elan .xl, Thermo Qtegra exports, or text image imports loaded via pewlib) contain isolated spikes or regions of extremely high/low signal from instrument noise that must be removed before calibration, segmentation, or visualization.

## When NOT to use

- The image is already drift-corrected and spike-free — the filter would replace genuine high-signal features at real spike positions only, but applying it unnecessarily risks altering valid pixels if the threshold is set too low.
- The input is spot-wise peak-integrated calibration data rather than a line-by-line image; Pew2's filtering tool is designed for image data where local window statistics are meaningful.
- Large contiguous regions of high or low signal (not isolated spikes) need removal — the rolling median filter targets isolated spikes at single positions, not broad regional artifacts that are better handled by segmentation or drift correction.

## Inputs

- LA-ICP-MS elemental image data loaded into Pew2 or as pewlib/NumPy arrays (Agilent MassHunter batches, Perkin Elmer Elan 'XL', Thermo Qtegra exports, text image / file-per-line exports)
- User-defined threshold value (distance in medians from local median, e.g. 3× standard deviation)
- Local rolling window size (e.g. 3, 5, or 7)

## Outputs

- Filtered elemental image with spike positions replaced by the local mean/median and valid pixels unchanged
- NumPy archive (.npz) of the cleaned image

## How to apply

Load the unfiltered elemental image into Pew2 (or as a pewlib/NumPy array). Open the Filtering tool and select the Rolling Median filter, whose threshold is defined as distance in medians from the local median. Choose a local window size (e.g. 3, 5, or 7) that balances outlier sensitivity against introduction of new artifacts. At each spike position, Pew2 compares the pixel value to the locally defined threshold — e.g. a pixel exceeding roughly 3× the standard deviation of surrounding pixels — and replaces only the outlying value with the local mean or median. Verify that valid pixels remain untouched and invalid spike values are replaced, then save the filtered image as a NumPy archive.

## Related tools

- **Pew2** (GUI application providing the Filtering tool with the Rolling Median option used to detect and replace spikes) — https://github.com/djdt/pewpew
- **pewlib** (Python library for importing LA-ICP-MS data and enabling scripted batch application of the rolling median filter) — https://github.com/djdt/pewlib
- **NumPy** (Data container for elemental images; filtered images are exported as NumPy archives)

## Examples

```
from pewlib import Pew; pew = Pew.load("calibrated.npz"); filtered = pew.filter_rolling_median(window=5, threshold=3.0)
```

## Evaluation signals

- Valid pixels remain untouched after filtering (compare original and filtered arrays outside spike positions)
- Invalid spike values are replaced by the local median of the surrounding window, not left as NaN or zero
- The filtered image retains the original image dimensions and element channel count from the imported dataset
- Visual inspection of the filtered image shows no residual isolated extreme-value spikes
- Threshold choice (distance in medians, e.g. ~3× SD) produces selective removal rather than broad suppression of genuine signal

## Limitations

- Filter aggressiveness depends on both threshold and window size; a window that is too large may mask genuine local features, while one too small may fail to detect wide spikes
- The filter operates on each spike position independently, so it cannot correct broad regional artifacts caused by drift or calibration error
- Requires the user to define the threshold manually; no automatic spike-count reporting is described

## Evidence

- [results] Spike removal by local mean/median filtering with a user-defined threshold: [evidence span withheld — anchor status: not_found]
- [supplementary] Pew2 filtering capabilities in supplementary demonstration: [evidence span withheld — anchor status: not_found]
- [readme] Pewlib availability for scripted batch processing: [evidence span withheld — anchor status: external_doc]
