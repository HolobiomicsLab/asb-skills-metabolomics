---
name: peak-intensity-sqrt-transformation
description: Use when preprocessing MS/MS spectra (from mzML, mzXML, or MGF peak files)
  for cosine-similarity comparison or clustering, and dominant high-intensity peaks
  would otherwise skew the similarity measure.
when_to_use_negative:
- When clustering metabolomics or top-down data where the bottom-up proteomics preprocessing
  defaults (including this scaling and the 5-peak/250 m/z-range filters) likely need
  adjustment
- When a downstream similarity measure other than cosine similarity is used, since
  the scaling method's benefit (de-emphasizing dominant peaks) is tied to cosine-based
  comparison and influences the eps threshold
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool whose preprocessing pipeline exposes the intensity
    `scaling` option (square root, logarithm, rank, or none) applied prior to feature
    hashing
  repo: https://github.com/bittremieux/falcon
- name: spectrum_utils
  role: Library used to implement spectrum preprocessing, including the square-root
    transformation of peak intensities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:9f738e3c20afd3010ccca001d33053799735f041dc391ca8b936fd8abfd69b49
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - NumPy
  - Numba
  - Pyteomics
  - Python
  - SciPy
  - falcon
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# peak-intensity-sqrt-transformation

## Summary

Square-root transform MS/MS peak intensities during spectrum preprocessing to de-emphasize overly dominant peaks before vectorization and cosine-similarity-based clustering, as done in the falcon spectrum clustering pipeline.

## When to use

Use when preprocessing MS/MS spectra (from mzML, mzXML, or MGF peak files) for cosine-similarity comparison or clustering, and dominant high-intensity peaks would otherwise skew the similarity measure.

## When NOT to use

- When clustering metabolomics or top-down data where the bottom-up proteomics preprocessing defaults (including this scaling and the 5-peak/250 m/z-range filters) likely need adjustment
- When a downstream similarity measure other than cosine similarity is used, since the scaling method's benefit (de-emphasizing dominant peaks) is tied to cosine-based comparison and influences the eps threshold

## Inputs

- Preprocessed MS/MS spectra (peaks after precursor-ion and noise-peak removal, restricted to the 50 most intense peaks) from mzML, mzXML, or MGF peak files

## Outputs

- Intensity-transformed spectra in which peak intensities are replaced by their square roots, ready for 0.05 m/z binning and feature hashing into length-800 dense vectors

## How to apply

After filtering peaks (removing the precursor-ion peak, low-intensity noise peaks below 10% of the base peak, and restricting to the 50 most intense peaks), apply a square-root transformation to the remaining peak intensities. The rationale is to de-emphasize overly dominant peaks so that cosine similarity between spectra better reflects shared fragment content rather than a few very intense peaks. In falcon this scaling is controlled by the `scaling` option (square root, logarithm, rank, or no scaling; default is no scaling, with square root scaling often giving good results) and is implemented via spectrum_utils (e.g., spectrum_utils 0.3.5). Critically, the scaling method interacts with the clustering threshold: the chosen scaling influences the cosine distance `eps` used by DBSCAN (falcon's ~1% incorrectly-clustered-spectra result used cosine distance 0.05), so re-tune `eps` (typically 0.05–0.15 for pure clusters) when switching scaling. For metabolomics or top-down data, revisit this together with `min_peaks`/`min_mz_range` defaults (5 peaks, 250 m/z range).

## Related tools

- **falcon** (Spectrum clustering tool whose preprocessing pipeline exposes the intensity `scaling` option (square root, logarithm, rank, or none) applied prior to feature hashing) — https://github.com/bittremieux/falcon
- **spectrum_utils** (Library used to implement spectrum preprocessing, including the square-root transformation of peak intensities)

## Examples

```
falcon peak/*.mzml falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10  # scaling option: 'sqrt' (spectrum_utils preprocessing) ; see falcon -h
```

## Evaluation signals

- Transformed intensities should equal the square roots of the original peak intensities (monotone, compressive transform; all values remain non-negative)
- The transformation is applied after peak filtering: only spectra with at least 5 peaks spanning at least 250 m/z survive to this stage
- Cosine distances computed on square-root-scaled spectra require a re-tuned DBSCAN eps (falcon's ~1% incorrectly clustered spectra corresponded to cosine distance 0.05); verify cluster purity at the chosen eps
- Hashed vectors derived from transformed spectra should approximately conserve the Euclidean norm of the original spectra (feature-hashing property), with shape (n_spectra, 800) and finite values

## Limitations

- The scaling method influences the cosine threshold `eps`, so changing to or from square-root scaling invalidates previously tuned clustering thresholds
- The influence of MS/MS spectrum preprocessing on various similarity measures was not evaluated; alternative similarity methods (probabilistic, fragment rarity, machine-learning-derived) might interact differently with intensity scaling
- Default settings including scaling are intended for bottom-up proteomics data; metabolomics and top-down data likely need adjusted settings

## Evidence
