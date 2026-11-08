---
name: multi-algorithm-peak-detection-integration
description: Use when GC-MS metabolomics peak detection with a single algorithm (centWave
  or matchedFilter) misses peaks, and you want to merge the high-quality classified
  peak sets from multiple algorithms into one final peak table.
when_to_use_negative:
- Input is already a single integrated feature/peak table — the skill applies to merging
  raw per-algorithm peak sets
- Only one peak-picking algorithm is enabled in config.yaml — integration requires
  multiple algorithms' outputs
- The goal is parameter optimization or classifier training rather than combining
  final peak detection results
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: WiPP
  role: Runs the peak-picking pipeline (run_WiPP.sh pp) that performs per-algorithm
    peak detection, seven-class SVM quality classification, output classification
    and result integration across algorithms
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which WiPP is built; used to load classified peak tables, filter
    high-quality peaks, merge sets and write the comparison CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:b5f4343163f27680feb07a277970b90a19aada61d278babadb39d76acc0be4ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-algorithm-peak-detection-integration

## Summary

Integrates peak sets produced by multiple GC-MS peak-picking algorithms (centWave and matchedFilter, as embedded in WiPP) by merging the peaks each algorithm classified as high-quality by the seven-class SVM peak-quality classifier, yielding a larger high-quality peak set than either algorithm alone.

## When to use

Use when GC-MS metabolomics peak detection with a single algorithm (centWave or matchedFilter) misses peaks, and you want to merge the high-quality classified peak sets from multiple algorithms into one final peak table.

## When NOT to use

- Input is already a single integrated feature/peak table — the skill applies to merging raw per-algorithm peak sets
- Only one peak-picking algorithm is enabled in config.yaml — integration requires multiple algorithms' outputs
- The goal is parameter optimization or classifier training rather than combining final peak detection results

## Inputs

- Classified peak-detection tables from each peak-picking algorithm (centWave, matchedFilter) produced by WiPP's peak-picking stage on the full dataset
- SVM peak-quality classifier labels (seven peak classes) per detected peak
- Raw GC-MS input files in mzML, mzData or NetCDF format organized in the WiPP Input_folder with condition subdirectories

## Outputs

- One merged final high-quality peak set integrating classifier quality information across algorithms
- Coverage comparison summary table (e.g. peak_counts_comparison.csv) with high-quality counts per algorithm vs merged
- Medium- and low-quality peaks retained for further inspection

## How to apply

Run WiPP's peak-picking stage (run_WiPP.sh pp) so each enabled algorithm — e.g. centWave and matchedFilter — performs peak detection on the full dataset, then have the SVM-based seven-class peak-quality classifier assign quality labels to every detected peak. Load the per-algorithm classified peak tables, filter each to peaks labeled high-quality, and merge them into one integrated result set; the algorithms' outputs only partially overlap, so the union recovers more high-quality peaks than either alone (e.g. in dataset 1 matchedFilter found 137 peaks with 88 high-quality vs centWave 238 peaks with 144 high-quality). Compute per-algorithm vs merged high-quality counts and write the comparison (e.g. peak_counts_comparison.csv) to quantify coverage gain. Medium- and low-quality peaks are kept for further inspection rather than discarded.

## Related tools

- **WiPP** (Runs the peak-picking pipeline (run_WiPP.sh pp) that performs per-algorithm peak detection, seven-class SVM quality classification, output classification and result integration across algorithms) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which WiPP is built; used to load classified peak tables, filter high-quality peaks, merge sets and write the comparison CSV)

## Examples

```
cd projects/example_project && ../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- Merged high-quality peak count must equal or exceed the count from each individual algorithm (coverage-difference non-negative)
- Merged set size exceeds each single-algorithm set, consistent with partial overlap between centWave and matchedFilter outputs (e.g. 88 vs 144 high-quality in dataset 1, merged larger than both)
- All peaks in the final set carry the 'high-quality' class from the seven-class SVM classifier
- Summary comparison table (peak_counts_comparison.csv) contains per-algorithm and merged high-quality counts with a computed coverage difference

## Limitations

- Coverage gain depends on algorithm output overlap; if both algorithms detect nearly identical peaks, merging adds little
- Classifier accuracy depends on the annotated training peaks (e.g. 1200 peaks per algorithm by default); an under-annotated classifier mislabels quality and distorts the merged set
- WiPP was tested only on Ubuntu 16 and CentOS 7.6; Ubuntu 18 is unsupported due to incorporated R packages

## Evidence

- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [results] task_003 finding: [evidence span withheld — anchor status: not_found]
- [readme] README pipeline substeps: "Many sequential substeps are in fact happening during this final step of the pipeline such as classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection"
- [readme] README algorithms config note: "Do not forget to enable in the `config.yaml` file the peak picking algorithms you want to use."
