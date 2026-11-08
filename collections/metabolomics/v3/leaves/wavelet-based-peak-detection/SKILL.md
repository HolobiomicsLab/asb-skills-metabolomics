---
name: wavelet-based-peak-detection
description: Use when raw 1D NMR spectra plus a ppm vector need to be converted into
  peak lists before grouping into a feature/sample/value matrix, especially when binning
  would lose multiplet information.
when_to_use_negative:
- Input is already a binned/bucketed feature table or a processed peak list — wavelet
  peak detection operates on raw spectra
- Only raw-spectra alignment and quantitation are needed; the older speaq (<= 1.2.3)
  CluPA alignment / BW-ratio workflow is the appropriate path
- The goal is concentration estimation of known metabolites, which BATMAN targets
  rather than peak-based profiling
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: Provides the wavelet-based peak-picking method that summarizes raw 1D NMR
    spectra
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributed implementation; getWaveletPeaks() performs the peak detection
    step
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment for the speaq package and the peak-detection workflow
- name: MetaboAnalyst
  role: Complementary downstream analysis tool whose lower-resolution grouping (245
    vs 677 features) the wavelet peak lists improve upon
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:8a4f17c5da96e67a82d82cf7acc190c73c8559774e9a9aac2eae0432a0e6822a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - MetaboAnalyst
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# wavelet-based-peak-detection

## Summary

speaq 2.0's wavelet (Mexican hat) peak detection summarizes raw 1D NMR spectra into picked-peak lists with minimal information loss and no user interaction, replacing binning/bucketing and avoiding baseline correction and smoothing.

## When to use

Use when raw 1D NMR spectra plus a ppm vector need to be converted into peak lists before grouping into a feature/sample/value matrix, especially when binning would lose multiplet information.

## When NOT to use

- Input is already a binned/bucketed feature table or a processed peak list — wavelet peak detection operates on raw spectra
- Only raw-spectra alignment and quantitation are needed; the older speaq (<= 1.2.3) CluPA alignment / BW-ratio workflow is the appropriate path
- The goal is concentration estimation of known metabolites, which BATMAN targets rather than peak-based profiling

## Inputs

- Raw 1D NMR spectra as a numeric matrix (samples × spectral points)
- Corresponding ppm vector for the spectra
- Class/group factor for the samples (needed for downstream differential analysis)

## Outputs

- Picked-peak list/data frame (peaks per spectrum with ppm positions)
- Grouped peaks ready for BuildFeatureMatrix()
- Feature/sample/peak-value matrix (e.g. 677 features on the onion intake in mice dataset)

## How to apply

Run the speaq R package's getWaveletPeaks() on a spectra matrix and its corresponding ppm vector, using the Mexican hat wavelet to detect peaks directly in the raw spectra; this eliminates the need for baseline correction and smoothing because the wavelet transform itself summarizes the peaks. The default mode is parallel peak detection. Because speaq 2.0 picks peaks and then groups them, it avoids the binning step whose summed bins intrinsically lose information and make bin boundaries hard to define in crowded spectra. The output peak list is then grouped with a moving-window procedure (e.g. 0.03 ppm window stepped in 0.015 ppm increments) and converted into a feature/sample/peak-value matrix, which preserves multiplet information that tools like MetaboAnalyst's grouping collapse (677 vs 245 features on the onion intake in mice dataset). Correct application is judged by whether downstream differential analysis (Benjamini-Hochberg FDR) yields more, not fewer, significant features than binned or MetaboAnalyst-grouped alternatives.

## Related tools

- **speaq 2.0** (Provides the wavelet-based peak-picking method that summarizes raw 1D NMR spectra) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributed implementation; getWaveletPeaks() performs the peak detection step) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment for the speaq package and the peak-detection workflow)
- **MetaboAnalyst** (Complementary downstream analysis tool whose lower-resolution grouping (245 vs 677 features) the wavelet peak lists improve upon)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = as.matrix(Winedata$spectra), X.ppm = as.numeric(Winedata$ppm))
```

## Evaluation signals

- Peak detection runs in the default parallel mode without requiring baseline correction or smoothing of the spectra beforehand
- The subsequent moving-window grouping (0.03 ppm window, 0.015 ppm steps) yields the expected 677-feature matrix on the onion intake in mice data, versus 245 for MetaboAnalyst grouping
- The picked peaks retain multiplet structure that summed bins would destroy, allowing ppm queries against databases like HMDB
- Downstream Benjamini-Hochberg FDR differential analysis flags highly significant features, each with a counterpart detected in the speaq 2.0 pipeline
- The feature matrix dimensions are greatly reduced relative to the raw spectra while showing similar or better performance than alignment-based alternatives

## Limitations

- Peak filling may not fully mitigate missed peaks in severely shifted spectra; the optional SilhouetR function can detect grouping problems via per-group silhouette values
- The absence of standardized/automated NMR workflows elsewhere introduces bias and lowers reproducibility — the wavelet pipeline's value depends on feeding it raw spectra
- speaq 2.0 is aimed at 1D NMR spectra; it complements rather than replaces LC-MS tools like XCMS

## Evidence

- [abstract] uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction: "uses wavelets to efficiently summarize the raw spectra with minimal information loss or user interaction"
- [intro] Mexican hat wavelet, no baseline correction or smoothing: "The basic proposition of speaq 2.0 is to use wavelets to summarize the peaks within the spectra."
- [abstract] peak picking followed by grouping avoids binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README example: getWaveletPeaks call: [evidence span withheld — anchor status: not_found]
- [supplementary] SilhouetR detects grouping problems: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
