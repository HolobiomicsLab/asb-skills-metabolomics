---
name: binary-classification-metric-computation
description: Use when a compound-identification run yields binary detection/containment
  calls per compound (e.g. mcfNMR assignment values vs. a detection threshold) and
  you need F1, precision and recall against a known mixture composition for benchmarking.
when_to_use_negative:
- You only have quantified concentrations and no binary detection calls — the paper
  notes limited quantification accuracy, and concentration reconstruction should be
  evaluated with regression-style metrics instead of F1.
- The library contains compounds absent from the ground truth in a way that invalidates
  binary labeling — the paper states the exact impact is difficult to predict, so
  conclusions about individual compounds are unreliable.
- The target spectrum has not been processed by any identification algorithm — there
  are no predictions to score.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3420
tools:
- name: mcfNMR
  role: Produces the per-compound detection/containment results that are scored with
    precision/recall/F1; its result pickles and CSV tables are the direct inputs to
    the metric computation.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Used in the underlying mcfNMR pipeline (HiGHS interface) whose classification
    outputs are evaluated; also usable for custom metric computation on prediction/ground-truth
    lists.
- name: MetaboMiner
  role: Baseline identification algorithm whose published F1 scores (0.65 on the artificial
    mixtures) are compared against the mcfNMR setups.
- name: COLMAR-HSQC
  role: Baseline identification algorithm whose published F1 scores (0.64 on the artificial
    mixtures) are compared against the mcfNMR setups.
- name: SMART-Miner
  role: Baseline identification algorithm whose published F1 scores (0.68 on the artificial
    mixtures) are compared against the mcfNMR setups.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:28826383accb27dfd03cbf0996e332a65973b0377139c506bdb3b74ec2077895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - COLMAR-HSQC
  - MetaboMiner
  - SMART-Miner
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# binary-classification-metric-computation

## Summary

Computes precision, recall and F1 scores for compound identification results by comparing predicted detections/containment calls against a known ground-truth constituent list, as used to benchmark mcfNMR on the artificial mixtures N925, N987 and N988 and the plasma sample N907/N926.

## When to use

Use when a compound-identification run yields binary detection/containment calls per compound (e.g. mcfNMR assignment values vs. a detection threshold) and you need F1, precision and recall against a known mixture composition for benchmarking.

## When NOT to use

- You only have quantified concentrations and no binary detection calls — the paper notes limited quantification accuracy, and concentration reconstruction should be evaluated with regression-style metrics instead of F1.
- The library contains compounds absent from the ground truth in a way that invalidates binary labeling — the paper states the exact impact is difficult to predict, so conclusions about individual compounds are unreliable.
- The target spectrum has not been processed by any identification algorithm — there are no predictions to score.

## Inputs

- Per-compound binary detection/containment predictions (mcfNMR mcf-results '.pickle' or '.csv' result tables)
- Known ground-truth constituent list of the analyzed mixture (e.g. 21–27 compounds for N925, N987, N988)
- Configuration parameters used for the run (detection threshold ϑ, assignment radius r, setup variant A–D)

## Outputs

- Precision, recall and F1 score per run/mixture
- Average F1 across mixtures for each method variant (A–D)
- Relative F1 margin over baselines (MetaboMiner, COLMAR-HSQC, SMART-Miner)

## How to apply

Run the identification workflow (e.g. mcfNMR with a 501-compound MetaboMiner/Wishart library on targets N925, N987, N988 or plasma samples N907/N926), then extract per-compound detection/containment results from the output mcf-results pickles or CSV tables. Treat each compound as a binary classification instance: an assignment above the detection threshold indicates containment, and compare these predictions with the known constituent lists (21–27 compounds per artificial mixture). Compute precision, recall and the F1 score (the harmonic mean) per run and, for multi-setup comparisons, average F1 across mixtures. Expectation from the paper: the best MCF setups reach F1 ≈ 0.83 (incremental setup C), 0.81 (single-pass A), 0.80 (isolated B), 0.78 (peak-list D), exceeding MetaboMiner (0.65), COLMAR-HSQC (0.64) and SMART-Miner (0.68) by a relative margin of ~15%; on human blood plasma F1 values were only slightly larger than 0.5. Report F1 alongside the detection threshold and assignment radius used, since no definite protocol exists for choosing these parameters.

## Related tools

- **mcfNMR** (Produces the per-compound detection/containment results that are scored with precision/recall/F1; its result pickles and CSV tables are the direct inputs to the metric computation.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Used in the underlying mcfNMR pipeline (HiGHS interface) whose classification outputs are evaluated; also usable for custom metric computation on prediction/ground-truth lists.)
- **MetaboMiner** (Baseline identification algorithm whose published F1 scores (0.65 on the artificial mixtures) are compared against the mcfNMR setups.)
- **COLMAR-HSQC** (Baseline identification algorithm whose published F1 scores (0.64 on the artificial mixtures) are compared against the mcfNMR setups.)
- **SMART-Miner** (Baseline identification algorithm whose published F1 scores (0.68 on the artificial mixtures) are compared against the mcfNMR setups.)

## Examples

```
mcfNMR -c config_setupC.toml  # then compute precision/recall/F1 from output results.csv against the N925/N987/N988 constituent lists
```

## Evaluation signals

- Per-compound predictions and ground-truth lists must cover the same compound namespace (e.g. the 501-compound Wishart/MetaboMiner library) before computing TP/FP/FN counts.
- F1 should equal the harmonic mean of the computed precision and recall; recompute manually as a sanity check.
- Average F1 over N925/N987/N988 should fall near the reported values: ~0.83 for setup C, ~0.81 for A, ~0.80 for B, ~0.78 for D.
- The best MCF setup should exceed MetaboMiner (F1=0.65), COLMAR-HSQC (F1=0.64) and SMART-Miner (F1=0.68) by roughly a 15% relative margin.
- Results depend on detection_threshold and assignment_radius; record and report both, and check that no detections in a blank/noise spectrum survive the chosen threshold ϑ.

## Limitations

- No definite answer or protocol exists for choosing the assignment radius r, absorption cost cø, or detection threshold ϑ, so F1 scores are parameter-dependent.
- Performance diminishes for mixtures of increasing complexity, and overlapping peaks impede complete reconstruction, so F1 on very complex samples will be depressed.
- On human blood plasma the tested methods achieved F1 scores only slightly larger than 0.5, so benchmark numbers from artificial mixtures do not transfer directly to natural samples.
- Peak shifting, shape changes, or disappearance depending on sample matrix and temperature affects detection calls and hence the metrics.

## Evidence

- [results] best MCF setup exceeds baselines by 15% relative F1 margin: "our method achieved higher F1 scores than all other
methods (by a relative margin of 15%)"
- [discussion] plasma F1 slightly larger than 0.5: "They
achieved F1 scores of values slightly larger than 0.5, pointing toward
a potential for further refinements"
- [discussion] no protocol for parameter choice: "To date, we have no definite
answer or protocol to choose these values"
- [readme] detection threshold indicates containment: "`detection_threshold`:  Detection threshold (assignment > th indicates containment)."
- [readme] mcfNMR writes result pickles and CSV tables: "Output directory for the result files (mcf-results with extension '.pickle', results with extension '.csv' and graphics"
- [intro] benchmark on 501-compound library and artificial mixtures: "compare its performance on a standard classification
task, mapping a library of 501 compounds onto artificial mixtures
of 21–27 compounds"
- [intro] overlapping peaks impede complete reconstruction: "overlapping peaks. Since peaks have a considerable
extension, such overlaps occur frequently in complex mixtures"
