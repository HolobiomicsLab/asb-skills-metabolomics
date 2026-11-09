---
name: peak-list-spectrum-filtering
description: Use when raw or converted MS2 peak lists (e.g., .mzXML/.mgf files) are
  about to be clustered, library-searched, or networked in GNPS and need the standard
  GNPS spectrum cleaning filters applied.
when_to_use_negative:
- Input is already a feature table or quantification table (classic molecular networking
  is qualitative, MS2-only; quantitation requires follow-up targeted LC-MS)
- Applying in silico annotation tools or isomer differentiation — MS2 spectra typically
  cannot differentiate regio- or stereo-isomers
- Data already passed through feature-based molecular networking (FBMN) preprocessing
  in MZmine2, where feature-based rather than spectrum-level filtering applies
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the molecular networking workflow (METABOLOMICS-SNETS-V2) that applies
    the spectrum filters before MS-Cluster clustering
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw files to open peak-list formats (.mzXML,
    .mzML, .mgf) before filtering
- name: MZmine2
  role: Open-source MS data processing tool that can prepare peak lists for GNPS workflows
  repo: https://github.com/mzmine/mzmine2/releases
- name: OpenMS TOPPView
  role: Visualization/inspection of MS data during file conversion and preparation
  repo: https://github.com/OpenMS/OpenMS/releases
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:ad806fb08a5272cc73924444240553f4fcfdeb2c7ed023bb250b54817129abb6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MSConvert (ProteoWizard)
  - MZmine2
  - OpenMS TOPPView
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# peak-list-spectrum-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the GNPS MS2 spectrum filters (precursor-ion window deletion, top-N peak windowing, and minimum peak intensity) to peak lists before consensus clustering or molecular networking, so that noisy and redundant fragment peaks are removed prior to spectral comparison.

## When to use

Use when raw or converted MS2 peak lists (e.g., .mzXML/.mgf files) are about to be clustered, library-searched, or networked in GNPS and need the standard GNPS spectrum cleaning filters applied.

## When NOT to use

- Input is already a feature table or quantification table (classic molecular networking is qualitative, MS2-only; quantitation requires follow-up targeted LC-MS)
- Applying in silico annotation tools or isomer differentiation — MS2 spectra typically cannot differentiate regio- or stereo-isomers
- Data already passed through feature-based molecular networking (FBMN) preprocessing in MZmine2, where feature-based rather than spectrum-level filtering applies

## Inputs

- MS2 peak-list files in an open format (.mzXML, .mzML, .mgf)
- Instrument-specific raw data files (converted via MSConvert/ProteoWizard or vendor converters)
- Filter parameter settings (precursor ion window, peaks in 50 Da window count, minimum peak intensity, k for std-dev threshold)

## Outputs

- Filtered MS2 spectra ready for MS-Cluster consensus clustering or direct molecular networking/library search
- Downstream consensus-cluster table (cluster index, precursor m/z, member-spectrum count, consensus peak list)

## How to apply

Convert instrument raw files to an open peak-list format (.mzXML, .mzML, or .mgf) and run the three standard GNPS spectrum filters on each MS2 spectrum: (1) Filter Precursor Ion Window — delete all peaks within a ±17 Da range around the precursor ion mass (default on); (2) Filter Peaks in 50 Da Window — keep only the top six most intense peaks within each ±50 Da window; (3) Filter Below Std Dev / Minimum Peak Intensity — delete fragment ions below a raw intensity threshold, either mean + k×s.d. (recommended off, k=0) or a fixed raw intensity cutoff (default 0). These filters reduce noise and standardize spectra before MS-Cluster grouping or consensus-spectrum generation, since downstream cosine scoring and consensus merging depend on comparable peak sets. Filtered spectra are then fed to MS-Cluster, which groups near-identical spectra whose precursor m/z values fall within the parent-ion mass tolerance and merges fragment peaks into consensus spectra.

## Related tools

- **GNPS** (Hosts the molecular networking workflow (METABOLOMICS-SNETS-V2) that applies the spectrum filters before MS-Cluster clustering) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw files to open peak-list formats (.mzXML, .mzML, .mgf) before filtering)
- **MZmine2** (Open-source MS data processing tool that can prepare peak lists for GNPS workflows) — https://github.com/mzmine/mzmine2/releases
- **OpenMS TOPPView** (Visualization/inspection of MS data during file conversion and preparation) — https://github.com/OpenMS/OpenMS/releases

## Evaluation signals

- Filtered spectra contain no peaks within the ±17 Da window around the precursor ion mass
- Each spectrum retains at most the top six peaks per ±50 Da window
- Consensus-cluster output (cluster count, precursor m/z values, member counts) is comparable to the 'view all clusters with IDs' output of the corresponding GNPS molecular networking job on MSV000083437
- Minimum Peak Intensity and Filter Below Std Dev parameters match defaults (0) unless deliberately changed

## Limitations

- The Filter Below Std Dev filter is recommended off (default k=0); over-aggressive intensity thresholds can delete informative fragment ions
- Filtering happens at the MS2-scan level only, so classic molecular networking provides only qualitative information and no calibrated quantitation
- Even after filtering, only ~5–6% of MS2 spectra in untargeted metabolomics are annotated; most remain unannotated 'dark matter'

## Evidence

- [full_text] MS-Cluster collapses MS2 spectra deemed identical into one consensus spectrum (node): [evidence span withheld — anchor status: not_found]
