---
name: spectral-alignment-method-benchmarking
description: 'Decision support for choosing a 1H NMR spectra summarization or alignment
  method: when shifted spectra need objective comparison of peak-based vs alignment-plus-binning
  workflows using classification performance (ROC/PRC).'
when_to_use_negative:
- The input is already a processed feature table or peak matrix — benchmarking of
  spectra summarization methods requires raw spectra as the common starting point.
- Real experimental data without known ground-truth class labels is used and the goal
  is only alignment quality, not classification performance comparison.
- Only one summarization method is of interest — this skill is specifically a comparative
  benchmark across competing workflows.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: 'Peak-based processing method under evaluation: wavelet peak picking followed
    by grouping, avoiding the binning step'
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R package providing getWaveletPeaks, PeakGrouper, BuildFeatureMatrix, and
    CluPA alignment used in the benchmark
  repo: https://cran.r-project.org/package=speaq
- name: CluPA
  role: Competing spectral alignment algorithm whose aligned spectra are subsequently
    binned for comparison
- name: icoshift
  role: Competing spectral alignment algorithm whose aligned spectra are subsequently
    binned for comparison
- name: R
  role: Environment in which the speaq R package and the full benchmark workflow are
    executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:be8a50f61f13d2759a8c9dd43cb00257156489a890717ffd9602b1f0c3930382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - CluPA
  - R
  - icoshift
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-alignment-method-benchmarking

## Summary

Benchmark strategy for 1H NMR spectra summarization/alignment workflows on a simulated case-vs-control dataset, comparing peak-based speaq 2.0 (wavelet peak finding + grouping, no binning) against binning of raw spectra and binning after CluPA or icoshift alignment, scored by ROC and Precision-Recall curves.

## When to use

Decision support for choosing a 1H NMR spectra summarization or alignment method: when shifted spectra need objective comparison of peak-based vs alignment-plus-binning workflows using classification performance (ROC/PRC).

## When NOT to use

- The input is already a processed feature table or peak matrix — benchmarking of spectra summarization methods requires raw spectra as the common starting point.
- Real experimental data without known ground-truth class labels is used and the goal is only alignment quality, not classification performance comparison.
- Only one summarization method is of interest — this skill is specifically a comparative benchmark across competing workflows.

## Inputs

- Reference 1H NMR spectra of two HMDB metabolites (HMDB0000440, HMDB0001336) from the HMDB database
- Simulated 1H NMR spectra matrix (20 spectra: 10 case, 10 control) with noise, baseline distortion, and random peak shifts
- ppm axis vector and class factor labels for the spectra

## Outputs

- Feature/sample/peak-value matrix per method (speaq 2.0 peak-based, raw binning, CluPA+binning, icoshift+binning)
- ROC curves for each method overlaid in ROC_PRC_curves.png
- Precision-Recall curves for each method overlaid in ROC_PRC_curves.png
- Ranked comparison showing whether peak-based speaq 2.0 performs similar or better than alignment-plus-binning alternatives

## How to apply

Simulate a case-vs-control 1H NMR dataset from reference spectra of known metabolites (here HMDB0000440 and HMDB0001336 from HMDB), generating 20 spectra (10 case, 10 control) with added noise, baseline distortion, and random peak shifts as prescribed in the paper's S2 Appendix. Process the same spectra with the competing workflows: (1) the speaq 2.0 peak-based pipeline using Mexican hat wavelet peak picking followed by grouping (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix), avoiding the binning step; (2) raw binning without alignment as baseline; (3) CluPA alignment followed by binning; (4) icoshift alignment followed by binning. From each resulting feature/sample/peak-value matrix, compute ROC curves and Precision-Recall curves for the case-vs-control classification task and overlay all curves (ROC_PRC_curves.png). The rationale is that alignment algorithms are not always optimal for severely shifted spectra, so the simulation with known true class labels gives a ground truth for judging which summarization strategy preserves discriminative information.

## Related tools

- **speaq 2.0** (Peak-based processing method under evaluation: wavelet peak picking followed by grouping, avoiding the binning step) — https://github.com/beirnaert/speaq
- **speaq R package** (R package providing getWaveletPeaks, PeakGrouper, BuildFeatureMatrix, and CluPA alignment used in the benchmark) — https://cran.r-project.org/package=speaq
- **CluPA** (Competing spectral alignment algorithm whose aligned spectra are subsequently binned for comparison)
- **icoshift** (Competing spectral alignment algorithm whose aligned spectra are subsequently binned for comparison)
- **R** (Environment in which the speaq R package and the full benchmark workflow are executed)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- The speaq 2.0 peak-based method must show ROC/PRC performance similar to or better than binning of raw shifted spectra and binning after CluPA or icoshift alignment.
- Each method must yield a complete feature/sample/peak-value matrix for the same 20 simulated spectra (10 case, 10 control).
- All four methods' ROC and Precision-Recall curves must be computed from case-vs-control classification of the feature matrices and plotted together in ROC_PRC_curves.png.
- Simulated spectra must incorporate noise, baseline distortion, and random peak shifts as prescribed in S2 Appendix, built from the two HMDB metabolite reference spectra.
- The simulation should include severe enough peak shifts that alignment-plus-binning methods can underperform, exercising the known limitation of spectral alignment on severely shifted spectra.

## Limitations

- Spectral alignment algorithms are not always optimal when dealing with severely shifted spectra, which affects subsequent processing steps such as binning.
- The benchmark relies on simulated data from only two HMDB metabolites; alignment quality can be misestimated (e.g., the speaq 2.0 algorithm may delete one of the large peaks in the grouping step, a problem usually mitigated by the peak filling step but not always).
- The optional SilhouetR function (silhouette values per group) can detect grouping problems, but detection is not mitigation.

## Evidence

- [supplementary] The speaq 2.0 method, based on finding peaks and subsequently grouping, performs similar or better compared to the other methods: "The speaq 2.0 method, based on finding peaks and subsequently grouping, performs similar or better compared to the other methods"
- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] Simulated data based comparison of speaq 2.0 vs alignment algorithms.: "Simulated data based comparison of speaq 2.0 vs alignment algorithms."
- [supplementary] The results of spectral alignment algorithms is not always optimal when dealing with severely shifted spectra.: "The results of spectral alignment algorithms is not always optimal when dealing with severely shifted spectra."
- [readme] # peak detection (the default mode is parallel)
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)
# peak grouping
groups <- PeakGrouper(Y.peaks = peaks): "# peak detection (the default mode is parallel)
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)"
