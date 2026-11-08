---
name: cross-software-feature-table-harmonization
description: Use when one raw LC-MS mzML file must be processed independently by MS-DIAL,
  MZmine3, xcms, and MassCube so their exported feature tables can be matched against
  a ground-truth list to benchmark peak-detection accuracy.
when_to_use_negative:
- Input is already a processed feature table — harmonization requires each tool to
  process the identical raw mzML itself, so pre-extracted features defeat the benchmark.
- The goal is metabolite identification rather than peak-detection benchmarking; MassCube
  does not directly perform metabolite identification and this skill only compares
  detection accuracy.
- Real experimental data without a known truth list — the accuracy counts depend on
  the 27,000-feature synthetic ground truth, and real data have characteristics (varying
  scan numbers across peaks, peak tailing) that are difficult to capture systematically
  with synthetic data.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3521
- http://edamontology.org/topic_0102
tools:
- name: MassCube
  role: Python LC-MS processing framework under test; runs untargeted_metabolomics_workflow
    (process_single_file) on the shared mzML to produce its feature table
  repo: https://github.com/huaxuyu/masscube
- name: MS-DIAL
  role: Benchmark comparator; run in metabolomics (LC-MS) mode at the standardized
    parameter set to export its feature table
- name: MZmine3
  role: Benchmark comparator; mass detection, chromatogram building, deconvolution,
    and alignment at reported defaults before feature-table export
- name: XCMS
  role: Benchmark comparator; centWave-based peak detection and group alignment in
    R at reported defaults before feature-table export
- name: Python
  role: Runtime for installing MassCube (pip install masscube) and for scripting the
    TP/FP/missed-feature matching against ground truth
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:152caecea3f3223eaa243a6eb933067774cfbb51847fb1f8b03e05e4a2872850
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3521
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# cross-software-feature-table-harmonization

## Summary

Harmonize LC-MS feature tables by running MS-DIAL 4.9, MZmine 3.90, xcms 4.0.0, and MassCube independently on the identical raw mzML file, then matching each tool's detected features to a ground-truth list to benchmark peak-detection accuracy (MassCube 96.5% vs 85.4-88.4% for the others).

## When to use

Use when one raw LC-MS mzML file must be processed independently by MS-DIAL, MZmine3, xcms, and MassCube so their exported feature tables can be matched against a ground-truth list to benchmark peak-detection accuracy.

## When NOT to use

- Input is already a processed feature table — harmonization requires each tool to process the identical raw mzML itself, so pre-extracted features defeat the benchmark.
- The goal is metabolite identification rather than peak-detection benchmarking; MassCube does not directly perform metabolite identification and this skill only compares detection accuracy.
- Real experimental data without a known truth list — the accuracy counts depend on the 27,000-feature synthetic ground truth, and real data have characteristics (varying scan numbers across peaks, peak tailing) that are difficult to capture systematically with synthetic data.

## Inputs

- A single raw LC-MS mzML file shared by all four tools (benchmark file contains 13,500 true single-peak and 13,500 true double-peak features)
- Ground-truth feature list (m/z, retention time) for the synthetic file
- Standardized per-tool parameter sets (MS-DIAL 4.9 metabolomics/LC-MS mode, MZmine 3.90 defaults, xcms 4.0.0 centWave defaults, MassCube defaults)

## Outputs

- One exported feature table per tool (feature m/z, retention time, intensity)
- Per-tool true-positive, false-positive, and missed-feature counts versus ground truth
- Accuracy metrics with significance tests and a combined cross-software comparison table (Fig. 2e style)

## How to apply

Process the same synthetic mzML file — 13,500 true single-peak and 13,500 true double-peak features — through each tool independently: MS-DIAL 4.9 in metabolomics (LC-MS) mode with the paper's standardized parameter set, MZmine 3.90 at reported defaults for mass detection, chromatogram building, deconvolution, and alignment, xcms 4.0.0 with centWave-based peak detection plus group alignment, and MassCube's untargeted_metabolomics_workflow (process_single_file). Export a feature table (m/z, retention time, intensity) from every tool so the four outputs share a comparable schema. Match each tool's detected features against the ground-truth feature list to count true positives, false positives, and missed features, then compute accuracy and pairwise significance — the paper reports p = 0.0029 vs MZmine3, p = 0.016 vs xcms, and p = 0.0065 vs MS-DIAL. Assemble the four accuracy values into a single comparison table mirroring Fig. 2e and verify the ordering: MassCube 96.5% > MZmine3 88.4% > xcms 87.4% > MS-DIAL 85.4%. Rationale: holding the raw file and standardized parameter set fixed makes accuracy differences attributable to algorithms — e.g., MassCube's mass-trace construction via signal clustering and Gaussian-filter assisted edge detection — rather than to preprocessing variance.

## Related tools

- **MassCube** (Python LC-MS processing framework under test; runs untargeted_metabolomics_workflow (process_single_file) on the shared mzML to produce its feature table) — https://github.com/huaxuyu/masscube
- **MS-DIAL** (Benchmark comparator; run in metabolomics (LC-MS) mode at the standardized parameter set to export its feature table)
- **MZmine3** (Benchmark comparator; mass detection, chromatogram building, deconvolution, and alignment at reported defaults before feature-table export)
- **XCMS** (Benchmark comparator; centWave-based peak detection and group alignment in R at reported defaults before feature-table export)
- **Python** (Runtime for installing MassCube (pip install masscube) and for scripting the TP/FP/missed-feature matching against ground truth)

## Examples

```
from masscube import untargeted_metabolomics_workflow; untargeted_metabolomics_workflow.process_single_file('synthetic_benchmark.mzML')
```

## Evaluation signals

- Accuracy ordering and values reproduce the paper: MassCube 96.5% > MZmine3 88.4% > xcms 87.4% > MS-DIAL 85.4% on the synthetic benchmark.
- Reported significance holds: p = 0.0029 vs MZmine3, p = 0.016 vs xcms, p = 0.0065 vs MS-DIAL.
- Per-tool TP + FP + missed-feature counts reconcile against the 27,000 ground-truth features (13,500 single-peak + 13,500 double-peak).
- All four feature tables derive from the identical mzML file under the standardized parameter set and share a comparable m/z / RT / intensity schema.
- MassCube's peak-detection run reports 100% signal coverage with comprehensive chromatographic metadata for quality assurance.

## Limitations

- External validated benchmarking datasets with large-scale ground-truth data are still missing, so accuracy results hinge on the assumptions of the synthetic benchmark file.
- Convoluted experimental data display characteristics difficult to explore with synthetic data (varying scan numbers across peaks, peak tailing), so accuracy numbers may not transfer to real samples.
- Results depend on the chosen standardized parameters; MS-DIAL, MZmine3, and xcms must be run at reported defaults, which may not be optimal for a given instrument or acquisition method.

## Evidence

- [abstract] MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy.: "MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy."
- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [intro] For example, XCMS6, MS-DIAL7, MZmine8, El-MAVEN9, OpenMS10, and TidyMass11 do not directly annotate ISFs and often report many false positive features: "For example, XCMS6, MS-DIAL7, MZmine8, El-MAVEN9, OpenMS10, and TidyMass11 do not directly annotate ISFs and often report many false positive features"
- [discussion] convoluted experimental data also display characteristics that are difficult to explore in a systematic way using synthetic data, such as varying scan numbers across peaks, peak tailing: "convoluted experimental data also display characteristics that are difficult to explore in a systematic way using synthetic data, such as varying scan numbers across peaks, peak tailing"
- [readme] masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing.: "masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
- [readme] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
