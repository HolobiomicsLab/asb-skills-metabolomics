---
name: masscube-untargeted-workflow-execution
description: Use when raw nontargeted LC-MS data (mzML or Bruker timsTOF .d) must
  be converted into a feature table with peak detection, adduct/ISF grouping, and
  annotation, especially when accuracy benchmarked against MS-DIAL/MZmine3/xcms matters.
when_to_use_negative:
- Input is already a processed feature table rather than raw vendor or mzML data —
  MassCube's pipeline starts from raw imports and raw data management
- Direct metabolite identification is required — MassCube does not directly perform
  metabolite identification and must be paired with other databases/software for interpretation
- A graphical user interface is required — no GUI is available in the current version
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: 'Primary tool: executes peak detection (mass-trace construction with signal
    clustering and Gaussian-filter-assisted edge detection), peak grouping for adduct/ISF
    detection, alignment, and annotation on raw LC-MS files'
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime environment in which MassCube's untargeted_metabolomics_workflow (process_single_file)
    is invoked via pip-installed package
- name: MS-DIAL
  role: 'Comparison baseline: run in metabolomics (LC-MS) mode v4.9 on the same mzML
    to benchmark accuracy (85.4% vs MassCube 96.5%)'
- name: MZmine3
  role: 'Comparison baseline: run v3.90 with mass detection, chromatogram building,
    deconvolution, and alignment defaults to benchmark accuracy (88.4% vs MassCube
    96.5%)'
- name: XCMS
  role: 'Comparison baseline: run v4.0.0 in R with centWave peak detection and group
    alignment defaults to benchmark accuracy (87.4% vs MassCube 96.5%)'
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:15dba3f3ecd6f9a4c472b49a4ab532866ecab872ece4ebb3f65893acc0465c80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MS-DIAL
  - MZmine3
  - MassCube
  - Python
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# masscube-untargeted-workflow-execution

## Summary

Run MassCube's untargeted metabolomics workflow on LC-MS raw files (mzML, Bruker .d) from raw import through peak detection, adduct/ISF grouping, alignment, and gap filling to produce annotated feature tables. MassCube reports higher peak-detection accuracy (96.5% on synthetic single/double-peak benchmarks) than MS-DIAL, MZmine3, and xcms.

## When to use

Use when raw nontargeted LC-MS data (mzML or Bruker timsTOF .d) must be converted into a feature table with peak detection, adduct/ISF grouping, and annotation, especially when accuracy benchmarked against MS-DIAL/MZmine3/xcms matters.

## When NOT to use

- Input is already a processed feature table rather than raw vendor or mzML data — MassCube's pipeline starts from raw imports and raw data management
- Direct metabolite identification is required — MassCube does not directly perform metabolite identification and must be paired with other databases/software for interpretation
- A graphical user interface is required — no GUI is available in the current version

## Inputs

- Raw nontargeted LC-MS mzML files (Orbitrap/QTOF/Astral)
- Bruker timsTOF DDA-PASEF TDF2 .d directories (MsMsType=8, TimsCompressionType=2)
- Experimental LC-MS raw files paired with labeled EIC ground-truth metadata (Zenodo 14159704, 722 labeled EICs)

## Outputs

- Feature table with peak detection results, chromatographic metadata, and peak_shape quality column
- Feature groups annotated as isotopes, adducts, and in-source fragments
- MS/MS annotations via identity search and fuzzy (analog) search
- Aligned feature table after chromatographic alignment and gap filling
- Versioned .mcraw workflow cache under the project tmp directory

## How to apply

Install the package with 'pip install masscube' and run the untargeted_metabolomics_workflow on raw mzML files (or Bruker DDA-PASEF timsTOF .d directories). Peaks are detected by constructing mass traces through signal clustering and Gaussian-filter-assisted edge detection, then peaks are grouped for adduct and in-source fragment (ISF) detection, compounds annotated by identity and fuzzy searches, and final tables quality-controlled. The workflow decodes each source once into a versioned .mcraw cache under the project tmp directory; feature detection, gap filling, and aligned feature grouping reload this cache instead of reparsing the source. For single-file TXT output, set Params.output_peak_shape = True if full chromatographic traces are needed. Validate the run by matching detected features against ground truth (e.g., the 13,500 single-peak and 13,500 double-peak synthetic mzML benchmark) and comparing accuracy and runtime against MS-DIAL 4.9, MZmine 3.90, and xcms 4.0.0 feature tables.

## Related tools

- **MassCube** (Primary tool: executes peak detection (mass-trace construction with signal clustering and Gaussian-filter-assisted edge detection), peak grouping for adduct/ISF detection, alignment, and annotation on raw LC-MS files) — https://github.com/huaxuyu/masscube
- **Python** (Runtime environment in which MassCube's untargeted_metabolomics_workflow (process_single_file) is invoked via pip-installed package)
- **MS-DIAL** (Comparison baseline: run in metabolomics (LC-MS) mode v4.9 on the same mzML to benchmark accuracy (85.4% vs MassCube 96.5%))
- **MZmine3** (Comparison baseline: run v3.90 with mass detection, chromatogram building, deconvolution, and alignment defaults to benchmark accuracy (88.4% vs MassCube 96.5%))
- **XCMS** (Comparison baseline: run v4.0.0 in R with centWave peak detection and group alignment defaults to benchmark accuracy (87.4% vs MassCube 96.5%))

## Examples

```
pip install masscube; python -c "from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.d'); print(data.scans[data.ms1_idx_arr[0]].signals)"
```

## Evaluation signals

- On the synthetic mzML benchmark (13,500 true single peaks + 13,500 true double peaks), MassCube accuracy should be ~96.5%, exceeding MZmine3 (88.4%, p=0.0029), xcms (87.4%, p=0.016), and MS-DIAL (85.4%, p=0.0065)
- On 722 manually labeled experimental EICs, MassCube should reach ~93.5% double-peak and ~97.3% single-peak accuracy; write per-image agreement flags to peak_detection_accuracy.csv
- Feature detection should achieve 100% signal coverage with comprehensive reporting of chromatographic metadata for quality assurance
- The .mcraw cache should be created under the project tmp directory on first pass, and downstream feature detection, gap filling, and aligned feature grouping should reload it rather than reparsing the source
- Runtime check: 105 GB of Astral MS data should process on a laptop in ~64 min (other programs took 8–24 times longer)

## Limitations

- No graphical user interface is available in the current version; non-programming users must work through Python
- MassCube does not directly perform metabolite identification; downstream biological or clinical experiments remain necessary to confirm biological function
- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios (e.g., varying scan numbers, peak tailing in convoluted experimental data)
- Planned features not yet available: time-series analyses, SERRF data normalization, formula predictions, chemical classifications, and a GUI

## Evidence

- [abstract] Peak detection via mass traces with edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] Peak grouping for adduct/ISF and annotation: "Peaks are then grouped for adduct and in-source fragment detection"
- [abstract] Benchmark accuracy result: "MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy."
- [methods] mcraw cache mechanism: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] Bruker .d reader support: "MassCube can read DDA-PASEF TDF2 directories directly, including calibrated m/z and inverse reduced ion mobility (`1/K0`)"
- [readme] Installation and quickstart: "To install **masscube**, open a terminal and run:

```bash
pip install masscube
```"
- [readme] peak_shape column behavior in TXT output: "Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default. Set `Params.output_peak_shape = True`"
