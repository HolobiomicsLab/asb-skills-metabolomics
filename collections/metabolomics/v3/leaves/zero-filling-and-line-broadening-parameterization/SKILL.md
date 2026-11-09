---
name: zero-filling-and-line-broadening-parameterization
description: Use when raw time-domain 1H-NMR FIDs (e.g., Daphnia extract spectra)
  must be converted to frequency-domain spectra with defined apodization (0.3 Hz exponential)
  and zero-filling (131,072 points) before Fourier transform and automatic phase correction.
when_to_use_negative:
- Input spectra are already frequency-domain and phase-corrected — re-applying 0.3
  Hz exponential broadening would double-apodize and distort linewidths.
- The task is isotopomer distribution analysis from combined NMR multiplet and GC-MS
  data — that belongs to the metabolabpytools train/predict workflow, not FID apodization.
- Processing 2D-NMR data or multivariate-statistics preparation steps (segmental alignment,
  spectral scaling, variance stabilisation) rather than the FID-to-spectrum conversion
  step itself.
edam_topics: []
tools:
- name: MetaboLabPy
  role: Executes the zero-filling (131,072 points), 0.3 Hz exponential line broadening,
    Fourier transform, and reference-spectrum baseline-based automatic phase correction
    on the 100-spectra batch; fastest at 28 s (reference-based) / 45 s (individual).
  repo: https://github.com/ludwigc/MetaboLabPy
- name: metabolabpy
  role: Python package whose example data provides the Daphnia extract 1H-NMR spectra
    and whose API loads and processes the FID batch.
  repo: https://github.com/ludwigc/metabolabpy
- name: numpy
  role: Python toolbox used by MetaboLabPy for the numerical array operations underlying
    line broadening, zero-filling, and the Fourier transform.
- name: scipy
  role: Python toolbox used by MetaboLabPy for signal-processing routines in the NMR
    processing pipeline.
- name: pandas
  role: Python toolbox used by MetaboLabPy; supports assembling the timing comparison
    table (processing_times_comparison.csv).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:be21ccfcf5268ef4f1d2703c227d18f3241abd6160ce672a91347073df1f12d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# zero-filling-and-line-broadening-parameterization

## Summary

Applies exponential line broadening (apodization) and zero-filling to raw 1H-NMR FIDs before Fourier transform, using the benchmark parameters from the MetaboLabPy study (0.3 Hz line broadening, zero-filling to 131,072 points) so batches of spectra are processed comparably and timing benchmarks are reproducible.

## When to use

Use when raw time-domain 1H-NMR FIDs (e.g., Daphnia extract spectra) must be converted to frequency-domain spectra with defined apodization (0.3 Hz exponential) and zero-filling (131,072 points) before Fourier transform and automatic phase correction.

## When NOT to use

- Input spectra are already frequency-domain and phase-corrected — re-applying 0.3 Hz exponential broadening would double-apodize and distort linewidths.
- The task is isotopomer distribution analysis from combined NMR multiplet and GC-MS data — that belongs to the metabolabpytools train/predict workflow, not FID apodization.
- Processing 2D-NMR data or multivariate-statistics preparation steps (segmental alignment, spectral scaling, variance stabilisation) rather than the FID-to-spectrum conversion step itself.

## Inputs

- Raw 1H-NMR FID data files (time-domain, e.g., metabolabpy example Daphnia extract spectra)
- Apodization parameters: exponential line broadening value (0.3 Hz in the benchmark)
- Zero-filling target size (131,072 points in the benchmark)
- Batch size definition (100 spectra built by copying four source spectra)

## Outputs

- Zero-filled, line-broadened, Fourier-transformed 1H-NMR spectra (131,072 points each) ready for automatic phase correction
- Per-batch processing timings for each software/algorithm setting (e.g., 28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s)
- Comparison table such as processing_times_comparison.csv with one row per software/algorithm setting

## How to apply

Load the raw FIDs — in the reference benchmark, four Daphnia extract 1H-NMR spectra from the metabolabpy example data are copied to build a 100-spectra batch. Apply exponential line broadening of 0.3 Hz to each FID, then zero-fill to 131,072 points (a power-of-two doubling that improves digital resolution after the Fourier transform). Fourier-transform the batch and apply automatic phase correction to every spectrum; in MetaboLabPy the reference-spectrum baseline-based automatic phase correction algorithm was used, which completed all 100 spectra in 28 s versus 45 s for the individual (per-spectrum) workflow. When comparing software, keep line broadening, zero-filling size, and phase-correction settings identical across tools (MetaboLabPy 28 s/45 s, MNova drag-and-drop 1 min 13 s, TopSpin Python script 1 min 24 s, MetaboLab scripting 3 min 53 s) so timing differences reflect implementation rather than parameterization. Assemble the resulting timings into a single comparison table (e.g., processing_times_comparison.csv).

## Related tools

- **MetaboLabPy** (Executes the zero-filling (131,072 points), 0.3 Hz exponential line broadening, Fourier transform, and reference-spectrum baseline-based automatic phase correction on the 100-spectra batch; fastest at 28 s (reference-based) / 45 s (individual).) — https://github.com/ludwigc/MetaboLabPy
- **metabolabpy** (Python package whose example data provides the Daphnia extract 1H-NMR spectra and whose API loads and processes the FID batch.) — https://github.com/ludwigc/metabolabpy
- **numpy** (Python toolbox used by MetaboLabPy for the numerical array operations underlying line broadening, zero-filling, and the Fourier transform.)
- **scipy** (Python toolbox used by MetaboLabPy for signal-processing routines in the NMR processing pipeline.)
- **pandas** (Python toolbox used by MetaboLabPy; supports assembling the timing comparison table (processing_times_comparison.csv).)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- processing_times_comparison.csv exists and contains exactly five rows, one per software/algorithm setting (MetaboLabPy reference-based, MetaboLabPy individual, MNova, TopSpin, MetaboLab).
- Reported values match the published benchmark: 28 s, 45 s, 1 min 13 s, 1 min 24 s, and 3 min 53 s.
- The MetaboLabPy timings (28 s and 45 s) fall within the same order of magnitude as the reported benchmark.
- Each processed spectrum has the expected length of 131,072 points after zero-filling, confirming the zero-fill step ran before the Fourier transform.
- The 0.3 Hz exponential line broadening was applied to the FID prior to the Fourier transform, so apodization is not skipped or applied twice.

## Limitations

- Timings were measured on a single benchmark (100 Daphnia extract 1H-NMR spectra, zero-filled to 131,072 points with 0.3 Hz line broadening) and may not transfer to other hardware, batch sizes, or acquisition parameters.
- The workflow variant matters: MetaboLabPy's reference-spectrum baseline-based automatic phase correction (28 s) is faster than the per-spectrum individual workflow (45 s), and MetaboLab scripting was the slowest at 3 min 53 s.
- No changelog found for the project (per the discussion section), so exact version-to-version behaviour changes in processing routines may be undocumented.

## Evidence

- [abstract] This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc.: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
