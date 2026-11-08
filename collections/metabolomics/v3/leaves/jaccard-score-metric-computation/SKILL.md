---
name: jaccard-score-metric-computation
description: Use when evaluating a bond- or site-level predictor whose output is a
  set of predicted locations (e.g., BoMX−Y pairs or SoMs) per molecule, and you need
  a single per-class score that penalizes false positives and false negatives symmetrically.
when_to_use_negative:
- When the prediction task is binary per molecule (e.g., CypReact's reactant/non-reactant
  classification) rather than set-valued per bond/atom — precision/recall or accuracy
  is more appropriate there
- When comparing tools evaluated on different datasets or with different per-instance
  matching rules; the Jaccard comparison (e.g., CypBoM vs FAME2 25% better) is only
  valid when both tools score identical [compound, bond] instances on the same dataset
  (EBoMD/HDFAME)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: CypBoM
  role: Predictor whose reactive-bond (BoMX−Y) outputs are scored with the Jaccard
    metric per CYP450 isoform
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: FAME2
  role: Competitor SoM predictor scored with the same Jaccard metric on EBoMD for
    benchmarking (CypBoM 25% better, Table 3 left)
- name: FAME3
  role: Competitor SoM predictor scored with the same Jaccard metric on EBoMD for
    benchmarking (CypBoM 36% better, Table 3 right)
- name: CyProduct
  role: Suite containing the CypBoM module whose cross-validated Jaccard scores are
    computed against EBoMD annotations
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:afd07fae51aa37e9f50e2887c89e4f929dce32fcdde78a38e89fdfc73233d807
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - FAME2
  - FAME3
  - WEKA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# jaccard-score-metric-computation

## Summary

Computes the Jaccard score TP/(TP+FN+FP) as the primary performance metric for set-valued predictions such as reactive bonds-of-metabolism (BoMX−Y) or sites-of-metabolism, where hits must match both the compound and the specific atom/bond. Used in the CypBoM/CyProduct evaluation to score per-enzyme prediction across the nine major human CYP450 isoforms.

## When to use

Use when evaluating a bond- or site-level predictor whose output is a set of predicted locations (e.g., BoMX−Y pairs or SoMs) per molecule, and you need a single per-class score that penalizes false positives and false negatives symmetrically.

## When NOT to use

- When the prediction task is binary per molecule (e.g., CypReact's reactant/non-reactant classification) rather than set-valued per bond/atom — precision/recall or accuracy is more appropriate there
- When comparing tools evaluated on different datasets or with different per-instance matching rules; the Jaccard comparison (e.g., CypBoM vs FAME2 25% better) is only valid when both tools score identical [compound, bond] instances on the same dataset (EBoMD/HDFAME)

## Inputs

- Binary annotation labels per [compound, bond] instance per isoform from EBoMD.sdf (828 η-η, 1348 η-H and 86 η-SPN bonds over 679 Zaretzki-derived substrates)
- Predicted reactive bond sets (BoMX−Y pairs) per isoform from CypBoM or a competitor tool (FAME2, FAME3, GLORY)
- Cross-validation fold assignments (external 5-fold) or hold-out dataset (EBoMD2.sdf) split definition

## Outputs

- Per-isoform Jaccard score TP/(TP+FN+FP) for each of the nine CYP450 enzymes (e.g., written to cypbom_cv_jaccard.csv)
- Aggregate score range across isoforms (e.g., 0.380–0.452 for CypBoMη−η under external 5-fold CV)
- Statistical significance of the score vs. a random classifier baseline (e.g., p-value)

## How to apply

For each predicted instance, count a true positive (TP) only when the predicted location exactly matches the annotated location (e.g., the canonically ordered η-η bond atom pair 〈i, j〉 equals the EBoMD-annotated reactive bond) for that [compound, isoform] pair; count missed annotations as false negatives (FN) and predictions without annotation as false positives (FP). Compute the Jaccard score as TP/(TP+FN+FP) per class — here per CYP450 isoform α ∈ {1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4} — rather than pooling instances, since class imbalance differs strongly per isoform. Evaluate under external 5-fold cross-validation (as done for CypBoM on EBoMD) or on a hold-out set (EBoMD2), and report the per-isoform score range. Compare against a random classifier baseline for significance, and against competitor tools (FAME2, FAME3, GLORY) using the same Jaccard metric on the same dataset to make the comparison meaningful.

## Related tools

- **CypBoM** (Predictor whose reactive-bond (BoMX−Y) outputs are scored with the Jaccard metric per CYP450 isoform) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **FAME2** (Competitor SoM predictor scored with the same Jaccard metric on EBoMD for benchmarking (CypBoM 25% better, Table 3 left))
- **FAME3** (Competitor SoM predictor scored with the same Jaccard metric on EBoMD for benchmarking (CypBoM 36% better, Table 3 right))
- **CyProduct** (Suite containing the CypBoM module whose cross-validated Jaccard scores are computed against EBoMD annotations) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\  # then compute Jaccard = TP/(TP+FN+FP) per enzyme against EBoMD annotations
```

## Evaluation signals

- Each of the nine per-isoform external 5-fold cross-validation Jaccard scores falls in the 0.380–0.452 range reported for CypBoM on EBoMD
- All per-isoform classifiers beat the random baseline with reported significance p < 3.2E-10
- TP is only counted on exact [compound, canonical bond] matches — canonically ordered η-η pairs (smaller atomic number first, then CA count, then lexicographic AtomDescriptor 10-tuple) must equal the annotation, not merely overlap
- When benchmarking, the same metric on the same instances reproduces the relative ordering: CypBoM Jaccard 25% better than FAME2 and 36% better than FAME3 for SoMs (Tables 3 left/right)
- Scores are computed per isoform, not pooled across enzymes, so the output table has exactly nine rows (one per CYP450 enzyme) plus any aggregate range

## Limitations

- The metric is sensitive to the canonical ordering rule for bond pairs; inconsistent atom ordering between annotation and prediction silently deflates TP counts
- Extreme class imbalance (only ~5% of η-η bonds are reactive vs ~95% non-reactive in EBoMD) means Jaccard scores are low in absolute terms and should always be compared to a random baseline
- Cost-sensitive training (false-positive cost β grid-searched in {2, 2.5, 3, 3.5, 4, 5, ..., 8} via internal 5-fold CV) directly trades off FP vs FN and therefore shifts the Jaccard optimum; comparisons across tools require identical [β, t, N] selection discipline to be fair

## Evidence
