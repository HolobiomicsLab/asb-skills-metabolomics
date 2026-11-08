---
name: ppm-tolerance-feature-comparison
description: Use when comparing feature tables produced by different LC-MS preprocessing
  tools and you need to quantify whether reported m/z values are distinguishable at
  a stated instrument mass resolution (ppm tolerance), e.g. auditing XCMS/MZmine output
  against asari's mass-track granularity.
when_to_use_negative:
- Input is raw centroided mzML data rather than an already-produced feature table
  — feature detection/processing must happen first
- Data are low-resolution or from non-Orbitrap platforms where asari's mass-track
  settings are not optimized and ppm-based distinctness is not meaningful
- Goal is comparing annotation or quantification accuracy rather than m/z distinguishability
  at instrument resolution
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: asari
  role: Reference feature table generator whose mass-track alignment yields features
    with mSelectivity close to 1; its preferred_Feature_table.tsv is the benchmark
    output
  repo: https://github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Widely used comparison tool whose feature table is evaluated for poor mSelectivity
- name: MZmine
  role: Widely used comparison tool whose feature table is evaluated for poor mSelectivity
- name: Python
  role: Implementation language for computing mSelectivity scores and plotting distributions
    (requires Python 3.8+)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:1102df79ae8613565364c4558c2106ea867ab924608cb14fde7666f6c3fce8cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - MZmine
  - Python
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ppm-tolerance-feature-comparison

## Summary

A benchmarking method that scores m/z features from different LC-MS processing tools (XCMS, MZmine, asari) by mSelectivity — the distinctness of each feature's m/z relative to nearest neighbors within the instrument's ppm resolution (e.g. 5 ppm on HZV029) — after rounding m/z to the 3rd decimal place, to expose mass-alignment deficiencies.

## When to use

Use when comparing feature tables produced by different LC-MS preprocessing tools and you need to quantify whether reported m/z values are distinguishable at a stated instrument mass resolution (ppm tolerance), e.g. auditing XCMS/MZmine output against asari's mass-track granularity.

## When NOT to use

- Input is raw centroided mzML data rather than an already-produced feature table — feature detection/processing must happen first
- Data are low-resolution or from non-Orbitrap platforms where asari's mass-track settings are not optimized and ppm-based distinctness is not meaningful
- Goal is comparing annotation or quantification accuracy rather than m/z distinguishability at instrument resolution

## Inputs

- XCMS feature table for the HZV029 dataset (m/z, rt, sample intensities)
- MZmine feature table for the HZV029 dataset
- asari preferred_Feature_table.tsv (reference features from mass-track processing)
- Instrument mass-resolution tolerance in ppm (5 ppm for HZV029)

## Outputs

- Per-feature mSelectivity scores for XCMS, MZmine, and asari features
- Side-by-side mSelectivity distributions (histogram/violin) reproducing the Fig. 1d pattern

## How to apply

Load the XCMS and MZmine feature tables for the study dataset (e.g. HZV029) and round each feature's m/z value to the 3rd decimal place to mimic asari's mass-track m/z granularity. For each feature, compute the mSelectivity score as the distinctness of its m/z measurement relative to the closest neighboring m/z values within the 5 ppm mass-resolution tolerance. As a reference, compute the same mSelectivity scores for features in asari's preferred_Feature_table.tsv, since asari performs mass alignment first into mass tracks and composite mass tracks. Plot the mSelectivity distributions of XCMS, MZmine, and asari side by side (histograms or violin plots) to reproduce the Fig. 1d contrast. The paper's rationale is that inconsistency among tools is attributable to deficiencies in mass alignment and lack of feature quality controls, so mSelectivity is a direct quality metric on the consensus m/z values reported after cross-sample correspondence.

## Related tools

- **asari** (Reference feature table generator whose mass-track alignment yields features with mSelectivity close to 1; its preferred_Feature_table.tsv is the benchmark output) — https://github.com/shuzhao-li-lab/asari
- **XCMS** (Widely used comparison tool whose feature table is evaluated for poor mSelectivity)
- **MZmine** (Widely used comparison tool whose feature table is evaluated for poor mSelectivity)
- **Python** (Implementation language for computing mSelectivity scores and plotting distributions (requires Python 3.8+))

## Examples

```
asari process --mode pos --input mydir/projectx_dir  # then score mSelectivity of preferred_Feature_table.tsv vs XCMS/MZmine tables within 5 ppm after rounding m/z to 3 decimals
```

## Evaluation signals

- XCMS and MZmine mSelectivity distributions show poor (low) distinctness values, while the asari distribution clusters near 1 (perfect compliance of mass resolution), matching the Fig. 1d contrast
- All asari-derived features have mSelectivity close to 1 after m/z rounding to the 3rd decimal place
- The count of features with poor mSelectivity is very large in XCMS/MZmine output, inconsistent with the 5 ppm instrument resolution
- Feature counts per tool are comparable so the distribution contrast reflects mass alignment quality, not differing feature-detection sensitivity

## Limitations

- Inconsistency among tools is attributed to deficiencies in mass alignment and controls of feature quality, so mSelectivity differences reflect alignment strategy, not absolute feature correctness
- ppm is only a practical approximation; a 2nd order polynomial model may be required to cover the full m/z range more accurately
- Asari is mostly tested on Orbitrap platforms and does not work with low-resolution data, limiting the reference distribution to high-resolution acquisitions

## Evidence

- [abstract] Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [intro] When peaks are later aligned cross samples (i.e., correspondence), the m/z values vary slightly in each sample and the algorithm must ensure the correct peaks are grouped and report a consensus m/z: "When peaks are later aligned cross samples (i.e., correspondence), the m/z values vary slightly in each sample and the algorithm must ensure the correct peaks are grouped"
- [methods] From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid.: "From each mzML data file, mass tracks of unique m/z values are extracted, then aligned into a MassGrid."
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [readme] The recommended feature table is `preferred_Feature_table.tsv`.: "The recommended feature table is `preferred_Feature_table.tsv`."
