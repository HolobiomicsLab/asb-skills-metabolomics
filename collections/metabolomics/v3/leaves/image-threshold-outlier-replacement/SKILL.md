---
name: image-threshold-outlier-replacement
description: Use when an imported elemental image contains isolated pixels of extremely
  low or high signal from instrument noise that would distort visualization or quantification,
  and the goal is targeted spike removal while leaving valid pixels untouched.
when_to_use_negative:
- Input is already a clean image with no isolated extreme-pixel artifacts — applying
  the filter would risk replacing valid high-signal features with local background
  values
- Noise is a broad drift or background trend across the whole acquisition rather than
  isolated spikes — use Pew2's drift correction tool instead
- Data is spot-wise or line-by-line with continuous intensity variation rather than
  a 2D image; the local window filter assumes spatial neighborhood structure in an
  image
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: Pew2
  role: GUI filtering tool that applies the local mean/median outlier replacement
    to elemental images
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for importing and exporting LA-ICP-MS data so images can be
    loaded as NumPy arrays for filtering
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Array backend used by pewlib to represent and export the filtered image
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:6e0825375db18dd18c142c4d0a26e253513509234f785960af6c39832dfa6c5c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# image-threshold-outlier-replacement

## Summary

Removes instrumental noise spikes from LA-ICP-MS elemental images by comparing each pixel value to a local mean or median and replacing outliers exceeding a user-defined threshold with the local value, as implemented in Pew2's filtering tool.

## When to use

Use when an imported elemental image contains isolated pixels of extremely low or high signal from instrument noise that would distort visualization or quantification, and the goal is targeted spike removal while leaving valid pixels untouched.

## When NOT to use

- Input is already a clean image with no isolated extreme-pixel artifacts — applying the filter would risk replacing valid high-signal features with local background values
- Noise is a broad drift or background trend across the whole acquisition rather than isolated spikes — use Pew2's drift correction tool instead
- Data is spot-wise or line-by-line with continuous intensity variation rather than a 2D image; the local window filter assumes spatial neighborhood structure in an image

## Inputs

- Unfiltered LA-ICP-MS elemental image (single element channel) as a pewlib/NumPy array
- Pew2-compatible import format: Agilent MassHunter batch (.b directory), Perkin Elmer Elan .xl, Thermo Qtegra export, delimited text image, or imzML
- User-defined outlier threshold (e.g. 3× standard deviation or distance in medians)

## Outputs

- Filtered elemental image with spike pixels replaced by local mean or median values
- NumPy archive (.npz) containing the filtered image for downstream analysis

## How to apply

Load the unfiltered elemental image (e.g. from a pewlib-imported Agilent MassHunter batch, Perkin Elmer Elan, Thermo Qtegra export, or delimited text image) into Pew2 and open the Filtering tool. Select the local mean or median filter and set a threshold such as 3× the standard deviation of surrounding pixels; the tool treats the threshold as a distance in medians from the local median when the rolling median mode is used. Choose a small local window (e.g. 3, 5, or 7 pixels) that balances outlier sensitivity against the risk of introducing new artifacts. Apply the filter so that, at each spike position, only values exceeding the locally defined threshold are replaced with the local mean or median. Save the filtered image as a NumPy archive for downstream segmentation or standardization steps.

## Related tools

- **Pew2** (GUI filtering tool that applies the local mean/median outlier replacement to elemental images) — https://github.com/djdt/pewpew
- **pewlib** (Python library for importing and exporting LA-ICP-MS data so images can be loaded as NumPy arrays for filtering) — https://github.com/djdt/pewlib
- **NumPy** (Array backend used by pewlib to represent and export the filtered image) — https://github.com/djdt/pewlib

## Examples

```
from pewlib.importer import read; img = read('batch.b'); img = pewlib.filter.local_median(img, window=5, threshold=3); pewlib.export(img, 'filtered.npz')
```

## Evaluation signals

- Valid (non-spike) pixel values remain numerically unchanged after applying the filter
- Only isolated extreme pixels are replaced; the surrounding local window's median or mean is unchanged within the expected neighborhood
- Filtered image, when visualized in Pew2 with a scale bar and color bar, no longer shows the isolated high/low signal anomalies
- The saved NumPy archive retains the correct image shape and element channel after filtering

## Limitations

- Choosing too small a local window can introduce new filtering artifacts rather than removing them
- The threshold is user-defined, so an inappropriate threshold may miss genuine spikes or overwrite valid features
- Spike identification assumes at least one element is present across the entirety of the sample with similar peak positions, limiting the technique to images where a marker element can guide integration

## Evidence

- [results] Spike removal uses local mean/median filter at spike positions: [evidence span withheld — anchor status: not_found]
- [results] Pew2 filtering tool workflow: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 written in Python using NumPy and PySide2: [evidence span withheld — anchor status: not_found]
- [methods] pewlib library for import and export: [evidence span withheld — anchor status: not_found]
