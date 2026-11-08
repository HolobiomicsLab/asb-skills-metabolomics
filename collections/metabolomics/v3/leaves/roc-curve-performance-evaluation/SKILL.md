---
name: roc-curve-performance-evaluation
description: Use when multiple peak-curation strategies (e.g., active-learning scores,
  CV-of-QC-peak-area filters, NeatMS) must be ranked by how well their scores separate
  manually annotated true peaks from artifacts.
when_to_use_negative:
- No manual annotations of true peaks versus artifacts exist, so there is no ground
  truth to compute a ROC curve against
- The goal is unsupervised peak filtering with a fixed cutoff rather than comparative
  benchmarking of curation methods
- Input is a single method's binary pass/fail filter output with no continuous score,
  since ROC requires score variation across thresholds
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Computes ROC curves and AUC values for each peak-curation method against the
    manual annotations
- name: matplotlib
  role: Plots all ROC curves together, reproducing Figure 2a-b
- name: PeakDetective
  role: Provides the active-learning classifier scores and the supervised (250-peak)
    baseline scores being evaluated by the ROC analysis
  repo: https://github.com/pattilab/PeakDetective
- name: NeatMS
  role: Pre-trained model whose peak classifications serve as one of the benchmarked
    curation approaches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:461bcf4554643c6c6cc91e8c08dba2ac4765126b077650b5b789eba56a3eb1be
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - NeatMS
  - PeakDetective
  - TensorFlow
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# roc-curve-performance-evaluation

## Summary

Compute ROC curves and AUROC values to benchmark peak-curation methods against manual annotations of true peaks versus artifacts, as done in PeakDetective's evaluation of active learning against CV filtering, peak-area filtering, NeatMS, and supervised training on Dataset 1 (negative-mode human plasma HILIC/TOF LC/MS).

## When to use

Use when multiple peak-curation strategies (e.g., active-learning scores, CV-of-QC-peak-area filters, NeatMS) must be ranked by how well their scores separate manually annotated true peaks from artifacts.

## When NOT to use

- No manual annotations of true peaks versus artifacts exist, so there is no ground truth to compute a ROC curve against
- The goal is unsupervised peak filtering with a fixed cutoff rather than comparative benchmarking of curation methods
- Input is a single method's binary pass/fail filter output with no continuous score, since ROC requires score variation across thresholds

## Inputs

- Per-peak curation scores for each method (PeakDetective active-learning scores, CV of QC peak areas, peak-area magnitude, NeatMS model outputs)
- Manually annotated ground-truth labels of true peaks versus artifacts
- PeakDetective supervised baseline scores from training on 250 manually annotated peaks

## Outputs

- ROC curves for all methods overlaid in a single matplotlib figure (Figure 2a-b reproduction)
- AUC table quantifying each method's AUROC
- FDR and TPR estimates per score cutoff from validation data

## How to apply

For each curation approach, obtain a per-peak score: PeakDetective active-learning classifier scores, coefficient of variation of QC peak areas, peak-area magnitude, or the pre-trained NeatMS model's classification. Treat the manual annotations of true peaks versus artifacts as ground-truth labels and compute ROC curves and AUC values for every method using scikit-learn, then plot all curves together with matplotlib to reproduce Figure 2a-b. The comparison criterion is that the active-learning PeakDetective ROC curve lies above the supervised-with-250-peaks baseline (trained on 250 manually annotated peaks) and above the filter-based baselines. Report FDR and TPR estimates based on the validation data for each score cutoff when judging operating points.

## Related tools

- **scikit-learn** (Computes ROC curves and AUC values for each peak-curation method against the manual annotations)
- **matplotlib** (Plots all ROC curves together, reproducing Figure 2a-b)
- **PeakDetective** (Provides the active-learning classifier scores and the supervised (250-peak) baseline scores being evaluated by the ROC analysis) — https://github.com/pattilab/PeakDetective
- **NeatMS** (Pre-trained model whose peak classifications serve as one of the benchmarked curation approaches)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(true_labels, peak_scores); auc = roc_auc_score(true_labels, peak_scores)
```

## Evaluation signals

- Active-learning PeakDetective AUROC exceeds the supervised (250-peak) PeakDetective AUROC
- Active-learning PeakDetective ROC curve lies above both the supervised baseline and the filter-based baselines (CV filtering, peak-area filtering, NeatMS)
- All ROC curves are computed against the same manually annotated true-peak/artifact ground truth and plotted together for direct comparison
- FDR and TPR estimates based on the validation data are reported for each cutoff
- AUC values and the saved figure are exported as an AUC table and figure file

## Limitations

- FDR and TPR estimates depend on the small validation set of manually labeled peaks (e.g., the numValPeaks validation peaks labeled before active-learning rounds), so cutoff-level estimates carry sampling uncertainty
- Performance is specific to the LC/MS method and sample type; PeakDetective must be tailored to the dataset (here, negative-mode human plasma HILIC/TOF) to maximize performance
- The supervised baseline is limited to 250 manually annotated peaks, so the comparison reflects low-label regimes

## Evidence

- [methods] ROC/AUC benchmarking against baselines: [evidence span withheld — anchor status: not_found]
- [methods] Active learning superiority criterion: [evidence span withheld — anchor status: not_found]
