---
name: binary-classifier-accuracy-evaluation
description: Use when a trained binary classifier (e.g. EVA CNN distinguishing good
  vs poor EIC plots) must be evaluated on a large manually labeled dataset and overall
  accuracy validated against a published performance claim.
when_to_use_negative:
- The evaluation goal is peak-level preprocessing or feature annotation rather than
  scoring an already-trained classifier against known labels
- Ground-truth labels do not exist and a manual inspection pass to assign good/poor
  EIC quality has not been done
- Inputs are raw MS data files (.mzXML/.mzML) rather than the labeled EIC plot images
  the classifier was built to consume
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: EVA
  role: C#-embedded CNN program whose classifier accuracy is being evaluated; also
    the Windows application for processing metabolic features
  repo: https://github.com/HuanLab/pyEVA
- name: pyEVA
  role: Python package used to instantiate the pretrained EVA CNN and run inference
    on each EIC plot during evaluation
  repo: https://github.com/HuanLab/pyEVA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:29430977e0a0001cb53245eb2466b96616ac74f09d203f4438249654ad8d4306
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/pyEVA
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# binary-classifier-accuracy-evaluation

## Summary

Benchmark a binary CNN classifier's end-to-end classification accuracy on a large manually annotated dataset by running inference on every sample image, comparing predictions against ground-truth labels, and computing the aggregate accuracy metric against a reported threshold (here, >90%).

## When to use

Use when a trained binary classifier (e.g. EVA CNN distinguishing good vs poor EIC plots) must be evaluated on a large manually labeled dataset and overall accuracy validated against a published performance claim.

## When NOT to use

- The evaluation goal is peak-level preprocessing or feature annotation rather than scoring an already-trained classifier against known labels
- Ground-truth labels do not exist and a manual inspection pass to assign good/poor EIC quality has not been done
- Inputs are raw MS data files (.mzXML/.mzML) rather than the labeled EIC plot images the classifier was built to consume

## Inputs

- 25095 manually annotated EIC plot images from 22 LC-MS-based metabolomics projects
- Manually assigned good/poor EIC quality labels for each EIC plot (ground truth)
- Pretrained EVA CNN model (via pyEVA Python package or C# EVA Windows application)

## Outputs

- Per-plot predicted good/poor class for all 25095 EIC plots (e.g. eva_evaluation_results.csv with predictions plus ground truth)
- Aggregate classification accuracy metric for the full evaluation set

## How to apply

Assemble the full evaluation set — in EVA's case, 25095 manually inspected EIC plots from 22 LC-MS-based metabolomics projects, each with a human-assigned good/poor EIC quality label. Instantiate the pretrained CNN via the pyEVA Python package (github.com/HuanLab/pyEVA) and run inference on each EIC plot image to obtain a predicted class. Compare every predicted label against the manual ground truth across all 25095 plots, compute the overall classification accuracy, and record per-plot predictions alongside the aggregate metric. Judge correctness by checking the aggregate accuracy exceeds the reported threshold (>90%), confirming the evaluation reproduces the paper's 'comprehensive evaluation' result.

## Related tools

- **EVA** (C#-embedded CNN program whose classifier accuracy is being evaluated; also the Windows application for processing metabolic features) — https://github.com/HuanLab/pyEVA
- **pyEVA** (Python package used to instantiate the pretrained EVA CNN and run inference on each EIC plot during evaluation) — https://github.com/HuanLab/pyEVA

## Examples

```
python main.py  # run pyEVA inference to obtain per-plot good/poor predictions for accuracy evaluation
```

## Evaluation signals

- Computed overall classification accuracy across all 25095 plots exceeds 90%, matching the reported comprehensive evaluation result
- Per-plot prediction file contains a class for every one of the 25095 manually inspected EIC plots (no missing predictions)
- Every predicted label maps to exactly one manually assigned good/poor ground-truth label for comparison
- Aggregate accuracy is computed over the full evaluation set, not a subset or random sample of the 22 projects

## Limitations

- Performance claim is specific to the 25095-plot, 22-project LC-MS evaluation set; accuracy on other data types or acquisition conditions is not established by this metric
- Evaluation requires exhaustive manual inspection to assign good/poor EIC quality labels before the accuracy comparison can be made
- Accuracy is an aggregate metric — per-class or per-project breakdowns are not described in the source context

## Evidence

- [abstract] EVA achieves >90% accuracy on the manually inspected evaluation set: "Our comprehensive evaluation of EVA indicates that it achieves over 90% classification accuracy."
- [abstract] Training set scope: 25095 plots from 22 projects with manual labels: "Our CNN model was trained using 25095 EIC plots collected from 22 LC-MS-based metabolomics projects of various sample types, LC and MS conditions."
- [abstract] Manual inspection step that creates ground-truth labels: "Notably, we manually inspected all the EIC plots to assign good or poor EIC quality for accurate model training."
- [readme] pyEVA as the Python vehicle for running the trained model: [evidence span withheld — anchor status: external_doc]
- [readme] Concrete pyEVA invocation producing per-plot predictions: [evidence span withheld — anchor status: external_doc]
