---
name: swath-msms-spectrum-quality-evaluation
description: Use when a multisample DIA-MS/SWATH dataset (e.g. SCIEX WIFF negative
  lipidomics files) has been deconvoluted and per-feature MS/MS spectrum quality must
  be scored, or when deciding whether correlation-based deconvolution (CorrDec) improves
  on MS2Dec.
when_to_use_negative:
- 'Single-sample DIA-MS data: CorrDec assumes precursor/fragment ion intensities correlate
  across samples, so it is not applicable without multiple samples (minimum ~10 unique
  samples).'
- DDA-MS/MS data or spectra that are already deconvoluted — no coelution to resolve,
  so MS2Dec/CorrDec comparison is pointless.
- Proprietary vendor formats (e.g. WIFF) when using the 'Debug vendor unsupported'
  source build, which only reads abf (Reifycs), cdf (NetCDF), and mzML.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MS-DIAL
  role: Console application (MSDIALCUI.exe) that executes both the baseline MS2Dec
    and the CorrDec deconvolution arms and emits .mdpeak/.mdalign per-feature outputs
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: MS2Dec
  role: Baseline (original MS-DIAL) deconvolution method used as the comparison arm
    for spectral quality scoring
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: CorrDec
  role: Correlation-based deconvolution method whose per-feature match scores are
    compared against MS2Dec to evaluate MS2 spectrum quality on DIA-MS data
  repo: https://github.com/systemsomicslab/MsdialWorkbench
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c01980
    title: Correlation-Based Deconvolution (CorrDec) To Generate High-Quality MS2
      Spectra from Data-Independent Acquisition in Multisample Studies
  source_content_hash: sha256:407f227bfcf50b4286e8fd3abbde9e16b1879bb6ac4b0c1855604800dfa9f327
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systemsomicslab/MsdialWorkbench
  tools:
  - .NET Framework
  - CorrDec
  - MS-DIAL
  - MS2Dec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# swath-msms-spectrum-quality-evaluation

## Summary

Evaluates the quality of MS2 spectra deconvoluted from DIA-MS (SWATH) data by comparing the correlation-based CorrDec method against the original MS-DIAL MS2Dec method, using per-feature match scores (Simple/Weighted/Reverse dot products) as quality signals.

## When to use

Use when a multisample DIA-MS/SWATH dataset (e.g. SCIEX WIFF negative lipidomics files) has been deconvoluted and per-feature MS/MS spectrum quality must be scored, or when deciding whether correlation-based deconvolution (CorrDec) improves on MS2Dec.

## When NOT to use

- Single-sample DIA-MS data: CorrDec assumes precursor/fragment ion intensities correlate across samples, so it is not applicable without multiple samples (minimum ~10 unique samples).
- DDA-MS/MS data or spectra that are already deconvoluted — no coelution to resolve, so MS2Dec/CorrDec comparison is pointless.
- Proprietary vendor formats (e.g. WIFF) when using the 'Debug vendor unsupported' source build, which only reads abf (Reifycs), cdf (NetCDF), and mzML.

## Inputs

- DIA-MS/SWATH LC-MS/MS raw data files (e.g. 7 SCIEX WIFF LC-MS/MS negative lipidomics files from the FastLC WIFF demo)
- MS-DIAL console method file (method.txt) with deconvolution options (MS2Dec baseline or CorrDec enabled)
- MS/MS reference spectral library, e.g. lib\MSMS-Public_all-neg-VS19.msp and lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt

## Outputs

- .mdpeak/.mdalign per-feature deconvolution outputs per run
- Side-by-side per-feature comparison table with paired MS2Dec vs CorrDec score columns (Simple/Weighted/Reverse dot products)
- Per-feature outcome classes: reference match, low score, no MS2, unknown

## How to apply

