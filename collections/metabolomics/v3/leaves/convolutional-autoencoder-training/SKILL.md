---
name: convolutional-autoencoder-training
description: Use when raw LC/MS mzML files and an XCMS-generated peak list (mz, rt
  columns) must be converted into learned EIC embeddings for peak curation, and no
  task-specific labeled training set is available for the encoder stage.
when_to_use_negative:
- You already have a trained PeakDetective smoother — load existing weights with integ.load
  instead of retraining.
- You need peak classification labels, not representations — the smoother is unsupervised;
  classification requires trainClassifierActive with user-labeled peaks.
- You have no raw mzML data and only a processed feature table — the EIC matrix (60
  datapoints per 1-min window) must be reconstructed from raw scans.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: Python package providing the PeakDetective object, rawData loader, makeDataMatrix,
    and trainSmoother used to train the convolutional autoencoder
  repo: https://github.com/pattilab/PeakDetective
- name: TensorFlow
  role: Backend deep-learning framework used to build and train the autoencoding CNN
- name: numpy
  role: Required package used for array handling of the n x 60 EIC data matrix
- name: pyteomics
  role: Required package used for reading mzML raw data into rawData objects
- name: XCMS
  role: R tool that generates the input peak list (mz, rt columns) consumed by the
    autoencoder pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:2eeeeb461d473256c93e445a478c55e5a44c7a7f040eed46f6b2232f6a65867f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - TensorFlow
  - XCMS
  - numpy
  - pyteomics
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# convolutional-autoencoder-training

## Summary

Train an unsupervised autoencoding convolutional neural network (the PeakDetective 'smoother') that compresses each extracted ion chromatogram (EIC) into a low-dimensional latent representation, denoising raw LC/MS peak shapes so a downstream active-learning classifier can separate artifacts from true peaks.

## When to use

Use when raw LC/MS mzML files and an XCMS-generated peak list (mz, rt columns) must be converted into learned EIC embeddings for peak curation, and no task-specific labeled training set is available for the encoder stage.

## When NOT to use

- You already have a trained PeakDetective smoother — load existing weights with integ.load instead of retraining.
- You need peak classification labels, not representations — the smoother is unsupervised; classification requires trainClassifierActive with user-labeled peaks.
- You have no raw mzML data and only a processed feature table — the EIC matrix (60 datapoints per 1-min window) must be reconstructed from raw scans.

## Inputs

- mzML raw LC/MS files (loaded as PeakDetective.rawData objects)
- XCMS peak list CSV with mz and rt columns (e.g. peaks_formatted.csv)
- EIC data matrix built by makeDataMatrix: n peaks x 60 datapoints per 1-min window

## Outputs

- Trained smoother autoencoder weights (saved via integ.save into PeakDetectiveObject directory)
- n x 5 latent representation encoding each EIC
- Smoothed/denoised EIC representations for downstream classifier training

## How to apply

Load each mzML file into PeakDetective.rawData objects with readRawDataFile using a 25.5 ppm MS1 tolerance, then build the EIC data matrix with makeDataMatrix so every 1-minute EIC window is sampled at 60 datapoints (resolution=60), optionally with retention-time alignment (align=True). Initialize PeakDetective(numCores=10, resolution=60) and call trainSmoother with min_peaks=100000, smooth_epochs=10, batchSizeSmoother=64, and validationSplitSmoother=0.1; the autoencoder is trained unsupervised on this large unlabeled peak corpus, so no manual labels are needed at this stage. The trained encoder outputs an n x 5 latent matrix encoding each n x 60 EIC, which serves as the fixed feature representation fed to the subsequent active-learning classifier; persist the weights with integ.save before classification so the same smoother is reused at inference.

## Related tools

- **PeakDetective** (Python package providing the PeakDetective object, rawData loader, makeDataMatrix, and trainSmoother used to train the convolutional autoencoder) — https://github.com/pattilab/PeakDetective
- **TensorFlow** (Backend deep-learning framework used to build and train the autoencoding CNN)
- **numpy** (Required package used for array handling of the n x 60 EIC data matrix)
- **pyteomics** (Required package used for reading mzML raw data into rawData objects)
- **XCMS** (R tool that generates the input peak list (mz, rt columns) consumed by the autoencoder pipeline)

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True); integ.trainSmoother(peaklist, raw_data, 100000, 10, 64, 0.1); integ.save('PeakDetectiveObject/')
```

## Evaluation signals

- Data matrix shape check: makeDataMatrix should yield 60 sampled datapoints per 1-minute EIC window (resolution=60) for every peak in the XCMS list.
- Training converged when the smoother trains on at least min_peaks=100000 peaks over 10 epochs with a 0.1 validation split without errors.
- Latent dimensionality check: the encoded representation is n x 5 per EIC.
- Round-trip persistence: weights saved with integ.save into PeakDetectiveObject/ can be reloaded with integ.load and produce consistent EIC encodings.
- Downstream sanity: the subsequent active-learning classifier reaches a bimodal score histogram and dropping val_mean_absolute_error after 5-10 iterations, implying the latent features are informative.

## Limitations

- Model parameters (resolution=60, window=1.0 min, min_peaks=100000, batchSizeSmoother=64) are flagged in the README as 'not recomended to change', so the smoother expects EICs of this fixed resolution.
- Training the full stack requires installing R packages for XCMS/mz.unity (can take >20 min) if peak lists must be generated from scratch.
- The autoencoder must be retrained or fine-tuned per LC/MS method and sample type; PeakDetective is designed to be 'rapidly tailored to specific LC/MS methods and sample types to maximize performance'.

## Evidence

- [full_text] source card finding on matrix shape: [evidence span withheld — anchor status: not_found]
