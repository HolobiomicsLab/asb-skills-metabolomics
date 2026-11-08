---
name: multisample-ion-correlation-analysis
description: Use when processing multisample DIA-MS/SWATH acquisitions where coeluting
  low-abundance compounds produce mixed fragment spectra, and quantitative ion correlations
  across samples can be exploited to separate them.
when_to_use_negative:
- 'Single-sample DIA-MS datasets: CorrDec requires multiple samples because it relies
  on cross-sample precursor/fragment ion intensity correlations (fewer than ~10 unique
  samples showed reduced contribution)'
- Data already containing purified/MS1-isolated MS2 spectra or authentic-standard
  spectra that need no deconvolution
- Proprietary vendor formats with the 'Debug/Release vendor unsupported' source build,
  which reads only abf/cdf/mzML and cannot read e.g. SCIEX WIFF files directly
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MS-DIAL
  role: Console/host application that runs both the baseline and correlation-based
    deconvolution pipelines on the DIA-MS files
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: CorrDec
  role: Correlation-based deconvolution method enabled via the method file; deconvolutes
    complex DIA spectra using cross-sample ion intensity correlations
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: MS2Dec
  role: Original MS-DIAL deconvolution method run as the baseline arm for the per-feature
    comparison
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: .NET Framework 4.7.2
  role: Framework targeted when building the MSDIALCUI.exe console for the comparison
    runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c01980
    title: Correlation-Based Deconvolution (CorrDec) To Generate High-Quality MS2
      Spectra from Data-Independent Acquisition in Multisample Studies
  source_content_hash: sha256:0d6e8c15936da3dbbe2feeabb391f1f1e30a4aa50f9e7ae022634bb7fdb1ea96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systemsomicslab/MsdialWorkbench
  tools:
  - .NET Framework
  - .NET Framework 4.7.2
  - CorrDec
  - MS-DIAL
  - MS2Dec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# multisample-ion-correlation-analysis

## Summary

Exploits cross-sample ion abundance correlations in multisample DIA-MS studies (CorrDec) to deconvolute complex, co-eluting DIA-MS2 spectra, as implemented in MS-DIAL as an alternative to the original MS2Dec deconvolution method.

## When to use

Use when processing multisample DIA-MS/SWATH acquisitions where coeluting low-abundance compounds produce mixed fragment spectra, and quantitative ion correlations across samples can be exploited to separate them.

## When NOT to use

- Single-sample DIA-MS datasets: CorrDec requires multiple samples because it relies on cross-sample precursor/fragment ion intensity correlations (fewer than ~10 unique samples showed reduced contribution)
- Data already containing purified/MS1-isolated MS2 spectra or authentic-standard spectra that need no deconvolution
- Proprietary vendor formats with the 'Debug/Release vendor unsupported' source build, which reads only abf/cdf/mzML and cannot read e.g. SCIEX WIFF files directly

## Inputs

- Multisample DIA-MS/SWATH LC-MS/MS raw files (e.g. SCIEX WIFF files from negative-mode lipidomics)
- MS-DIAL console method file (.txt) with deconvolution settings (MS2Dec arm vs CorrDec-enabled arm)
- MS/MS reference library (e.g. MSMS-Public_all-neg-VS19.msp) and text DB (e.g. 20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt)
- MS-DIAL analysis parameter file (analysis_files.csv)

## Outputs

- Per-feature deconvolution outputs (.mdpeak/.mdalign files) for each deconvolution arm
- Side-by-side comparison table with one row per feature and paired MS2Dec/CorrDec columns of Simple/Weighted/Reverse dot-product scores
- Outcome classification per feature (reference match, low score, no MS2, unknown)
- Normalized mzTab export with comparable mzTab-ID values across runs

## How to apply

Build the MS-DIAL console (MSDIALCUI.exe) from the systemsomicslab/MsdialWorkbench repository targeting .NET Framework 4.7.2, then run baseline MS2Dec deconvolution on the DIA-MS files (e.g. the FastLC WIFF demo of 7 SCIEX WIFF negative-lipidomics files) via the lcms subcommand with a method file. Re-run the identical command with the CorrDec correlation-based deconvolution option enabled in the method file, writing to a separate output directory. CorrDec assumes peak intensities of precursor and fragment ions correlate across samples and uses this quantitative information to deconvolute complex DIA spectra, so the input cohort should contain multiple samples (a randomized assessment showed contributions from as few as 10 unique samples). Parse the .mdpeak/.mdalign per-feature outputs of both runs, extracting per-feature MS/MS match scores (Simple/Weighted/Reverse dot products) and fragment correlation-derived spectral values keyed by peak/alignment ID, and normalize any timestamp-derived mzTab MTD mzTab-ID differences so the two arms are comparable. The chemistry judgment of which compounds are correctly deconvoluted is deferred to expert review; the agent's deliverable is the side-by-side per-feature score table.

## Related tools

- **MS-DIAL** (Console/host application that runs both the baseline and correlation-based deconvolution pipelines on the DIA-MS files) — https://github.com/systemsomicslab/MsdialWorkbench
- **CorrDec** (Correlation-based deconvolution method enabled via the method file; deconvolutes complex DIA spectra using cross-sample ion intensity correlations) — https://github.com/systemsomicslab/MsdialWorkbench
- **MS2Dec** (Original MS-DIAL deconvolution method run as the baseline arm for the per-feature comparison) — https://github.com/systemsomicslab/MsdialWorkbench
- **.NET Framework 4.7.2** (Framework targeted when building the MSDIALCUI.exe console for the comparison runs)

## Examples

```
MSDIALCUI.exe lcms -i <demo>\analysis_files.csv -o <out1> -m <demo>\method.txt
```

## Evaluation signals

- The comparison table has the same number of feature rows in both deconvolution arms (MS2Dec and CorrDec)
- Score fields (Simple/Weighted/Reverse dot products) are populated for each outcome class: reference match, low score, no MS2, unknown
- mzTab MTD mzTab-ID timestamp differences are normalized so rows are joinable across the two runs by peak/alignment ID
- Alignment spot counts are consistent between arms (e.g. full alignment retaining 908 LC-MS alignment spots matches alignment-light streaming)
- Expert review of which coeluting low-abundance compounds were correctly deconvoluted is deferred, not auto-scored, by the agent

## Limitations

- CorrDec requires measurement of multiple samples; randomized assessment showed utility only from as few as 10 unique samples upward
- The vendor-unsupported source configuration cannot read proprietary manufacturer formats (only abf, cdf, mzML), so WIFF inputs require an official release build
- Correlation-based deconvolution cannot be validated automatically for chemical correctness; which compounds are correctly deconvoluted requires expert review

## Evidence

- [abstract] CorrDec assumption quote: "CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples and exploits this quantitative information to deconvolute complex DIA spectra."
- [abstract] CorrDec advantage over MS2Dec: "The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds."
- [abstract] Minimum sample count: "our randomized assessment demonstrated that CorrDec can contribute to studies with as few as 10 unique samples"
- [intro] SWATH deconvolution goal: [evidence span withheld — anchor status: not_found]
- [methods] Alignment spot invariant: "full alignment retained 908 LC-MS alignment spots. alignment-light streaming retained 908 LC-MS alignment spots"
- [readme] Vendor format restriction: [evidence span withheld — anchor status: external_doc]
