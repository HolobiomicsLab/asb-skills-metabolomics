---
name: grid-search-gradient-descent-optimization
description: 'Use when aligning MRM/SRM chromatograms across samples: EIC peak tables
  show retention time shifts between acquisitions, and per-sample/per-transition-group
  drift values must be estimated to enable peak alignment and automatic EIC grouping.'
when_to_use_negative:
- Input is already an aligned feature table or a peak-picked matrix with no raw EIC
  traces — there are no continuous chromatograms to shift and score
- Data are DDA/DIA (untargeted) spectra rather than MRM EICs, where no per-transition-group
  reference chromatogram can be defined
- A quantitative comparison of analysis software output is needed — MRMPro's authors
  note it 'relies on classical algorithms', and advanced peak-extraction algorithms
  are only planned future work
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MRMPro
  role: Web-based MRM platform whose retention time drift estimation algorithm implements
    the grid-search + gradient-descent optimization of cosine similarity; its drift
    estimates feed peak alignment and automatic EIC grouping
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Benchmark tool against which MRMPro's quantification accuracy (which depends
    on this drift estimation) was compared
- name: AirdPro
  role: Converts vendor MRM raw files (including Wiff2) into the Aird format that
    MRMPro reads; supports MRM/SRM acquisition methods
  repo: https://github.com/CSi-Studio/AirdPro
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:2a159be6b4ce5766f5f7c8ea09490274d1cef4576a5fc3739721fe0df540a7e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - AirdPro
  - MRMPro
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# grid-search-gradient-descent-optimization

## Summary

A two-stage 1-D optimization scheme from MRMPro (BMC Bioinformatics, 2024) that estimates retention time (RT) drifts between MRM samples by first coarsely scanning candidate RT shifts with grid search, then refining the optimum via gradient descent, both maximizing the cosine similarity between a sample's extracted-ion chromatogram (EIC) and a reference EIC.

## When to use

Use when aligning MRM/SRM chromatograms across samples: EIC peak tables show retention time shifts between acquisitions, and per-sample/per-transition-group drift values must be estimated to enable peak alignment and automatic EIC grouping.

## When NOT to use

- Input is already an aligned feature table or a peak-picked matrix with no raw EIC traces — there are no continuous chromatograms to shift and score
- Data are DDA/DIA (untargeted) spectra rather than MRM EICs, where no per-transition-group reference chromatogram can be defined
- A quantitative comparison of analysis software output is needed — MRMPro's authors note it 'relies on classical algorithms', and advanced peak-extraction algorithms are only planned future work

## Inputs

- Extracted-ion chromatogram (EIC) peak tables for all MRM samples, as a pandas DataFrame
- Transition group identifiers defining which EICs are compared
- Set of candidate RT shift values for the grid search

## Outputs

- rt_drift_estimates.csv with one row per sample per transition group, columns: sample, transition group, estimated RT drift
- Per-drift cosine similarity scores (before and after gradient descent) for acceptance checks

## How to apply

Load the EIC peak tables for all MRM samples into a pandas DataFrame. Select one reference EIC per transition group by computing the product of peak count and maximum peak height per sample and taking the arg-max sample as reference, so the most information-rich chromatogram anchors the alignment. For each non-reference sample, run a coarse grid search over candidate RT shifts, scoring each shift by the cosine similarity between the shifted chromatogram and the reference chromatogram after calibration. Refine the best grid-search shift with gradient descent on the RT shift parameter, iterating only while cosine similarity continues to increase. Collect the converged drift per sample and transition group into a table with columns sample, transition group, and estimated RT drift. The rationale is that cosine similarity is a smooth objective over the shift parameter, so grid search avoids local minima while gradient descent converges to the precise drift that MRMPro then uses for peak alignment and automatic EIC grouping.

## Related tools

- **MRMPro** (Web-based MRM platform whose retention time drift estimation algorithm implements the grid-search + gradient-descent optimization of cosine similarity; its drift estimates feed peak alignment and automatic EIC grouping) — https://github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Benchmark tool against which MRMPro's quantification accuracy (which depends on this drift estimation) was compared)
- **AirdPro** (Converts vendor MRM raw files (including Wiff2) into the Aird format that MRMPro reads; supports MRM/SRM acquisition methods) — https://github.com/CSi-Studio/AirdPro

## Examples

```
best_shift = max(candidate_shifts, key=lambda d: cosine(ref, np.interp(rt - d, rt, eic)))  # grid search; then refine by gradient descent while cosine similarity increases, and write rt_drift_estimates.csv
```

## Evaluation signals

- Output table contains exactly one drift estimate per input sample per transition group (row-count × group-count completeness check)
- Each reported drift attains a cosine similarity no lower than its grid-search initialization — the acceptance check for this reconstruction, as the paper reports no numeric threshold for drift quality
- Gradient descent iterations strictly increase cosine similarity and stop only when similarity no longer improves
- Reference EIC per group equals the arg-max of (peak count × maximum peak height), verifiable by recomputing the selection from the input DataFrame
- Downstream peak alignment and automatic EIC grouping succeed using the estimated drifts, consistent with the paper's reported use of the estimates

## Limitations

- MRMPro 'relies on classical algorithms to provide a stable and precise analytical framework' rather than advanced analytical algorithms; incorporation of advanced algorithms is only planned future work
- The paper reports no numeric threshold for acceptable drift quality, so acceptance must be judged by the monotonic-similarity invariant rather than a fixed cutoff
- Peak extraction accuracy itself is not improved by this method; the authors state improvement of peak extraction accuracy is deferred to future work

## Evidence

- [abstract] RT drift estimation is based on chromatogram similarity: "we proposed a retention time drift estimation algorithm based on similarity of chromatograms"
- [abstract] Drift estimates feed alignment and EIC grouping: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [discussion] Classical algorithms underpin the method: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
- [readme] AirdPro supports MRM acquisition for input conversion: [evidence span withheld — anchor status: external_doc]
