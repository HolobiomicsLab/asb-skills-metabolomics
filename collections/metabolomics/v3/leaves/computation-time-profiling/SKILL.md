---
name: computation-time-profiling
description: 'Decision support for selecting an LC-MS annotation algorithm based on
  computation time: time each annotator on identical filtered peak lists across multiple
  assays and compare per-dataset runtimes.'
when_to_use_negative:
- Input peak lists have not been filtered (signal-free features retained or 80% rule
  not applied), making runtime comparisons non-comparable to the benchmark
- The annotator under test does not support KEGG-only databases or operates with a
  different feature-matching configuration than the mWISE FELLA-raw setup
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0218
tools:
- name: mWISE
  role: Annotator under test; runtime is profiled with the FELLA-raw diffusion configuration
    (matching Stage, features Clustering/DBSCAN, cluster Based.filter, set.diffusion
    via DiffuStats)
  repo: https://github.com/b2slab/mWISE
- name: xMSannotator
  role: State-of-the-art baseline annotator whose per-dataset computation time is
    benchmarked against mWISE
- name: FELLA
  role: Supplies the biological network and raw scores diffusion configuration used
    in the timed mWISE runs
- name: Genomic Ranges
  role: Used by mWISE matching Stage to rapidly match LC-MS peaks to the KEGG database
- name: Diffu Stats
  role: Used by set.diffusion to diffuse the label vector in the FELLA network during
    timed runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:02edafe2506937f27de8c4d0e412964b9b8214407800f2a5f7693027369039e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# computation-time-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-dataset wall-clock runtime profiling of LC-MS feature annotation pipelines, benchmarking mWISE (FELLA-raw diffusion configuration) against xMSannotator, mummichog, MI-Pack and CAMERA on the same filtered peak lists.

## When to use

Decision support for selecting an LC-MS annotation algorithm based on computation time: time each annotator on identical filtered peak lists across multiple assays and compare per-dataset runtimes.

## When NOT to use

- Input peak lists have not been filtered (signal-free features retained or 80% rule not applied), making runtime comparisons non-comparable to the benchmark
- The annotator under test does not support KEGG-only databases or operates with a different feature-matching configuration than the mWISE FELLA-raw setup

## Inputs

- Filtered benchmark dataset peak lists (LC-MS features, signal-free features removed, 80% rule applied)
- mWISE annotation pipeline (FELLA-raw diffusion configuration)
- xMSannotator run configuration

## Outputs

- Per-dataset computation times (wall-clock seconds) for each annotator
- Runtime comparison table across Assays 1–6

## How to apply

Run each annotator on the same filtered benchmark peak lists (LC-MS features without signal removed, 80% rule applied where intensity equals 0). For mWISE, run the full workflow — matching Stage with GenomicRanges against KEGG, features Clustering/DBSCAN spectral clustering, cluster Based.filter, and set.diffusion via DiffuStats with the FELLA-raw configuration — and record wall-clock seconds per dataset. Run xMSannotator on the identical peak lists and record its computation time per dataset. Assemble a single per-dataset runtime table; the article reports mWISE FELLA-raw at 15.70–53.38 s versus xMSannotator at 106.34–318.04 s across Assays 1–6 (MI-Pack is far slower at 1892.27–5547.60 s; mummichog is much faster at 1.05–1.55 s). Verify the measured ranges align with these reported values before drawing conclusions.

## Related tools

- **mWISE** (Annotator under test; runtime is profiled with the FELLA-raw diffusion configuration (matching Stage, features Clustering/DBSCAN, cluster Based.filter, set.diffusion via DiffuStats)) — https://github.com/b2slab/mWISE
- **xMSannotator** (State-of-the-art baseline annotator whose per-dataset computation time is benchmarked against mWISE)
- **FELLA** (Supplies the biological network and raw scores diffusion configuration used in the timed mWISE runs)
- **Genomic Ranges** (Used by mWISE matching Stage to rapidly match LC-MS peaks to the KEGG database)
- **Diffu Stats** (Used by set.diffusion to diffuse the label vector in the FELLA network during timed runs)

## Examples

```
Rscript time_mwise_vs_xmsannotator.R --input filtered_peaks.csv --config FELLA-raw
```

## Evaluation signals

- Per-dataset runtimes for mWISE FELLA-raw fall within the reported 15.70–53.38 s range across Assays 1–6
- Per-dataset runtimes for xMSannotator fall within the reported 106.34–318.04 s range across Assays 1–6
- All annotators are timed on identical filtered peak lists (80% rule applied where intensity equals 0)
- Runtime table covers all six benchmark assays for every annotator compared

## Limitations

- mWISE only offers annotation in the KEGG database; xMSannotator supports KEGG, HMDB, Lipid Maps and T3DB, so runtime comparisons reflect different database coverage
- mummichog is substantially faster (1.05–1.55 s) but achieves lower F1 (0.08–0.29) than mWISE, so runtime alone is not a sufficient selection criterion

## Evidence

- [results] Per-assay runtimes (minutes): mWISE FELLA-raw 15.70/53.38/43.40/26.45/17.84/35.07 for Assays 1-6: [evidence span withheld — anchor status: not_found]
