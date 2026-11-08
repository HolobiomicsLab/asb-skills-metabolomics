---
name: polyphenol-metabolite-spectral-grouping
description: Use when an LC–MS/MS dataset (imported as .mgf) contains a feature matched
  to a known reference standard (e.g., Procyanidin-A2) and the goal is to characterize
  the local ms2deepscore network topology and fragmentation-pattern overlaps around
  it for structural annotation hypotheses.
when_to_use_negative:
- Datasets much larger than ~5000 features — the paper notes the dashboard is expected
  to scale well to 5000 features but may face difficulties for larger datasets and
  liberal settings
- Spectra/metabolites poorly covered by or unrelated to the ms2deepscore training
  data, since ms2deepscore similarities are only expected to work well within that
  coverage and no coverage check is performed
- When the goal is fully automatic compound annotation rather than interactive, manual
  exploration of local network topology and fragmentation overlaps (specXplore supports
  manual hypothesis generation, not automated ID)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: specXplore
  role: Interactive Python dashboard providing t-SNE overview, partial network drawings,
    similarity heatmaps, and fragmentation overview maps (fragmaps) for thresholded
    local exploration around the reference-standard feature
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity scores used as
    the quantitative backbone for thresholding (e.g., 0.7) and defining the dense
    neighborhood
- name: matchms
  role: Imports .mgf MS/MS spectral data into Python and provides Spectrum objects
    for retrieving (m/z, intensity) peak tuples and fragment comparisons
- name: MS2Query
  role: Pretrained models used in the specXplore importing pipeline for machine-learning
    scores and local library search during preprocessing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:a6988a788e2a780bb4b4e25a6e6f1670d93cf54887dc7c14f4368c6776a51aeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# polyphenol-metabolite-spectral-grouping

## Summary

Uses the specXplore interactive dashboard to isolate and characterize the densely connected local network neighborhood around a known reference-standard feature (e.g., Procyanidin-A2 in a urine polyphenol exposome LC–MS/MS dataset), quantifying fragment-ion overlap and spectral complexity as starting points for MS/MS structural hypothesis generation and manual annotation.

## When to use

Use when an LC–MS/MS dataset (imported as .mgf) contains a feature matched to a known reference standard (e.g., Procyanidin-A2) and the goal is to characterize the local ms2deepscore network topology and fragmentation-pattern overlaps around it for structural annotation hypotheses.

## When NOT to use

- Datasets much larger than ~5000 features — the paper notes the dashboard is expected to scale well to 5000 features but may face difficulties for larger datasets and liberal settings
- Spectra/metabolites poorly covered by or unrelated to the ms2deepscore training data, since ms2deepscore similarities are only expected to work well within that coverage and no coverage check is performed
- When the goal is fully automatic compound annotation rather than interactive, manual exploration of local network topology and fragmentation overlaps (specXplore supports manual hypothesis generation, not automated ID)

## Inputs

- specXplore session data object or outputs from the specXplore importing pipeline (t-SNE embedding, pairwise similarity matrices)
- .mgf file of MS/MS spectra with a 'feature_id' metadata key per feature
- ms2deepscore pairwise similarity matrix (values in [0,1])
- Pretrained MS2Query model files (positive or negative mode) used by the importing pipeline

## Outputs

- Isolated local network neighborhood around the reference-standard feature (e.g., Procyanidin-A2 node) at a chosen similarity threshold
- Pairwise fragment-ion overlap statistics among spectra in the region (binned, sorted m/z comparison)
- Summary statistics of the within-region ms2deepscore similarity distribution
- Aggregate table such as procyanidin_a2_region_overlap_stats.csv with region size, overlap statistics, and similarity summaries

## How to apply

Load the precomputed ms2deepscore pairwise similarity matrix and t-SNE embedding produced by the specXplore importing pipeline (which imports .mgf spectra via matchms and computes ms2deepscore, modified cosine, and spec2vec similarities). Locate the reference-standard feature node on the 2D t-SNE overview, then threshold the ms2deepscore similarity matrix (default pairwise threshold of 0.7, i.e., a minimum fragmentation overlap) with adjustable top-K neighbor limits and maximal molecular family sizes to extract the dense local neighborhood. Retrieve (m/z, intensity) peak tuples for all spectra in the neighborhood from the .mgf data using matchms, and compute pairwise fragment-ion overlap fractions by comparing binned and sorted m/z fragments in the manner of the specXplore fragmap construction. Summarize the ms2deepscore similarity distribution (bounded [0,1]) within the region; a much higher spectral complexity and characteristic fragment overlap among group members supports structural hypothesis generation alongside local topology and joined-in metadata.

## Related tools

- **specXplore** (Interactive Python dashboard providing t-SNE overview, partial network drawings, similarity heatmaps, and fragmentation overview maps (fragmaps) for thresholded local exploration around the reference-standard feature) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity scores used as the quantitative backbone for thresholding (e.g., 0.7) and defining the dense neighborhood)
- **matchms** (Imports .mgf MS/MS spectral data into Python and provides Spectrum objects for retrieving (m/z, intensity) peak tuples and fragment comparisons)
- **MS2Query** (Pretrained models used in the specXplore importing pipeline for machine-learning scores and local library search during preprocessing)

## Examples

```
conda create --name specxplore_environment python=3.8; conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- The output table (e.g., procyanidin_a2_region_overlap_stats.csv) contains fragment-overlap and similarity-distribution fields with a nonzero feature count
- The computed overlap statistics reproduce the characteristic fragmentation overlap the paper reports for the Procyanidin-A2 feature group, with experimental spectra markedly more complex than the reference
- The isolated neighborhood corresponds to a densely connected region on the t-SNE overview at the chosen ms2deepscore threshold (default 0.7 minimum fragmentation overlap)
- Within-region ms2deepscore similarity values are bounded in [0,1] and show an elevated distribution consistent with a coherent fragmentation-pattern group
- K-medoid clustering of the region shows good agreement with the t-SNE projection and topological insights, as the paper reports

## Limitations

- The current specXplore version works on macOS and Linux but fails on Windows; additionally, ms2deepscore on macOS arm64 may produce unreliable similarity predictions (matchms/ms2deepscore issue 199) without any error or warning
- The paper demonstrates the Procyanidin-A2 dense-feature example on the wheat LC–MS/MS data; the urine dataset example is deferred to Supporting Information Section S3.2
- No tool version numbers are stated; no online hosted version exists (installation is local only), and clustering consistency across different k-medoid k settings is not enforced

## Evidence

- [results] Exploring the densely connected feature area around the Procyanidin-A2 corresponding feature reveals the characteristic fragment ion overlap and the much higher complexity of the experimental spectra: [evidence span withheld — anchor status: not_found]
- [abstract] specXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone, while allowing fast changes of threshold and connectivity limitation settings: "SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone, while allowing fast changes of threshold and connectivity limitation settings"
- [discussion] the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [methods] Spectral data are imported into Python using matchms, and basic specXplore data processing is performed: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [readme] SpecXplore is a python dashboard tool for adjustable LC-MS/MS spectral data exploration. It joins a t-SNE embedding that serves as an overview representation of mass spectral similarities based on ms2deepscore: [evidence span withheld — anchor status: external_doc]
