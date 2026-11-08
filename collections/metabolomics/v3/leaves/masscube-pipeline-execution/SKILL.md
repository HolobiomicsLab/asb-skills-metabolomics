---
name: masscube-pipeline-execution
description: Use when raw LC-MS/MS data (mzML, Bruker timsTOF .d, Astral MS) must
  be processed into a quality-controlled feature table with adduct/ISF grouping, MS/MS
  annotation, and benchmarked peak-detection accuracy.
when_to_use_negative:
- Input is already a processed feature table — MassCube's pipeline starts from raw
  file imports and management, not downstream tables
- Direct metabolite identification is required — MassCube does not directly perform
  metabolite identification, only enhances annotation accuracy
- A graphical user interface is required — no GUI is available in the current version,
  and planned features like time-series analyses, SERRF normalization, formula predictions,
  and chemical classification are not yet available
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: 'Python package executing the full untargeted LC-MS pipeline: raw import,
    peak detection, adduct/ISF grouping, alignment, annotation, and QC'
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime language chosen for MassCube to leverage array programming and Python's
    numerical library ecosystem
- name: MZmine3
  role: Comparison tool against which MassCube's double-peak accuracy (95.2% vs. lower
    baselines) and speed are benchmarked
- name: MS-DIAL
  role: Comparison tool benchmarked in the synthetic mzML accuracy and Astral speed
    evaluations
- name: XCMS
  role: Comparison tool benchmarked in the synthetic mzML accuracy and Astral speed
    evaluations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:2136a34a1f07494096f26878884b5a2f6d00927cbf5230c46c68054bfef54bbe
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# masscube-pipeline-execution

## Summary

Run MassCube's end-to-end untargeted LC-MS metabolomics pipeline in Python, from raw mzML or Bruker .d imports through peak detection, adduct/ISF grouping, alignment, and annotation. It is used because MassCube reports higher peak-detection accuracy and speed than MZmine3, xcms, and MS-DIAL.

## When to use

Use when raw LC-MS/MS data (mzML, Bruker timsTOF .d, Astral MS) must be processed into a quality-controlled feature table with adduct/ISF grouping, MS/MS annotation, and benchmarked peak-detection accuracy.

## When NOT to use

- Input is already a processed feature table — MassCube's pipeline starts from raw file imports and management, not downstream tables
- Direct metabolite identification is required — MassCube does not directly perform metabolite identification, only enhances annotation accuracy
- A graphical user interface is required — no GUI is available in the current version, and planned features like time-series analyses, SERRF normalization, formula predictions, and chemical classification are not yet available

## Inputs

- Raw LC-MS data files in mzML format
- Bruker timsTOF .d directories (TDF2, DDA-PASEF MsMsType=8) with calibrated m/z and 1/K0 ion mobility
- Insert-peak ground-truth lists for accuracy benchmarking (Zenodo deposit 14159704)
- MS/MS reference databases such as GNPS, MetabolomicsWorkbench, and MetaboLights for annotation

## Outputs

- Quality-controlled aligned feature table (feature detection, gap filling, aligned feature grouping)
- Feature group annotations for isotopes, adducts, and in-source fragments (ISF)
- MS/MS spectrum annotations via identity and fuzzy (analog) search
- Per-peak chromatographic metadata and optionally peak_shape columns when Params.output_peak_shape = True
- Benchmark accuracy CSV (e.g. peak_detection_accuracy.csv with single-peak, double-peak, and overall true-positive rates)

## How to apply

Install MassCube from PyPI with 'pip install masscube'. Load each raw file via read_raw_file_to_obj, then run the untargeted metabolomics workflow (UntargetedMetabolomicsWorkflow / process_single_file), which first constructs mass traces by signal clustering with Gaussian-filter-assisted edge detection using peak-detection parameters sigma = 1.2 and prominence = 0.1. Peaks are then grouped for adduct and in-source fragment detection, compounds are annotated by both identity and fuzzy (analog) MS/MS searches, and final data tables undergo quality control before use in phenotype prediction. The workflow decodes each mzML or supported Bruker .d source once into a versioned .mcraw cache under the project tmp directory, so feature detection, gap filling, and aligned feature grouping reload that cache instead of reparsing the source; set Params.output_peak_shape = True only when full chromatographic traces are explicitly needed. For benchmarking, match detected peaks against an inserted-peak ground-truth list (e.g. Zenodo 14159704 with 13,500 single and 13,500 double peaks) using m/z and retention-time agreement, then compute true-positive accuracy separately for single, co-eluting, and overall classes.

## Related tools

- **MassCube** (Python package executing the full untargeted LC-MS pipeline: raw import, peak detection, adduct/ISF grouping, alignment, annotation, and QC) — https://github.com/huaxuyu/masscube
- **Python** (Runtime language chosen for MassCube to leverage array programming and Python's numerical library ecosystem)
- **MZmine3** (Comparison tool against which MassCube's double-peak accuracy (95.2% vs. lower baselines) and speed are benchmarked)
- **MS-DIAL** (Comparison tool benchmarked in the synthetic mzML accuracy and Astral speed evaluations)
- **XCMS** (Comparison tool benchmarked in the synthetic mzML accuracy and Astral speed evaluations)

## Examples

```
pip install masscube; from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.d')
```

## Evaluation signals

- On the synthetic mzML benchmark (Zenodo 14159704), expect overall average peak-detection accuracy of ~96.5% and double-peak accuracy of ~95.2% true-positive rate
- Peak detection should achieve ~100% signal coverage with comprehensive reporting of chromatographic metadata for quality assurance
- A versioned .mcraw cache must appear under the project tmp directory after the first decode, and later stages must reload it rather than reparse the source
- Throughput check: 105 GB of Astral MS data should process on a laptop within ~64 min; orders-of-magnitude slower runs suggest misconfiguration relative to the 8–24x faster MassCube baseline
- Single-file TXT output keeps a peak_shape column but must not embed full chromatographic traces unless Params.output_peak_shape = True

## Limitations

- Currently supported Bruker data are limited to TDF2 (TimsCompressionType=2), DDA-PASEF (MsMsType=8), m/z calibration model 1 with dC2=0 and C3=C4=0, and static TIMS calibration model 2; the reported 1/K0 applies static mobility calibration but not the vendor's per-frame pressure compensation
- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios (e.g. varying scan numbers, peak tailing hard to explore with synthetic data)
- External validated benchmarking datasets with large-scale ground truth are still missing; downstream biological or clinical experiments remain necessary to confirm biological function

## Evidence

- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata for quality assurance.: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata for quality assurance."
- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] Currently supported Bruker data are TDF2 (TimsCompressionType=2), DDA-PASEF (MsMsType=8), m/z calibration model 1 with dC2=0 and C3=C4=0, and static TIMS calibration model 2.: "Currently supported Bruker data are TDF2 (`TimsCompressionType=2`), DDA-PASEF
(`MsMsType=8`), m/z calibration model 1 with `dC2=0` and `C3=C4=0`, and static
TIMS calibration model 2."
- [discussion] if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated: "if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach."
