---
name: cross-tool-feature-mapping-comparison
description: Use when you need to benchmark a peak-picking/grouping workflow against
  an established tool like MetaboAnalyst on the same raw NMR spectra, and must quantify
  differences in feature resolution and overlap of statistically significant features.
when_to_use_negative:
- Input is already a feature table produced by a single tool and no cross-tool benchmarking
  comparison is required
- The NMR data require binning/bucketing as the mandated summarization approach rather
  than peak picking plus grouping (speaq 2.0 explicitly avoids the binning step)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: speaq 2.0
  role: Wavelet-based peak finding and moving-window grouping of the onion NMR data
    to build the high-resolution 677-feature matrix and run differential analysis
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R package distributing the speaq 2.0 functions (getWaveletPeaks, PeakGrouper,
    BuildFeatureMatrix) used to construct the comparison feature matrix
  repo: https://cran.r-project.org/package=speaq
- name: MetaboAnalyst
  role: Established NMR analysis tool applied to the same onion peak list with matching
    moving-window grouping parameters to produce the 245-feature comparison matrix
- name: R
  role: Environment in which the speaq 2.0 workflow, differential analysis, and cross-mapping
    are executed
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:260bd7df317565f8bbc945693db93a28a8d59aeec1f604e017b0c4e168f04a37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - MetaboAnalyst
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-tool-feature-mapping-comparison

## Summary

A benchmarking protocol for comparing feature matrices produced by different NMR peak-grouping implementations — here speaq 2.0 (677 features) versus MetaboAnalyst (245 features) on the onion intake in mice dataset — by running identical differential analyses (Benjamini-Hochberg FDR) on both matrices and cross-mapping the highly significant features.

## When to use

Use when you need to benchmark a peak-picking/grouping workflow against an established tool like MetaboAnalyst on the same raw NMR spectra, and must quantify differences in feature resolution and overlap of statistically significant features.

## When NOT to use

- Input is already a feature table produced by a single tool and no cross-tool benchmarking comparison is required
- The NMR data require binning/bucketing as the mandated summarization approach rather than peak picking plus grouping (speaq 2.0 explicitly avoids the binning step)

## Inputs

- Publicly available onion intake in mice 1D NMR peak data (peak lists / spectra matrix with ppm vector, as in Winedata-style R objects)
- MetaboAnalyst moving-window grouping configuration (0.03 ppm window, 0.015 ppm steps)
- Dietary group labels (class factor) for the mice samples

## Outputs

- speaq 2.0 peak/sample/value feature matrix with 677 features
- MetaboAnalyst feature matrix with 245 features
- Benjamini-Hochberg FDR-corrected differential analysis results for both matrices
- Cross-mapping table (Table 2) linking each highly significant MetaboAnalyst feature to at least one highly significant speaq 2.0 counterpart

## How to apply

Load the publicly available onion intake in mice 1D NMR peak data into R and run the speaq 2.0 workflow (Mexican hat wavelet peak picking via getWaveletPeaks, then PeakGrouper grouping, avoiding binning) using the moving-window procedure with a 0.03 ppm window stepped in 0.015 ppm increments to obtain the 677-feature peak/sample/value matrix. Process the identical peak list in MetaboAnalyst with the same moving-window grouping parameters (0.03 ppm window, 0.015 ppm steps) to obtain its 245-feature matrix — the lower resolution arises because MetaboAnalyst sums close peaks together, destroying multiplet information. Run Benjamini-Hochberg FDR-corrected differential abundance analysis on both matrices to flag highly significant features between the dietary groups. Cross-map each highly significant MetaboAnalyst feature onto the highly significant speaq 2.0 features by ppm proximity and compile the coverage comparison (e.g. Table 2), recording the 677 vs 245 feature-resolution difference as the principal explanatory finding.

## Related tools

- **speaq 2.0** (Wavelet-based peak finding and moving-window grouping of the onion NMR data to build the high-resolution 677-feature matrix and run differential analysis) — https://github.com/beirnaert/speaq
- **speaq R package** (R package distributing the speaq 2.0 functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) used to construct the comparison feature matrix) — https://cran.r-project.org/package=speaq
- **MetaboAnalyst** (Established NMR analysis tool applied to the same onion peak list with matching moving-window grouping parameters to produce the 245-feature comparison matrix)
- **R** (Environment in which the speaq 2.0 workflow, differential analysis, and cross-mapping are executed) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec=as.matrix(onion$spectra), X.ppm=onion$ppm); groups <- PeakGrouper(Y.peaks=peaks); Features <- BuildFeatureMatrix(Y.data=groups)
```

## Evaluation signals

- The speaq 2.0 feature matrix must contain 677 features while the MetaboAnalyst matrix contains 245 features on the same onion peak list
- Every highly significant MetaboAnalyst feature must map to at least one highly significant speaq 2.0 counterpart in the cross-mapping table
- Both grouping runs must use identical moving-window parameters: 0.03 ppm window with 0.015 ppm steps
- Differential analysis on both matrices must be Benjamini-Hochberg FDR corrected before flagging highly significant features
- The resolution gap (677 vs 245) should be attributable to MetaboAnalyst summing close peaks, i.e. loss of multiplet information

## Limitations

- MetaboAnalyst's lower resolution (245 vs 677 features) removes multiplet information because close peaks are summed together
- The binning/bucketing approach suffers from difficult bin boundary definition in crowded spectra and intrinsic loss of information, which is why speaq 2.0 avoids binning
- Comparison requires the same publicly available dataset (onion intake in mice) processed by both tools; results may not generalize to other NMR datasets

## Evidence

- [abstract] speaq 2.0 workflow starts with peak-picking followed by grouping, avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] Onion intake in mice dataset used for speaq 2.0 workflow: "speaq 2.0 workflow applied to the onion intake in mice data [37]."
- [intro] Speaq 2.0 positioned as complement to MetaboAnalyst: "to construct a method that can complement established tools for NMR data analysis like MetaboAnalyst [12], by improving performance"
- [readme] README minimal speaq workflow: peak detection, grouping, feature matrix construction: "peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)  

# peak grouping
groups <- PeakGrouper(Y.peaks = peaks)"
- [readme] README feature matrix step for downstream statistics: [evidence span withheld — anchor status: not_found]
