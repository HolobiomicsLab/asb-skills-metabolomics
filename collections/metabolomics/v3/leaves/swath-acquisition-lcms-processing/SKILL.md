---
name: swath-acquisition-lcms-processing
description: Use when processing multisample DIA-MS/MS data acquired with SWATH acquisition
  (e.g. SCIEX WIFF lipidomics/metabolomics files) and coeluted fragment spectra must
  be deconvoluted into compound-specific MS2 spectra without authentic standards.
when_to_use_negative:
- Single-sample DIA studies — CorrDec requires cross-sample abundance correlation,
  so a sample set of fewer than ~10 unique samples is unlikely to benefit
- Data-dependent acquisition (DDA) MS/MS data where spectra are already compound-specific
  and no deconvolution is needed
- When a baseline MS2Dec deconvolution is sufficient and coeluting low-abundance compound
  discrimination is not the goal
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-DIAL
  role: LC-MS/MS processing software whose console (MSDIALCUI.exe lcms) runs peak
    detection, alignment, and deconvolution of SWATH DIA data; CorrDec is implemented
    as an update to its processing chain
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: CorrDec
  role: Correlation-based deconvolution module that deconvolutes complex DIA-MS2 spectra
    by retaining fragment ions whose cross-sample abundance correlates with the precursor
    ion
  repo: https://github.com/systemsomicslab/MsdialWorkbench
- name: MS2Dec
  role: Original MS-DIAL deconvolution method used as the baseline against which CorrDec's
    improvement is compared
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c01980
    title: Correlation-Based Deconvolution (CorrDec) To Generate High-Quality MS2
      Spectra from Data-Independent Acquisition in Multisample Studies
  source_content_hash: sha256:96519687542b7346281fcb8dc48004969e8360c097d7910787722b2400a23802
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systemsomicslab/MsdialWorkbench
  tools:
  - .NET Framework
  - CorrDec
  - MS-DIAL
  - MS2Dec
  - SWATH acquisition
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# swath-acquisition-lcms-processing

## Summary

Process SWATH DIA LC-MS/MS data with MS-DIAL, using the CorrDec correlation-based deconvolution to generate high-quality MS2 spectra from multisample studies where precursor and fragment ion intensities correlate across samples.

## When to use

Use when processing multisample DIA-MS/MS data acquired with SWATH acquisition (e.g. SCIEX WIFF lipidomics/metabolomics files) and coeluted fragment spectra must be deconvoluted into compound-specific MS2 spectra without authentic standards.

## When NOT to use

- Single-sample DIA studies — CorrDec requires cross-sample abundance correlation, so a sample set of fewer than ~10 unique samples is unlikely to benefit
- Data-dependent acquisition (DDA) MS/MS data where spectra are already compound-specific and no deconvolution is needed
- When a baseline MS2Dec deconvolution is sufficient and coeluting low-abundance compound discrimination is not the goal

## Inputs

- Multisample SWATH DIA LC-MS/MS raw files (e.g. SCIEX WIFF, or abf/cdf/mzML for vendor-unsupported builds)
- MS-DIAL console method file with SWATH DIA parameters (Msp file path: lib\MSMS-Public_all-neg-VS19.msp; Text DB file path: lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt)
- analysis_files.csv listing the sample files
- MS-DIAL text database / MSP spectral library

## Outputs

- .mdpeak text table of detected peaks
- .mdalign text table of aligned peaks
- Alignment representative .dcl file containing CorrDec-deconvolved MS2 spectra
- Console run logs over the sample set

## How to apply

Build the MS-DIAL console from the systemsomicslab/MsdialWorkbench repository (Release/net48 per the dotnet_test.yml recipe; pass -p:SkipLibraryDownload=true for the GUI project to skip the external Zenodo download). Prepare a method file with the SWATH DIA LC-MS/MS parameters, setting 'Msp file path' to lib\MSMS-Public_all-neg-VS19.msp and 'Text DB file path' to lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt, and enable CorrDec as the alternative to the MS2Dec deconvolution. Run the console as MSDIALCUI.exe lcms over the set of sample files (e.g. the seven SCIEX WIFF negative-lipidomics demo files). CorrDec computes per-peak ion abundance vectors across the multiple samples and retains fragment ions whose cross-sample abundance correlates with the precursor ion, so it requires multiple samples — a randomized assessment showed it can contribute to studies with as few as 10 unique samples. Export the .mdpeak and .mdalign text tables and the alignment representative .dcl file containing the CorrDec-deconvolved MS2 spectra.

## Related tools

- **MS-DIAL** (LC-MS/MS processing software whose console (MSDIALCUI.exe lcms) runs peak detection, alignment, and deconvolution of SWATH DIA data; CorrDec is implemented as an update to its processing chain) — https://github.com/systemsomicslab/MsdialWorkbench
- **CorrDec** (Correlation-based deconvolution module that deconvolutes complex DIA-MS2 spectra by retaining fragment ions whose cross-sample abundance correlates with the precursor ion) — https://github.com/systemsomicslab/MsdialWorkbench
- **MS2Dec** (Original MS-DIAL deconvolution method used as the baseline against which CorrDec's improvement is compared)

## Examples

```
MSDIALCUI.exe lcms -i <demo>\analysis_files.csv -o <out> -m <demo>\method.txt
```

## Evaluation signals

- Console run completes and generates the .mdpeak, .mdalign and alignment .dcl artifacts
- CorrDec spectra show discrimination of coeluting low-abundance compounds relative to an MS2Dec baseline, per the paper's reported advantage
- Alignment output consistency: .mdalign line count and SHA256 hash match between full and light-mode processing for identical inputs
- Method file contains the expected library paths (lib\MSMS-Public_all-neg-VS19.msp and lib\20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt) before the run

## Limitations

- CorrDec requires measurement of multiple samples; it cannot contribute to single-sample studies (minimum ~10 unique samples per the randomized assessment)
- Building the 'Debug/Release vendor unsupported' configuration cannot read proprietary vendor formats (WIFF, etc.); only abf (Reifycs), cdf (NetCDF), and mzML are supported in that configuration, so proprietary data must be converted or the official release used
- By default, building the project does not download the large library files; the -p:DownloadLibraries=true MSBuild property or -p:SkipLibraryDownload=true must be managed explicitly

## Evidence

- [abstract] CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples and exploits this quantitative information to deconvolute complex DIA spectra.: "CorrDec is based on the assumption that peak intensities of precursor and fragment ions correlate across samples and exploits this quantitative information to deconvolute complex DIA spectra."
- [abstract] The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds.: "The primary advantage of CorrDec over MS2Dec is the ability to discriminate coeluting low-abundance compounds."
- [abstract] our randomized assessment demonstrated that CorrDec can contribute to studies with as few as 10 unique samples: "our randomized assessment demonstrated that CorrDec can contribute to studies with as few as 10 unique samples"
- [intro] the program can deconvolute co-eluted fragment spectra in data independent MS/MS without authentic standard compounds.: [evidence span withheld — anchor status: not_found]
- [readme] Due to licensing restrictions, this version cannot read proprietary data formats from mass spectrometry manufacturers.: [evidence span withheld — anchor status: external_doc]
- [readme] By default, building the project does not download the large library files.: [evidence span withheld — anchor status: external_doc]
