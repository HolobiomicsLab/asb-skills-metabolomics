---
name: correlation-based-fragment-annotation
description: Use when analyzing multisample SWATH/DIA-MS2 datasets (e.g., SCIEX WIFF
  lipidomics or urine metabolomics files) where coeluting low-abundance compounds
  corrupt MS2 spectra and MS2Dec deconvolution is inadequate.
when_to_use_negative:
- Single-sample DIA studies — CorrDec is based on cross-sample ion abundance correlation
  and requires measurement of multiple samples
- When the goal is proprietary-format reading from a source-built 'vendor unsupported'
  MS-DIAL configuration, which only supports abf, cdf, and mzML
- When MS2Dec baseline deconvolution is sufficient (no coeluting low-abundance compounds
  to discriminate)
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-DIAL
  role: Host software whose processing chain executes the CorrDec deconvolution of
    DIA-MS2 spectra; built from MsdialWorkbench and run via MSDIALCUI.exe lcms
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: CorrDec
  role: Correlation-based deconvolution module that exploits cross-sample ion abundance
    correlation between precursor and fragment ions to deconvolute complex DIA spectra
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: MS2Dec
  role: Original MS-DIAL deconvolution method used as the baseline against which CorrDec's
    improvement is judged
  repo: https://github.com/systemsomicslab/MsdialWorkbench
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c01980
    title: Correlation-Based Deconvolution (CorrDec) To Generate High-Quality MS2
      Spectra from Data-Independent Acquisition in Multisample Studies
  source_content_hash: sha256:f0935f8cdfc247e4055c9629ef34f2c0fed248a5a3c6adf93f5989a65b2715ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systemsomicslab/MsdialWorkbench
  tools:
  - .NET Framework
  - CorrDec
  - MS-DIAL
  - MS2Dec
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# correlation-based-fragment-annotation

## Summary

Applies CorrDec, a correlation-based deconvolution method in MS-DIAL, to generate high-quality MS2 spectra from DIA-MS2 data by retaining fragment ions whose cross-sample ion abundance correlates with the precursor ion, replacing the older MS2Dec approach.

## When to use

Use when analyzing multisample SWATH/DIA-MS2 datasets (e.g., SCIEX WIFF lipidomics or urine metabolomics files) where coeluting low-abundance compounds corrupt MS2 spectra and MS2Dec deconvolution is inadequate.

## When NOT to use

- Single-sample DIA studies — CorrDec is based on cross-sample ion abundance correlation and requires measurement of multiple samples
- When the goal is proprietary-format reading from a source-built 'vendor unsupported' MS-DIAL configuration, which only supports abf, cdf, and mzML
- When MS2Dec baseline deconvolution is sufficient (no coeluting low-abundance compounds to discriminate)

## Inputs

- Multisample DIA-MS2 (SWATH) LC-MS/MS raw files, e.g., SCIEX WIFF negative-lipidomics demo files
- MS-DIAL console method file with SWATH DIA LC-MS/MS parameters
- Spectral libraries: lib\MSMS-Public_all-neg-VS19.msp and lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt
- analysis_files.csv input list for the MS-DIAL console run

## Outputs

- .mdpeak per-sample text tables
- .mdalign alignment text tables
- Alignment representative .dcl file containing CorrDec-deconvolved MS2 spectra

## How to apply

Build the MS-DIAL console from the MsdialWorkbench repository in Release/net48 following the dotnet_test.yml build recipe, passing -p:SkipLibraryDownload=true for the GUI project to skip the external Zenodo download. In the MS-DIAL processing chain, enable CorrDec instead of MS2Dec: compute per-peak ion abundance vectors across the multiple sample files and deconvolute each SWATH DIA-MS2 spectrum by keeping fragment ions whose cross-sample abundance correlates with the precursor ion. Prepare the method file with SWATH DIA LC-MS/MS parameters, setting 'Msp file path' to lib\MSMS-Public_all-neg-VS19.msp and 'Text DB file path' to lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt. Run MSDIALCUI.exe lcms over the sample set (e.g., the seven SCIEX WIFF negative-lipidomics demo files), then export .mdpeak and .mdalign text tables plus the alignment representative .dcl file containing the CorrDec-deconvolved MS2 spectra. CorrDec requires multiple samples — a randomized assessment showed it can contribute to studies with as few as 10 unique samples.

## Related tools

- **MS-DIAL** (Host software whose processing chain executes the CorrDec deconvolution of DIA-MS2 spectra; built from MsdialWorkbench and run via MSDIALCUI.exe lcms) — https://github.com/systemsomicslab/MsdialWorkbench
- **CorrDec** (Correlation-based deconvolution module that exploits cross-sample ion abundance correlation between precursor and fragment ions to deconvolute complex DIA spectra) — https://github.com/systemsomicslab/MsdialWorkbench
- **MS2Dec** (Original MS-DIAL deconvolution method used as the baseline against which CorrDec's improvement is judged) — https://github.com/systemsomicslab/MsdialWorkbench

## Examples

```
MSDIALCUI.exe lcms -i <demo>\analysis_files.csv -o <out> -m <demo>\method.txt
```

## Evaluation signals

- The MS-DIAL console run completes and generates the .mdpeak, .mdalign, and alignment .dcl artifacts
- CorrDec-deconvolved spectra show discrimination of coeluting low-abundance compounds relative to an MS2Dec baseline
- Improved deconvolution is reproduced in validation contexts like a dilution series of chemical standards (as in the paper's 224-sample urinary metabolomics study)
- Method file correctness: 'Msp file path' resolves to lib\MSMS-Public_all-neg-VS19.msp and 'Text DB file path' to lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt

## Limitations

- CorrDec requires measurement of multiple samples; it cannot contribute to single-sample analyses (minimum ~10 unique samples per randomized assessment)
- The 'Debug/Release vendor unsupported' build configuration cannot read proprietary vendor data formats; only abf (Reifycs), cdf (NetCDF), and mzML are supported, so WIFF files require the official release build or conversion

## Evidence

- [abstract] CorrDec exploits cross-sample correlation: "CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples and exploits this quantitative information to deconvolute complex DIA spectra."
- [abstract] CorrDec advantage over MS2Dec: "The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds."
- [abstract] Minimum sample requirement: "our randomized assessment demonstrated that CorrDec can contribute to studies with as few as 10 unique samples"
- [readme] Vendor-unsupported build format limits: [evidence span withheld — anchor status: external_doc]
- [intro] DIA-MS2 SWATH scope: [evidence span withheld — anchor status: not_found]
