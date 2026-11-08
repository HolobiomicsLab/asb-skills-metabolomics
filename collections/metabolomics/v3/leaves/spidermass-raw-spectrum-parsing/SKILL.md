---
name: spidermass-raw-spectrum-parsing
description: Use when raw SpiderMass mass spectra (m/z, intensity pairs, e.g. the
  Canine Sarcoma dataset) must be TIC-filtered and 0.1-Da-binned into a fixed-dimension
  feature matrix before model training.
when_to_use_negative:
- The input is already a binned, TIC-normalized feature matrix (the README states
  'the input of each dataset is tic normalized in this implementation'), so re-filtering/binning
  would be redundant
- The analysis requires high mass resolution or near-isobaric peak separation, since
  0.1-Da binning collapses intensities within each bin
- Spectra have low TIC by nature of the acquisition protocol, and the 1e4 cutoff would
  discard most of the data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: numpy
  role: Compute per-spectrum TIC, apply the >= 1e4 cutoff, and bin m/z axes at 0.1-Da
    intervals to build the fixed-dimension feature arrays
- name: pandas
  role: Parse raw spectrum m/z/intensity pairs and assemble/write the per-file binned
    feature matrix to CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:89d53075b490800fa9680940167e5b3819e13b45af2b0eb8e350e19dea2a654e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python (numpy/pandas)
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spidermass-raw-spectrum-parsing

## Summary

Reconstructs the SpiderMass raw data processing pipeline: filters mass spectra by total ion count (TIC >= 1e4) and bins the retained spectra along the m/z axis with a 0.1 Da bin width, producing a uniform 2D feature matrix per MS file that downstream deep learning classifiers (e.g. MSMCE-ResNet50) consume.

## When to use

Use when raw SpiderMass mass spectra (m/z, intensity pairs, e.g. the Canine Sarcoma dataset) must be TIC-filtered and 0.1-Da-binned into a fixed-dimension feature matrix before model training.

## When NOT to use

- The input is already a binned, TIC-normalized feature matrix (the README states 'the input of each dataset is tic normalized in this implementation'), so re-filtering/binning would be redundant
- The analysis requires high mass resolution or near-isobaric peak separation, since 0.1-Da binning collapses intensities within each bin
- Spectra have low TIC by nature of the acquisition protocol, and the 1e4 cutoff would discard most of the data

## Inputs

- Raw SpiderMass mass spectra files (m/z and intensity pairs), e.g. the Canine Sarcoma dataset
- Per-file list of spectra with spectrum identifiers, m/z values, and intensities

## Outputs

- CSV feature matrix per MS file (e.g. canine_sarcoma_feature_matrix.csv) with rows = TIC-retained spectra and columns = 0.1-Da m/z bins
- Set of retained spectra passing the TIC >= 1e4 cutoff

## How to apply

Parse each raw SpiderMass spectrum file into its m/z and intensity pairs, then compute each spectrum's total ion count (TIC) and discard any spectrum with TIC below 1x10^4 so only high-quality spectra are retained. Bin the retained spectra along the m/z axis with a 0.1 Da bin width (the README's bin_size default) so every spectrum is represented by a fixed number of m/z features — e.g. spectrum_dim 15000 for Canine Sarcoma, 12000 for NSCLC/CRLM, and 9900 for RCC in the MSMCE implementation. Assemble the binned spectra as one row per retained spectrum in a 2D feature matrix per MS file (rows = retained spectra, columns = 0.1-Da m/z bins), aggregating intensities that fall in the same bin. Apply TIC normalization to the matrix before model input, matching the README note that 'the input of each dataset is tic normalized in this implementation', and optionally enable the use_normalization flag during training. Write each file's matrix to CSV (e.g. canine_sarcoma_feature_matrix.csv) as the artifact consumed by the downstream training script.

## Related tools

- **numpy** (Compute per-spectrum TIC, apply the >= 1e4 cutoff, and bin m/z axes at 0.1-Da intervals to build the fixed-dimension feature arrays)
- **pandas** (Parse raw spectrum m/z/intensity pairs and assemble/write the per-file binned feature matrix to CSV)

## Examples

```
import numpy as np, pandas as pd; tic = df.groupby('spectrum_id')['intensity'].sum(); keep = tic[tic >= 1e4].index; binned = df[df.spectrum_id.isin(keep)].groupby('spectrum_id').apply(lambda s: np.histogram(s['m/z'], bins=np.arange(0, 1500.1, 0.1), weights=s['intensity'])[0])
```

## Evaluation signals

- Every retained row in the output matrix corresponds to a spectrum with TIC >= 1e4 (no below-threshold spectra survive the filter)
- All rows share a fixed column count equal to the 0.1-Da binned m/z dimension, matching the spectrum_dim expected by the downstream model (e.g. 15000 for Canine Sarcoma, 12000 for NSCLC/CRLM, 9900 for RCC)
- Summing a spectrum's binned intensities recovers its original TIC before normalization
- The CSV output contains one file per MS file, with one row per retained spectrum and the 0.1-Da binned columns as defined in the paper's data processing workflow

## Limitations

- The TIC >= 1x10^4 threshold is dataset-specific; spectra with low TIC but informative signal are discarded
- 0.1-Da binning reduces mass resolution and merges intensities of near-isobaric peaks within each bin
- The MSMCE implementation expects the input data to be TIC normalized; unnormalized matrices must be normalized (use_normalization) before model input
- Bin boundaries and the resulting spectrum_dim depend on the m/z range of each dataset, so dimensions must be tuned per dataset (as the article notes parameter choices require dataset-specific tuning)

## Evidence

- [results] SpiderMass TIC filter and binning: [evidence span withheld — anchor status: not_found]
- [readme] TIC-normalized input to MSMCE models: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
- [readme] 0.1-Da bin_size parameter: "bin_size           | Size of bins for spectrum data (default: `0.1`)."
