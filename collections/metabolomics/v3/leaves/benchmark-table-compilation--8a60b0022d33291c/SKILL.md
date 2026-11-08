---
name: benchmark-table-compilation
description: Use when two or more annotation tools must be evaluated side-by-side
  on identical benchmark datasets and the results need to be assembled into a per-assay
  table of metrics (computation time, sensitivity, specificity, F1) for direct comparison.
when_to_use_negative:
- When only a single tool is being profiled with no head-to-head comparison — a plain
  profiling script suffices instead of a comparative benchmark table
- When the input peak lists differ between methods (e.g. one tool receives a filtered
  subset) — tabulated runtime or metric comparisons would be confounded by unequal
  inputs
- When annotation quality cannot be scored against a reference compound list (no gold
  standard), leaving the table with computation time only and no performance column
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mWISE
  role: Candidate annotator being benchmarked; configured with FELLA network and raw-score
    diffusion so its per-assay computation time can be recorded and compared
  repo: https://dev.b2s.club/b2slab/mWISE
- name: xMSannotator
  role: Baseline state-of-the-art annotator timed on the identical peak lists to provide
    a direct comparison target
- name: mummichog
  role: Baseline annotator included in the benchmark table as a faster/less accurate
    comparator (F1 0.08–0.29, runtime 1.05–1.55 min)
- name: R
  role: Execution environment in which all annotators are run and from which per-assay
    timings and metrics are collected
- name: FELLA
  role: Provides the biological network used in the mWISE configuration whose computation
    time is being benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:3e567f4263f3104c9453b07067e253295e9c2b4e5dc0a7ac72e57963f8eed799
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - mummichog
  - xMSannotator
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# benchmark-table-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compile a per-assay, per-method benchmark table (e.g. wall-clock computation times or sensitivity/F1 metrics) that compares an emerging tool such as mWISE against established annotators like xMSannotator, mummichog, MI-Pack and CAMERA on the same six LC-MS benchmark assays.

## When to use

Use when two or more annotation tools must be evaluated side-by-side on identical benchmark datasets and the results need to be assembled into a per-assay table of metrics (computation time, sensitivity, specificity, F1) for direct comparison.

## When NOT to use

- When only a single tool is being profiled with no head-to-head comparison — a plain profiling script suffices instead of a comparative benchmark table
- When the input peak lists differ between methods (e.g. one tool receives a filtered subset) — tabulated runtime or metric comparisons would be confounded by unequal inputs
- When annotation quality cannot be scored against a reference compound list (no gold standard), leaving the table with computation time only and no performance column

## Inputs

- Filtered LC-MS peak lists from each benchmark assay (same input for every method being compared)
- Reference compound annotations per assay for computing TP/FP/FN, sensitivity, specificity and F1
- Wall-clock timing from each annotator run (e.g. seconds or minutes per assay)

## Outputs

- Per-assay benchmark table keyed by assay and method (e.g. mWISE FELLA-raw, xMSannotator, mummichog, MI-Pack, CAMERA)
- Columns for TP, FP, FN, sensitivity, specificity, F1, and/or computation time per assay
- Range summaries (e.g. min–max runtime or F1) for each method across assays

## How to apply

Run each candidate annotator — mWISE (with a fixed configuration such as FELLA network + raw scores diffusion), xMSannotator, mummichog, MI-Pack — on the same filtered LC-MS peak lists from each of the six benchmark assays, recording wall-clock time or annotation metrics per run. For mWISE, fix the pipeline (matching Stage via GenomicRanges, features Clustering/DBSCAN spectral clustering, cluster Based.filter, set.diffusion via DiffuStats on the FELLA network with raw scores) so that per-dataset timing is reproducible. Aggregate results into a single table keyed by assay and method, e.g. per-assay runtimes in minutes (mWISE FELLA-raw 15.70–53.38 min vs xMSannotator 106.34–318.04 min across Assays 1–6). Verify the tabulated values against reported ranges and include enough columns (TP, FP, FN, sensitivity, specificity, F1) to support both performance and computation-time comparisons.

## Related tools

- **mWISE** (Candidate annotator being benchmarked; configured with FELLA network and raw-score diffusion so its per-assay computation time can be recorded and compared) — https://dev.b2s.club/b2slab/mWISE
- **xMSannotator** (Baseline state-of-the-art annotator timed on the identical peak lists to provide a direct comparison target)
- **mummichog** (Baseline annotator included in the benchmark table as a faster/less accurate comparator (F1 0.08–0.29, runtime 1.05–1.55 min))
- **R** (Execution environment in which all annotators are run and from which per-assay timings and metrics are collected)
- **FELLA** (Provides the biological network used in the mWISE configuration whose computation time is being benchmarked)

## Examples

```
Rscript -e 't0<-Sys.time(); res<-mWISE::mWISE(peakList, adducts=..., network=FELLA, diffusion=rawScores); print(Sys.time()-t0)'
```

## Evaluation signals

- Per-assay runtime ranges must match reported values — mWISE FELLA-raw spans 15.70–53.38 minutes and xMSannotator 106.34–318.04 minutes across Assays 1–6
- Every method is run on the same filtered peak list per assay; if any tool consumed a different input subset, the timing or metric row is invalid
- Each performance column (TP, FP, FN, sensitivity, specificity, F1) must be present per assay so relative ranking of methods can be recomputed from the table
- F1 values in the compiled table should reproduce the reported per-assay figures (e.g. xMSannotator 0.15–0.22, mummichog 0.08–0.29 across assays)
- All six assays appear as separate rows for each method; missing an assay (e.g. Assay 3 runtime absent) breaks the range verification

## Limitations

- mWISE only supports KEGG as an annotation database, which is an important limitation with respect to xMSannotator (KEGG, HMDB, Lipid Maps, T3DB) and biases pure performance comparisons
- MI-Pack is far slower (1892.27–5547.60 minutes) and mummichog much faster (1.05–1.55 minutes), so runtime comparisons alone cannot substitute for performance metrics
- Timing depends on the fixed mWISE configuration (matching Stage, cluster Based.filter, diffusion parameters); changing the configuration invalidates the recorded per-assay runtimes

## Evidence

- [results] xMSannotator runs 106.34–318.04 min; mWISE FELLA-raw 15.70–53.38 min: [evidence span withheld — anchor status: not_found]
