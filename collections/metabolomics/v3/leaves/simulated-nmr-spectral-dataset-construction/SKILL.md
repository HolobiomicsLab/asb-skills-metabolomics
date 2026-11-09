---
name: simulated-nmr-spectral-dataset-construction
description: Use when you need a ground-truth-controlled benchmark to compare NMR
  spectra processing methods (binning, alignment like CluPA/icoshift, or peak-based
  approaches) where real spectra lack known per-peak labels.
when_to_use_negative:
- The real experimental dataset already has validated annotations and a well-established
  analysis workflow — simulation is only needed when ground-truth feature labels are
  required
- The goal is quantitation or absolute concentration estimation rather than method
  benchmarking on classification performance
- The spectra are LC-MS data — this skill targets 1D NMR spectra summarization comparisons
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: speaq 2.0
  role: 'Peak-based processing arm of the benchmark: Mexican hat wavelet peak picking
    then grouping, avoiding binning, on the simulated spectra'
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R distribution of speaq 2.0 used to run the peak-based workflow (getWaveletPeaks,
    PeakGrouper, BuildFeatureMatrix) on the simulated data
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Language in which the simulation, processing workflows, and ROC/PRC evaluation
    are executed
- name: CluPA
  role: Competing alignment method applied to the simulated spectra before binning
  repo: https://github.com/beirnaert/speaq
- name: icoshift
  role: Competing alignment method applied to the simulated spectra before binning
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:b01204a06ed5adf1a2536e3921c186f005c42ac22db5569f3815916af033b560
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# simulated-nmr-spectral-dataset-construction

## Summary

Construct a controlled simulated 1H NMR case-vs-control benchmark by combining reference metabolite spectra from HMDB (e.g. HMDB0000440 and HMDB0001336) with added noise, baseline distortion and random peak shifts, so that NMR processing workflows (peak-based vs alignment-plus-binning) can be compared via ROC and Precision-Recall curves on ground truth.

## When to use

Use when you need a ground-truth-controlled benchmark to compare NMR spectra processing methods (binning, alignment like CluPA/icoshift, or peak-based approaches) where real spectra lack known per-peak labels.

## When NOT to use

- The real experimental dataset already has validated annotations and a well-established analysis workflow — simulation is only needed when ground-truth feature labels are required
- The goal is quantitation or absolute concentration estimation rather than method benchmarking on classification performance
- The spectra are LC-MS data — this skill targets 1D NMR spectra summarization comparisons

## Inputs

- Reference 1H NMR metabolite spectra from HMDB (HMDB0000440 and HMDB0001336 in the speaq 2.0 benchmark)
- Simulation prescriptions from S2 Appendix: noise level, baseline distortion, and random peak shift parameters
- Case/control group labels for the simulated spectra (10 case, 10 control)

## Outputs

- Simulated 1H NMR spectra matrix (20 spectra, 10 case and 10 control) with known ground-truth peaks
- ROC curves and Precision-Recall curves per processing method, plotted together (ROC_PRC_curves.png)
- Feature/sample/peak-value matrices from each compared workflow (speaq 2.0 peak-based, raw binning, CluPA+binning, icoshift+binning)

## How to apply

Retrieve the reference 1H NMR spectra for the chosen metabolites (in the speaq 2.0 benchmark, HMDB0000440 and HMDB0001336) from the HMDB database. Simulate the dataset (20 spectra, 10 case and 10 control) by combining the two metabolite spectra with added noise, baseline distortion, and random peak shifts as prescribed in the paper's S2 Appendix, so the shift and distortion magnitudes match the published benchmark. Because the true underlying peaks are known from the HMDB references, the case-vs-control discriminating features are known exactly, enabling ROC and Precision-Recall evaluation of any feature matrix produced by a processing method. Process the identical simulated spectra with each competing summarization workflow — speaq 2.0 peak finding then grouping (Mexican hat wavelet based, avoiding binning), raw binning without alignment, and CluPA or icoshift alignment followed by binning — and compute ROC and Precision-Recall curves per method. The simulation design rationale is that alignment algorithms are 'not always optimal when dealing with severely shifted spectra', so controlled shift magnitudes expose this failure mode quantitatively.

## Related tools

- **speaq 2.0** (Peak-based processing arm of the benchmark: Mexican hat wavelet peak picking then grouping, avoiding binning, on the simulated spectra) — https://github.com/beirnaert/speaq
- **speaq R package** (R distribution of speaq 2.0 used to run the peak-based workflow (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) on the simulated data) — https://cran.r-project.org/package=speaq
- **R** (Language in which the simulation, processing workflows, and ROC/PRC evaluation are executed)
- **CluPA** (Competing alignment method applied to the simulated spectra before binning) — https://github.com/beirnaert/speaq
- **icoshift** (Competing alignment method applied to the simulated spectra before binning)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = simulated.spectra, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- The speaq 2.0 peak-based method (peak finding followed by grouping) must show ROC/PRC performance similar to or better than raw binning and CluPA/icoshift alignment-plus-binning alternatives
- The simulated dataset must contain the prescribed 20 spectra split 10 case / 10 control, built from exactly the two HMDB reference metabolites
- Each method's feature matrix must be derived from the identical simulated spectra so curves are directly comparable
- ROC and Precision-Recall curves for all methods must be computed from the case-vs-control classification of each resulting feature matrix and overlaid in a single plot (ROC_PRC_curves.png)
- Expected qualitative result: alignment-based approaches degrade on severely shifted spectra, while peak-based processing avoids binning-induced information loss

## Limitations

- Spectral alignment algorithm results 'is not always optimal when dealing with severely shifted spectra' — conclusions may not transfer to mildly shifted real spectra
- Simulation relies on the parameter prescriptions of S2 Appendix; different noise/shift magnitudes could change the ranking between methods
- The supplementary notes a case where the problem 'is usually mitigated by the peak filling step but in this case it is not' — peak deletion in grouping can affect the simulated-data comparison

## Evidence

- [supplementary] Simulated data based comparison of speaq 2.0 vs alignment algorithms.: "Simulated data based comparison of speaq 2.0 vs alignment algorithms."
- [supplementary] The speaq 2.0 method, based on finding peaks and subsequently grouping, performs similar or better compared to the other methods: "The speaq 2.0 method, based on finding peaks and subsequently grouping, performs similar or better compared to the other methods"
- [supplementary] The results of spectral alignment algorithms is not always optimal when dealing with severely shifted spectra.: "The results of spectral alignment algorithms is not always optimal when dealing with severely shifted spectra."
- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] peak detection via wavelets in the README example: [evidence span withheld — anchor status: not_found]
