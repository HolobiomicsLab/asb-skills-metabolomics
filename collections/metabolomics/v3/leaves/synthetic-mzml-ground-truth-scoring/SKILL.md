---
name: synthetic-mzml-ground-truth-scoring
description: Use when comparing peak-detection accuracy of LC-MS data-processing tools
  on synthetic mzML data where the true feature list is known, e.g. ranking MassCube,
  MS-DIAL, MZmine3, and xcms by TP/FP/missed-feature counts.
when_to_use_negative:
- Input is an experimental (non-synthetic) LC-MS file with no ground-truth feature
  list — accuracy cannot be scored without known true peaks
- Input is already a processed feature table — peak-detection benchmarking requires
  raw mzML as the common starting point
- Goal is metabolite identification confidence rather than peak-detection accuracy
  — synthetic single-peak/double-peak data with ground truth does not test annotation
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MassCube
  role: Python framework whose untargeted metabolomics workflow (process_single_file)
    generates the feature table being scored; achieved the highest accuracy (96.5%)
    on the synthetic benchmark
  repo: https://github.com/huaxuyu/masscube
- name: MS-DIAL
  role: Baseline peak-detection tool run in metabolomics (LC-MS) mode at version 4.9
    on the same synthetic mzML file (85.4% accuracy)
- name: MZmine3
  role: Baseline run at version 3.90 with mass detection, chromatogram building, deconvolution,
    and alignment at reported defaults (88.4% accuracy)
- name: XCMS
  role: Baseline run at version 4.0.0 in R with centWave-based peak detection and
    group alignment at default settings (87.4% accuracy)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:420ba0c121493b2e7ba0228e4fb09cb2ec8dcd90dc3aebecf7fe37970d068f2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MS-DIAL
  - MZmine3
  - MassCube
  - XCMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# synthetic-mzml-ground-truth-scoring

## Summary

Benchmark nontargeted LC-MS peak-detection software (MassCube, MS-DIAL, MZmine3, xcms) on a synthetic mzML file containing 13,500 true single peaks and 13,500 true double peaks, scoring detected features against a known ground-truth feature list.

## When to use

Use when comparing peak-detection accuracy of LC-MS data-processing tools on synthetic mzML data where the true feature list is known, e.g. ranking MassCube, MS-DIAL, MZmine3, and xcms by TP/FP/missed-feature counts.

## When NOT to use

- Input is an experimental (non-synthetic) LC-MS file with no ground-truth feature list — accuracy cannot be scored without known true peaks
- Input is already a processed feature table — peak-detection benchmarking requires raw mzML as the common starting point
- Goal is metabolite identification confidence rather than peak-detection accuracy — synthetic single-peak/double-peak data with ground truth does not test annotation

## Inputs

- Synthetic mzML file containing 13,500 true single peaks and 13,500 true double peaks
- Ground-truth feature list for the synthetic data
- Standardized parameter sets for MS-DIAL 4.9, MZmine 3.90, and xcms 4.0.0
- MassCube (Python) with untargeted_metabolomics_workflow

## Outputs

- Feature table per software (MS-DIAL, MZmine3, xcms, MassCube)
- Per-tool true positive, false positive, and missed feature counts
- Accuracy metrics comparison table mirroring Fig. 2e (MassCube 96.5% vs MZmine3 88.4%, xcms 87.4%, MS-DIAL 85.4%)

## How to apply

Process the same synthetic mzML file with each tool under standardized parameters: MS-DIAL 4.9 in metabolomics (LC-MS) mode, MZmine 3.90 with mass detection, chromatogram building, deconvolution, and alignment at reported defaults, xcms 4.0.0 in R with centWave-based peak detection and group alignment, and MassCube's untargeted metabolomics workflow (process_single_file). Export each tool's feature table, then match detected features against the synthetic-data ground-truth feature list to count true positives, false positives, and missed features per tool. Compute accuracy per tool and assemble a single comparison table mirroring the paper's Fig. 2e. Rationale: synthetic data with a known ground truth removes the ambiguity of experimental data, where 'existing software has often not been thoroughly validated and benchmarked'; MassCube's peak detection constructs mass traces through signal clustering and Gaussian-filter assisted edge detection, giving 100% signal coverage.

## Related tools

- **MassCube** (Python framework whose untargeted metabolomics workflow (process_single_file) generates the feature table being scored; achieved the highest accuracy (96.5%) on the synthetic benchmark) — https://github.com/huaxuyu/masscube
- **MS-DIAL** (Baseline peak-detection tool run in metabolomics (LC-MS) mode at version 4.9 on the same synthetic mzML file (85.4% accuracy))
- **MZmine3** (Baseline run at version 3.90 with mass detection, chromatogram building, deconvolution, and alignment at reported defaults (88.4% accuracy))
- **XCMS** (Baseline run at version 4.0.0 in R with centWave-based peak detection and group alignment at default settings (87.4% accuracy))

## Examples

```
pip install masscube
```

## Evaluation signals

- Assembled accuracy table's ordering matches the paper: MassCube (96.5%) > MZmine3 (88.4%, p = 0.0029) > xcms (87.4%, p = 0.016) > MS-DIAL (85.4%, p = 0.0065)
- Each tool's TP + FP - missed-feature counts reconcile with the ground-truth list of 13,500 single peaks and 13,500 double peaks
- All four tools were run on the identical synthetic mzML file with standardized, reported parameter sets (no per-tool tuning)
- Comparison table structure mirrors Fig. 2e with accuracy values and significance p-values per tool

## Limitations

- Convoluted experimental data display characteristics difficult to explore systematically with synthetic data, such as varying scan numbers across peaks and peak tailing
- External validated benchmarking datasets with large-scale ground-truth data are still missing, so synthetic-data results may not generalize
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios

## Evidence

- [abstract] MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy.: "MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy."
- [intro] Existing software has often not been thoroughly validated and benchmarked: "Existing software has often not been thoroughly validated and benchmarked for data processing efficiency and accuracy of final reports that start from raw data imports and management"
- [abstract] Peak detection via mass traces: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] README capabilities: "It provides: Highly accurate nontargeted peak detection and segmentation."
