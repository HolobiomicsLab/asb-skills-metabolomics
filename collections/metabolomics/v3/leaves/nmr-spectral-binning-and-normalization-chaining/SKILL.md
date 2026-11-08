---
name: nmr-spectral-binning-and-normalization-chaining
description: Use when you have phase-corrected, baseline-corrected, peak-aligned 1H-NMR
  spectra (e.g. Protomix spectra_df from Bruker FID processing) and need a binned,
  normalized sample-by-bin matrix as input to PCA or similar statistical analysis.
when_to_use_negative:
- The input is already a binned/normalized feature table — re-binning would destroy
  the existing bin structure and double-normalize the data
- The data is still in the FID/time domain (before fourier_transform, phase_correction,
  internal_referencing, baseline_correction and Icoshift peak alignment) — binning
  raw or unaligned spectra produces misleading bins
- The goal is high-resolution spectral deconvolution or targeted metabolite quantification
  rather than statistical fingerprinting — 0.1 ppm rectangular binning discards fine
  peak-shape information
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: Protomix
  role: Provides px.binning (0.1 ppm rectangular bins) and the normalization step
    (TotalArea) applied to the aligned spectra_df
  repo: https://github.com/mzniber/protomix
- name: pyicoshift
  role: Peak alignment (Icoshift) performed immediately before binning so corresponding
    signals fall into the same 0.1 ppm bins
  repo: https://github.com/sekro/pyicoshift
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:49f42b9a530301a1a7f6d2abc83995156a40b05adae741a9d916157fad000d9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  - plotly
  - pyicoshift
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# NMR spectral binning and normalization chaining

## Summary

Chains the final Protomix preprocessing steps — rectangular binning of aligned 1H-NMR spectra at 0.1 ppm resolution followed by TotalArea normalization — to convert full-resolution spectra into a compact, comparable feature matrix suitable for downstream multivariate analysis such as PCA.

## When to use

Use when you have phase-corrected, baseline-corrected, peak-aligned 1H-NMR spectra (e.g. Protomix spectra_df from Bruker FID processing) and need a binned, normalized sample-by-bin matrix as input to PCA or similar statistical analysis.

## When NOT to use

- The input is already a binned/normalized feature table — re-binning would destroy the existing bin structure and double-normalize the data
- The data is still in the FID/time domain (before fourier_transform, phase_correction, internal_referencing, baseline_correction and Icoshift peak alignment) — binning raw or unaligned spectra produces misleading bins
- The goal is high-resolution spectral deconvolution or targeted metabolite quantification rather than statistical fingerprinting — 0.1 ppm rectangular binning discards fine peak-shape information

## Inputs

- Aligned, full-resolution 1H-NMR spectra DataFrame (spectra_df, samples as rows indexed by ppm) from the Protomix phase_correction/internal_referencing/baseline_correction chain
- Peak-aligned spectra (px.Icoshift output) as the direct input to binning
- Bruker-acquired urine NMR dataset (Zenodo record 13467227, NMRDataset)

## Outputs

- Binned spectra DataFrame (bin_df) with 0.1 ppm rectangular bins as columns
- TotalArea-normalized binned spectra matrix (preprocessed_spectra artifact)
- Downstream input suitable for scikit-learn PCA (yielding per-component explained variance ratios)

## How to apply

After the Protomix FID-to-spectrum chain (extract_params, extract_fids, group_delay_removal, solvent_residuals_removal, apodization, zero_filling, fourier_transform, phase_correction, internal_referencing, baseline_correction) and peak alignment via px.Icoshift(), reduce each spectrum to rectangular bins of 0.1 ppm using px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular'). The decision point is bin size: the tutorial uses 0.1 ppm rectangular bins, which is coarse enough to tolerate residual peak misalignment but preserves the diagnostic metabolite regions. Then normalize each binned spectrum (e.g. TotalArea normalization) so spectra become comparable in intensity despite concentration/overall-signal differences between urine samples. Feed the normalized binned matrix into unsupervised PCA; in the prostate cancer vs benign prostatic hyperplasia urine dataset this chain produced the preprocessed_spectra artifact on which PC1-PC10 capture ~92.38% of total variance and PC1-PC4 capture ~68.25%.

## Related tools

- **Protomix** (Provides px.binning (0.1 ppm rectangular bins) and the normalization step (TotalArea) applied to the aligned spectra_df) — https://github.com/mzniber/protomix
- **pyicoshift** (Peak alignment (Icoshift) performed immediately before binning so corresponding signals fall into the same 0.1 ppm bins) — https://github.com/sekro/pyicoshift

## Examples

```
import protomix as px; bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular'); norm_df = px.normalization(bin_df, method='TotalArea')
```

## Evaluation signals

- bin_df column widths correspond to 0.1 ppm intervals over the retained spectral window (rectangular binning with bin_size=0.1)
- After TotalArea normalization, each sample row has equal total intensity across bins
- PCA fitted (unsupervised, without labels) on the normalized binned matrix reproduces the reference variance profile: PC1-PC10 ~92.38% cumulative explained variance and PC1-PC4 ~68.25%
- The component-wise and cumulative variance table is written to pca_variance_report.csv
- Every sample that entered binning appears in the normalized matrix (no rows dropped by the binning/normalization steps)

## Limitations

- Binning at 0.1 ppm trades spectral resolution for robustness; small residual peak shifts can still split a peak across adjacent bins if Icoshift alignment is skipped or imperfect
- The choice of bin size and normalization method (e.g. TotalArea) directly changes the variance structure that downstream PCA reports, so variance percentages are only comparable under the same preprocessing chain
- Protomix's documented parameters for these steps come from the tutorial workflow; the README only lists 'Binning' and 'Normalization' as available features without parameter detail

## Evidence

- [methods] binning parameters and chaining context: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [readme] Protomix feature scope includes binning and normalization: [evidence span withheld — anchor status: not_found]
- [intro] Protomix's purpose as NMR preprocessing package: "Protomix provides the tools you need to prepare your NMR data for analysis"
- [abstract] package scope covering extraction, preprocessing and visualization: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
