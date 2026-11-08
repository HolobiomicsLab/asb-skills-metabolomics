---
name: proteomics-benchmark-result-aggregation
description: Use when comparing peptide-level de novo sequencing performance of multiple
  tools (e.g., Casanovo vs PointNovo, DeepNovo, Novor) on a shared annotated MS/MS
  benchmark such as the nine-species dataset MassIVE MSV000090982.
when_to_use_negative:
- The set of proteins in the sample is known in advance and a standard sequence database
  search would suffice rather than a de novo benchmark.
- The data are DIA rather than DDA spectra — DeepNovo DDA outputs (and the DDA benchmark
  protocol) do not apply; a feature-based DIA framework in a separate repository is
  needed.
- Spectra are low quality with too few peaks and should first be discarded via the
  min_peaks filter rather than scored in the benchmark.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Casanovo
  role: Transformer-based de novo sequencer evaluated on each cross-validation fold;
    its peptide-level precision is the reference result being aggregated
  repo: https://github.com/Noble-Lab/casanovo
- name: DeepNovo
  role: CNN+RNN de novo baseline run on the same held-out test spectra per fold
  repo: https://github.com/nh2tran/DeepNovo
- name: PointNovo
  role: Order-invariant successor to DeepNovo used as a high-resolution de novo baseline
    in the benchmark comparison
- name: Novor
  role: Decision-tree-based de novo baseline run on the identical test spectra for
    every fold
- name: SearchGUI
  role: Common interface that can run Novor as a de novo engine within a benchmark
    pipeline
  repo: https://github.com/compomics/searchgui
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:17dd5c2327f5319757aa59e3e616b41bc9006d61fe99cfc52dbbc9d5e6e7cf81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - DeepNovo
  - Novor
  - PointNovo
  - SearchGUI
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# proteomics-benchmark-result-aggregation

## Summary

Aggregates peptide-level de novo sequencing benchmark results across tools and test splits into a single per-tool, per-species precision table, following the Casanovo cross-species evaluation protocol (Fig. 2a of Yilmaz et al. 2024).

## When to use

Use when comparing peptide-level de novo sequencing performance of multiple tools (e.g., Casanovo vs PointNovo, DeepNovo, Novor) on a shared annotated MS/MS benchmark such as the nine-species dataset MassIVE MSV000090982.

## When NOT to use

- The set of proteins in the sample is known in advance and a standard sequence database search would suffice rather than a de novo benchmark.
- The data are DIA rather than DDA spectra — DeepNovo DDA outputs (and the DDA benchmark protocol) do not apply; a feature-based DIA framework in a separate repository is needed.
- Spectra are low quality with too few peaks and should first be discarded via the min_peaks filter rather than scored in the benchmark.

## Inputs

- annotated MGF files of the nine-species benchmark (MassIVE MSV000090982)
- per-tool peptide prediction outputs (Casanovo, PointNovo, DeepNovo, Novor) on the same held-out test spectra
- cross-validation split definitions (held-out species as test set, remaining eight species as training set)

## Outputs

- per-tool, per-species peptide-level average precision table (e.g., denovo_benchmark_average_precision.tsv)
- aggregated average precision across the nine cross-validation folds for each tool

## How to apply

Download the annotated MGF files for the benchmark (MassIVE MSV000090982 for the nine-species cross-species dataset) and set up cross-validation splits, holding out each species in turn as the test set while training on the remaining eight. Run each de novo sequencer (Casanovo transformer model, PointNovo, DeepNovo, Novor) on the identical held-out test spectra per fold, then compute peptide-level average precision for each tool on each species split — this is the metric reported in Fig. 2a, where Casanovo reached 0.81 versus PointNovo 0.74, DeepNovo 0.70 and Novor 0.58. Aggregate the per-fold values by averaging across the nine leave-one-species-out folds and write both per-tool and per-species results to a tabular output such as denovo_benchmark_average_precision.tsv. Judge success by whether the reproduced average precisions match the reported values within reasonable tolerance, and by verifying every tool saw the same spectra so the comparison is fair.

## Related tools

- **Casanovo** (Transformer-based de novo sequencer evaluated on each cross-validation fold; its peptide-level precision is the reference result being aggregated) — https://github.com/Noble-Lab/casanovo
- **DeepNovo** (CNN+RNN de novo baseline run on the same held-out test spectra per fold) — https://github.com/nh2tran/DeepNovo
- **PointNovo** (Order-invariant successor to DeepNovo used as a high-resolution de novo baseline in the benchmark comparison)
- **Novor** (Decision-tree-based de novo baseline run on the identical test spectra for every fold)
- **SearchGUI** (Common interface that can run Novor as a de novo engine within a benchmark pipeline) — https://github.com/compomics/searchgui

## Evaluation signals

- Reproduced peptide-level average precisions match the reported Fig. 2a values (Casanovo 0.81, PointNovo 0.74, DeepNovo 0.70, Novor 0.58) within reasonable tolerance
- Results are reported per species and aggregated over nine leave-one-species-out folds, not on a single random split
- All four tools are evaluated on the identical held-out test spectra for every fold
- The aggregated per-tool and per-species results are written to a single TSV (denovo_benchmark_average_precision.tsv) with one row per tool-species pair
- The metric is peptide-level average precision, matching the evaluation metric used in the paper rather than amino-acid-level accuracy

## Limitations

- The field lacks an exhaustive and rigorous benchmark comparison of the growing set of de novo sequencing tools, so any aggregation over a subset of tools inherits that incompleteness.
- There is no rigorous method for confidence estimation for de novo sequencing — a key bottleneck — which limits how precision values from different tools can be statistically compared.
- Whether the number of model parameters impacts de novo sequencing performance is an open question left for future work, so differences in average precision may not be attributable to architecture alone.

## Evidence

- [methods] Cross-species benchmark source dataset: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [intro] Baseline tools compared: "PointNovo17, the successor to DeepNovo, leverages an order-invariant network architecture to model high-resolution MS/MS spectra18."
- [intro] Novor baseline: "the Novor algorithm15 improved the state of the art by using a decision tree as the score function for its dynamic programming algorithm."
- [abstract] Casanovo benchmark claim: "demonstrate that the model outperforms several state-of-the-art methods on a cross-species benchmark dataset"
- [discussion] Scope caveat on benchmarking: "Clearly, the field would benefit from an exhaustive and rigorous benchmark comparison of this growing field of tools."
- [readme] Novor runnable as de novo engine: [evidence span withheld — anchor status: external_doc]
