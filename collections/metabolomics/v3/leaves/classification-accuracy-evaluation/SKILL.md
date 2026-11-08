---
name: classification-accuracy-evaluation
description: Use when a CNN-classified set of EIC plots from LC-MS feature tables
  needs benchmarking against manually annotated good/poor labels, or when verifying
  that an extended pyEVA deployment maintains reported accuracy.
when_to_use_negative:
- The input feature table is already a validated output from the EVA/pyEVA pipeline
  and no ground-truth labels are available for comparison — accuracy cannot be computed
  without manual annotation.
- The task is to generate or smooth EIC plots themselves rather than to assess classifier
  performance on already-generated plots.
- Analyzing GC-MS or non-chromatographic MS data where EIC quality does not apply
  as the classification target.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: EVA
  role: C#-based Windows application whose embedded CNN model performs the good/poor
    EIC classification being evaluated; benchmark >90% accuracy is reported for this
    classifier.
  repo: github.com/HuanLab-backup/EVA
- name: pyEVA
  role: Python rewrite of EVA used to run EIC extraction and CNN inference on new
    vendor formats; its outputs (PredictionOutcomes.csv) are compared against manual
    labels for accuracy calculation.
  repo: https://github.com/HuanLab/pyEVA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:bb194d15fa54a143cda96e72164484df24ef213650080b10f4e8795db86ff4f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# classification-accuracy-evaluation

## Summary

Evaluate a deep learning classifier's peak-quality predictions against manually assigned good/poor labels for extracted ion chromatograms (EICs) from LC-MS metabolomics feature tables, computing overall classification accuracy against the >90% benchmark reported for the EVA CNN model trained on 25095 annotated EIC plots.

## When to use

Use when a CNN-classified set of EIC plots from LC-MS feature tables needs benchmarking against manually annotated good/poor labels, or when verifying that an extended pyEVA deployment maintains reported accuracy.

## When NOT to use

- The input feature table is already a validated output from the EVA/pyEVA pipeline and no ground-truth labels are available for comparison — accuracy cannot be computed without manual annotation.
- The task is to generate or smooth EIC plots themselves rather than to assess classifier performance on already-generated plots.
- Analyzing GC-MS or non-chromatographic MS data where EIC quality does not apply as the classification target.

## Inputs

- Feature table (.csv with m/z and rt columns plus intensity data)
- Matching .mzXML or .mzML raw LC-MS files
- CNN-classified EIC plots with good/poor labels
- Manually assigned good/poor EIC quality labels for the same features

## Outputs

- Overall classification accuracy (fraction of EICs whose CNN label matches manual label)
- Per-format accuracy table (vendor_format_accuracy.csv)
- PredictionOutcomes.csv with Boolean good/poor predictions per feature

## How to apply

Run the trained CNN (embedded in EVA/pyEVA, pretrained on 25095 manually annotated EIC plots from 22 LC-MS metabolomics projects) over EIC plots generated from your feature table and matching .mzXML or .mzML files. Compare each model-assigned good/poor EIC label against manually inspected ground-truth labels for the same features, and compute overall classification accuracy. The article's comprehensive evaluation of EVA establishes a benchmark of over 90% classification accuracy, so deviations below this range indicate a problem with input parsing, EIC rendering, or the classifier environment. When extending pyEVA to new LC-MS vendor formats, repeat the accuracy comparison per format and write results to vendor_format_accuracy.csv.

## Related tools

- **EVA** (C#-based Windows application whose embedded CNN model performs the good/poor EIC classification being evaluated; benchmark >90% accuracy is reported for this classifier.) — github.com/HuanLab-backup/EVA
- **pyEVA** (Python rewrite of EVA used to run EIC extraction and CNN inference on new vendor formats; its outputs (PredictionOutcomes.csv) are compared against manual labels for accuracy calculation.) — https://github.com/HuanLab/pyEVA

## Examples

```
python main.py
```

## Evaluation signals

- Overall classification accuracy should exceed 90% when evaluated against manually assigned good/poor EIC labels, consistent with the paper's comprehensive evaluation of EVA.
- PredictionOutcomes.csv should contain a Boolean value for every metabolic feature present in the input feature table.
- FinalTable.csv should contain results for the same number of rows as the input feature table (m/z, rt, and classification outcome columns).
- When comparing across LC-MS vendor formats, per-format accuracy figures in vendor_format_accuracy.csv should each approximate the reported benchmark rather than collapsing below ~90% for a working deployment.
- The number of EIC plots rendered in the classifier/EICplots folder should match the number of features in the input feature table, ensuring no features were silently dropped before classification.

## Limitations

- Accuracy depends on the quality of manual good/poor EIC annotation; the CNN was trained on 25095 manually inspected EIC plots from 22 LC-MS projects, so labels from very different sample types or LC/MS conditions may degrade performance.
- pyEVA requires specific environment setup (Anaconda conda environment from environment.yml, Python 3.7.12, R with xcms/ggplot2/scales/dplyr packages); version mismatches in PyCharm or VS Code may cause differences.
- The evaluation benchmark (>90% accuracy) is specific to EIC peak-quality classification for LC-MS metabolomics feature fidelity and may not transfer to other chromatographic peak tasks.

## Evidence

- [abstract] CNN trained on 25095 annotated EIC plots: "Our CNN model was trained using 25095 EIC plots collected from 22 LC-MS-based metabolomics projects of various sample types, LC and MS conditions."
- [abstract] >90% accuracy benchmark: "Our comprehensive evaluation of EVA indicates that it achieves over 90% classification accuracy."
- [abstract] Manual ground-truth annotation workflow: "Notably, we manually inspected all the EIC plots to assign good or poor EIC quality for accurate model training."
- [readme] pyEVA CLI invocation and output artifacts: [evidence span withheld — anchor status: external_doc]
- [readme] pyEVA prediction output file: [evidence span withheld — anchor status: external_doc]
- [readme] pyEVA input feature table format: [evidence span withheld — anchor status: external_doc]