Build the MS-DIAL console (MSDIALCUI.exe) from systemsomicslab/MsdialWorkbench against .NET Framework 4.7.2, then run the baseline MS2Dec deconvolution on the FastLC WIFF demo dataset via the lcms command with a method file. Re-run the identical command with the CorrDec correlation-based deconvolution option enabled in the method file, writing to a separate output directory. Parse the .mdpeak/.mdalign per-feature outputs of both runs, extracting per-feature MS/MS match scores (Simple/Weighted/Reverse dot products) and fragment correlation-derived spectral values, keyed by peak/alignment ID. Build a side-by-side comparison table with one row per feature and paired MS2Dec/CorrDec columns, normalizing mzTab MTD mzTab-ID differences between runs. Note that CorrDec relies on precursor/fragment ion abundance correlations across samples, so it requires multisample input (it can contribute with as few as 10 unique samples), and the chemistry judgment on which coeluting low-abundance compounds are correctly deconvoluted is deferred to expert review.

## Related tools

- **MS-DIAL** (Console application (MSDIALCUI.exe) that executes both the baseline MS2Dec and the CorrDec deconvolution arms and emits .mdpeak/.mdalign per-feature outputs) — https://github.com/systemsomicslab/MsdialWorkbench
- **MS2Dec** (Baseline (original MS-DIAL) deconvolution method used as the comparison arm for spectral quality scoring) — https://github.com/systemsomicslab/MsdialWorkbench
- **CorrDec** (Correlation-based deconvolution method whose per-feature match scores are compared against MS2Dec to evaluate MS2 spectrum quality on DIA-MS data) — https://github.com/systemsomicslab/MsdialWorkbench

## Examples

```
MSDIALCUI.exe lcms -i <demo>\analysis_files.csv -o <out1> -m <demo>\method.txt
```

## Evaluation signals

- The comparison table must have the same number of feature rows in both the MS2Dec and CorrDec deconvolution arms.
- Each feature row must have populated score fields for every outcome class (reference match, low score, no MS2, unknown).
- Runs must be keyed comparably by peak/alignment ID after normalizing any timestamp-derived mzTab MTD mzTab-ID differences.
- For aligned outputs, check reproducibility via SHA256/line-count identity between full and alignment-light mode runs (e.g. 908 LC-MS alignment spots retained in both).
- Chemistry-level agreement (which coeluting low-abundance compounds are correctly deconvoluted) is validated by expert review, not by automated scores alone.

## Limitations

- CorrDec requires measurement of multiple samples; it deconvolutes using cross-sample intensity correlations and cannot work on single-sample acquisitions.
- The 'Debug/Release vendor unsupported' build cannot read proprietary manufacturer formats (WIFF, etc.); only abf, cdf, and mzML are supported unless an official release binary is used.
- CorrDec's advantage is specifically the discrimination of coeluting low-abundance compounds; automated score tables do not by themselves settle chemical correctness.
- Building from source does not download large library files by default; pass -p:DownloadLibraries=true to MSBuild/dotnet build if library files are missing.

## Evidence

- [abstract] CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples and exploits this quantitative information to deconvolute complex DIA spectra.: "CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples and exploits this quantitative information to deconvolute complex DIA spectra."
- [abstract] CorrDec clearly improved deconvolution of the original MS-DIAL deconvolution method (MS2Dec) in a dilution series of chemical standards and a 224-sample urinary metabolomics study.: "CorrDec clearly improved deconvolution of the original MS-DIAL deconvolution method (MS2Dec) in a dilution series of chemical standards and a 224-sample urinary metabolomics study."
- [abstract] The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds.: "The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds."
- [methods] we primarily utilize the frameworks of .NET Framework 4.7.2, .NET Core 3.1, and .NET 6: "we primarily utilize the frameworks of .NET Framework 4.7.2, .NET Core 3.1, and .NET 6"
- [readme] For the 'Debug/Release vendor unsupported' version, only the following formats are supported: abf(Reifycs), cdf(NetCDF), and mzml.: [evidence span withheld — anchor status: external_doc]
