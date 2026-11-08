---
name: benchmark-metric-computation
description: Use when comparing peak-detection or feature-detection performance of
  LC-MS processing tools against a ground-truth feature list (synthetic mzML with
  known single/double peaks) or manually labeled extracted-ion chromatograms, and
  accuracy metrics must be computed per category.
when_to_use_negative:
- The goal is metabolite identification confidence scoring, not peak-detection accuracy
  benchmarking — MassCube does not directly perform metabolite identification.
- Input is already a processed, aligned feature table with no ground-truth reference;
  accuracy against truth cannot be computed without labels or a synthetic feature
  list.
- Comparing annotation of in-source fragments or isomer detection speed rather than
  chromatographic peak-shape classification, where different metrics apply.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MassCube
  role: Target tool being benchmarked; run via untargeted_metabolomics_workflow (process_single_file)
    to produce the feature table whose detections are matched against ground truth
  repo: https://github.com/huaxuyu/masscube
- name: MS-DIAL
  role: Comparator run in metabolomics (LC-MS) mode 4.9 with the paper's standardized
    parameter set; its feature table supplies detection counts for accuracy computation
- name: MZmine3
  role: Comparator run as MZmine 3.90 with standardized mass detection, chromatogram
    building, deconvolution, and alignment parameters; its exported feature table
    feeds the metric computation
- name: XCMS
  role: Comparator run as xcms 4.0.0 in R with centWave-based peak detection at default
    settings; its feature table is matched against ground truth to compute accuracy
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:ad30b8437d7b64cd263209756855956c51f86d32112de9a0653146498ac5c355
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# benchmark-metric-computation

## Summary

Compute accuracy-style benchmark metrics (per-category accuracy, feature-table comparisons, significance tests) when evaluating LC-MS peak-detection software such as MassCube against MS-DIAL, MZmine3, and xcms on synthetic or manually labeled ground-truth data.

## When to use

Use when comparing peak-detection or feature-detection performance of LC-MS processing tools against a ground-truth feature list (synthetic mzML with known single/double peaks) or manually labeled extracted-ion chromatograms, and accuracy metrics must be computed per category.

## When NOT to use

- The goal is metabolite identification confidence scoring, not peak-detection accuracy benchmarking — MassCube does not directly perform metabolite identification.
- Input is already a processed, aligned feature table with no ground-truth reference; accuracy against truth cannot be computed without labels or a synthetic feature list.
- Comparing annotation of in-source fragments or isomer detection speed rather than chromatographic peak-shape classification, where different metrics apply.

## Inputs

- Ground-truth synthetic mzML feature list (13,500 true single peaks and 13,500 true double peaks)
- Manually labeled EIC metadata list with peak-shape class for each of 722 extracted-ion chromatograms
- Per-software feature tables exported from MS-DIAL 4.9, MZmine 3.90, xcms 4.0.0, and MassCube

## Outputs

- peak_detection_accuracy.csv with double-peak and single-peak accuracy scores and per-image agreement flags
- Single comparison table of accuracy metrics across all four tools mirroring Fig. 2e
- Counts of true positives, false positives, and missed features per software

## How to apply

Assemble a ground-truth reference before computing any metric: for the synthetic benchmark this is the feature list behind the mzML file containing 13,500 true single peaks and 13,500 true double peaks; for the experimental benchmark it is the metadata list accompanying 722 pre-labeled EIC PNG images from Zenodo record 14159704. Run each tool (MassCube's untargeted_metabolomics_workflow / process_single_file, MS-DIAL 4.9, MZmine 3.90, xcms 4.0.0 with centWave) on the same input under standardized/default parameters and export its feature table. Match detected features to ground truth to count true positives, false positives, and missed features, or classify each detected peak as double-peak vs single-peak. Compute per-category accuracy as the fraction of ground-truth items agreeing with the software's prediction (reference values: 93.5% double peaks, 97.3% single peaks on labeled EICs; 96.5% average synthetic accuracy for MassCube vs 88.4% MZmine3, 87.4% xcms, 85.4% MS-DIAL). For head-to-head comparisons, assemble results into a single comparison table and assess significance (reported p-values: 0.0029, 0.016, 0.0065).

## Related tools

- **MassCube** (Target tool being benchmarked; run via untargeted_metabolomics_workflow (process_single_file) to produce the feature table whose detections are matched against ground truth) — https://github.com/huaxuyu/masscube
- **MS-DIAL** (Comparator run in metabolomics (LC-MS) mode 4.9 with the paper's standardized parameter set; its feature table supplies detection counts for accuracy computation)
- **MZmine3** (Comparator run as MZmine 3.90 with standardized mass detection, chromatogram building, deconvolution, and alignment parameters; its exported feature table feeds the metric computation)
- **XCMS** (Comparator run as xcms 4.0.0 in R with centWave-based peak detection at default settings; its feature table is matched against ground truth to compute accuracy)

## Examples

```
pip install masscube && python -c "from masscube import untargeted_metabolomics_workflow; untargeted_metabolomics_workflow(process_single_file('synthetic_benchmark.mzML'))"
```

## Evaluation signals

- Computed MassCube accuracies must match the reported reference values: 93.5% double-peak and 97.3% single-peak accuracy on the 722 labeled EICs, and 96.5% average accuracy on the synthetic benchmark
- Comparator accuracies must reproduce the reported ordering and magnitudes: MZmine3 88.4% (p = 0.0029), xcms 87.4% (p = 0.016), MS-DIAL 85.4% (p = 0.0065), with MassCube significantly outperforming all three
- True positives + false positives + missed features per software must reconcile with the total ground-truth count (27,000 synthetic features; 722 labeled EICs)
- peak_detection_accuracy.csv must contain one row per labeled EIC with a per-image agreement flag and both per-category accuracy scores
- The comparison table's accuracy ordering and values must be consistent with the paper's Fig. 2e result

## Limitations

- External validated benchmarking datasets with large-scale ground-truth data are still missing, limiting generalization beyond synthetic and curated-label benchmarks
- Characteristics of convoluted experimental data (varying scan numbers across peaks, peak tailing) are difficult to explore systematically with synthetic data
- Human-defined rules for peak shapes reflected in algorithmic parameters cannot account for all peak-shape scenarios, so accuracy is bounded by parameterization

## Evidence

- [results] MassCube achieved the highest accuracy for double peaks (93.5%) and single peaks (97.3%), outperforming MS-DIAL, MZmine3, and xcms on the 722 manually labeled extracted ion chromatograms.: [evidence span withheld — anchor status: not_found]
- [results] Synthetic benchmark accuracies: [evidence span withheld — anchor status: not_found]
- [methods] Matching protocol: [evidence span withheld — anchor status: not_found]
- [methods] Metric definition: [evidence span withheld — anchor status: not_found]
- [intro] Benchmarking gap: "Existing software has often not been thoroughly validated and benchmarked for data processing efficiency and accuracy"
- [abstract] Accuracy claim: "MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy."
- [discussion] Missing external benchmarks: "External validated datasets for rigorous benchmarking are still missing."
- [discussion] Synthetic-data limitation: "convoluted experimental data also display characteristics that are difficult to explore in a systematic way using synthetic data, such as varying scan numbers across peaks"
