---
name: feature-level-deconvolution-comparison
description: Use when the same DIA-MS/SWATH dataset must be processed through two
  deconvolution arms (MS2Dec vs CorrDec) and compared feature-by-feature via per-feature
  MS/MS match scores.
when_to_use_negative:
- 'Single-sample DIA-MS data: CorrDec exploits precursor-fragment abundance correlations
  across samples, so it needs multisample studies (it was shown to contribute from
  as few as 10 unique samples).'
- The goal is already a merged feature/alignment table and no MS2 spectral deconvolution
  comparison is required (e.g. downstream PeakCountFilter/QcFilter/BlankFilter tuning
  only).
- Data-dependent acquisition (DDA) data, where coeluting fragment spectra do not need
  DIA-style deconvolution.
edam_topics: []
tools:
- name: MS-DIAL
  role: Console application (MSDIALCUI.exe, 'lcms' mode) run twice from the same method
    file with only the deconvolution option changed between arms
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: MS2Dec
  role: Baseline MS-DIAL deconvolution method used as the comparison arm
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: CorrDec
  role: Correlation-based deconvolution method enabled in the method file for the
    treatment arm
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: .NET Framework 4.7.2
  role: Framework targeted when building MSDIALCUI.exe from the MsdialWorkbench source
  repo: https://github.com/systemsomicslab/MsdialWorkbench
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c01980
    title: Correlation-Based Deconvolution (CorrDec) To Generate High-Quality MS2
      Spectra from Data-Independent Acquisition in Multisample Studies
  source_content_hash: sha256:ac8cf5c669a25e1e6092f251e562d0f4299585d98d53d3c2740e9cfa1545cf31
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# feature-level-deconvolution-comparison

## Summary

Reproduces a controlled comparison between MS-DIAL's original MS2Dec deconvolution and the correlation-based CorrDec method on the same DIA-MS dataset, producing a per-feature table of paired MS2 spectral deconvolution outcomes.

## When to use

Use when the same DIA-MS/SWATH dataset must be processed through two deconvolution arms (MS2Dec vs CorrDec) and compared feature-by-feature via per-feature MS/MS match scores.

## When NOT to use

- Single-sample DIA-MS data: CorrDec exploits precursor-fragment abundance correlations across samples, so it needs multisample studies (it was shown to contribute from as few as 10 unique samples).
- The goal is already a merged feature/alignment table and no MS2 spectral deconvolution comparison is required (e.g. downstream PeakCountFilter/QcFilter/BlankFilter tuning only).
- Data-dependent acquisition (DDA) data, where coeluting fragment spectra do not need DIA-style deconvolution.

## Inputs

- 7 SCIEX WIFF LC-MS/MS negative lipidomics files (FastLC WIFF demo dataset)
- MS-DIAL console method file (.txt) with deconvolution method toggled between MS2Dec and CorrDec
- MS/MS spectral library: MSMS-Public_all-neg-VS19.msp
- Text DB: 20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt

## Outputs

- Per-feature .mdpeak/.mdalign deconvolution outputs for each deconvolution arm
- Per-feature MS/MS match scores (Simple/Weighted/Reverse dot products) keyed by peak/alignment ID
- Side-by-side comparison table with one row per feature and paired MS2Dec/CorrDec outcome columns (reference match, low score, no MS2, unknown)

## How to apply

Build the MS-DIAL console (MSDIALCUI.exe) from the systemsomicslab/MsdialWorkbench repository targeting .NET Framework 4.7.2, using the release build if proprietary vendor formats such as SCIEX WIFF must be read. Run the baseline MS2Dec deconvolution on the FastLC WIFF demo dataset (7 SCIEX WIFF LC-MS/MS negative lipidomics files) with 'MSDIALCUI.exe lcms -i <demo>\analysis_files.csv -o <out1> -m <demo>\method.txt', pointing 'Msp file path' at lib\MSMS-Public_all-neg-VS19.msp and the text DB at lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt. Re-run the identical command with the CorrDec option enabled in the method file, writing to a separate output directory, so both arms share peak picking and only differ in deconvolution. Parse the .mdpeak/.mdalign outputs of both runs, extracting per-feature MS/MS match scores (Simple/Weighted/Reverse dot products) and fragment correlation-derived spectral values keyed by peak/alignment ID. Build a side-by-side table with one row per feature and paired MS2Dec/CorrDec columns; normalize timestamp-derived mzTab MTD mzTab-ID differences so runs are comparable. Rationale: CorrDec assumes precursor and fragment ion peak intensities correlate across samples, which lets it discriminate coeluting low-abundance compounds that MS2Dec cannot.

## Related tools

- **MS-DIAL** (Console application (MSDIALCUI.exe, 'lcms' mode) run twice from the same method file with only the deconvolution option changed between arms) — https://github.com/systemsomicslab/MsdialWorkbench
- **MS2Dec** (Baseline MS-DIAL deconvolution method used as the comparison arm) — https://github.com/systemsomicslab/MsdialWorkbench
- **CorrDec** (Correlation-based deconvolution method enabled in the method file for the treatment arm) — https://github.com/systemsomicslab/MsdialWorkbench
- **.NET Framework 4.7.2** (Framework targeted when building MSDIALCUI.exe from the MsdialWorkbench source) — https://github.com/systemsomicslab/MsdialWorkbench

## Examples

```
MSDIALCUI.exe lcms -i <demo>\analysis_files.csv -o <out1> -m <demo>\method.txt
```

## Evaluation signals

- The comparison table has the same number of feature rows in both the MS2Dec and CorrDec arms.
- Score fields (Simple/Weighted/Reverse dot products) are populated for every outcome class in each arm (reference match, low score, no MS2, unknown).
- Timestamp-derived mzTab MTD mzTab-ID differences are normalized so the two runs are row-comparable.
- If alignment light mode is used, the .mdalign output must have the same line count and identical SHA256 hash as the full-alignment output (e.g. 908 alignment spots retained).
- Chemistry-level judgment of which compounds are correctly deconvoluted is deferred to expert review, not scored automatically.

## Limitations

- CorrDec requires measurement of multiple samples; a randomized assessment showed it contributes to studies with as few as 10 unique samples.
- The 'Debug/Release vendor unsupported' source build cannot read proprietary vendor formats (e.g. SCIEX WIFF); it supports only abf (Reifycs), cdf (NetCDF), and mzML, so the official release build or format conversion is needed for WIFF input.
- Which compounds are correctly deconvoluted is a chemistry judgment deferred to expert review, not automated by the comparison table itself.

## Evidence

- [abstract] CorrDec exploits cross-sample precursor-fragment intensity correlations: "CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples"
- [abstract] CorrDec's key advantage is discriminating coeluting low-abundance compounds: "The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds."
- [methods] MS-DIAL build targets .NET Framework 4.7.2: "we primarily utilize the frameworks of .NET Framework 4.7.2, .NET Core 3.1, and .NET 6"
- [methods] Library configuration for the deconvolution run: "`Msp file path` set to `lib\MSMS-Public_all-neg-VS19.msp`"
- [readme] Vendor-unsupported builds cannot read proprietary WIFF files: [evidence span withheld — anchor status: external_doc]
