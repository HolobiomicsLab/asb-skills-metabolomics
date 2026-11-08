---
name: topk-accuracy-and-fdr-evaluation
description: Use when a tool's annotation quality must be quantified against known
  metabolite identities — e.g. comparing MetDNA3 vs MetDNA2 coverage/correct rate
  on feature tables with known ground-truth annotations split into seed and validation
  sets.
when_to_use_negative:
- The dataset has no known metabolite annotations to serve as seeds and validation
  targets, so no ground-truth correct rate or FDR can be computed
- The goal is de novo annotation of unknown metabolites (e.g. discovery of metabolites
  absent from databases), not tool benchmarking against reference annotations
- Data are not untargeted metabolomics feature tables (MS1/MS2 from Orbitrap-type
  acquisition); the seed/validation propagation framework requires feature-level MS1
  and MS2 inputs
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3
  role: 'Annotation engine under evaluation: run via MrnAnnoAlgo() per CV fold, with
    assignConfidenceInMrn3() and rmRedunInMrn3() post-processing before scoring coverage
    and Top-k rates'
  repo: github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline annotation tool whose coverage (39.9%) and Top 3 correct rate (65.6%)
    define the reference point the benchmark must exceed
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:1e60ba9619f10d0fedba4863be8740227b5b09fb8769d17146a3bc037ff54cea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA2
  - MetDNA3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# topk-accuracy-and-fdr-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking protocol for metabolite annotation accuracy: run 10-fold cross-validation (30% seed / 70% validation) over 20 Orbitrap datasets and compute annotation coverage, Top 1/3/10 correct rates, and Top 3 FDR to quantify whether MetDNA3's two-layer network propagation beats a baseline such as MetDNA2.

## When to use

Use when a tool's annotation quality must be quantified against known metabolite identities — e.g. comparing MetDNA3 vs MetDNA2 coverage/correct rate on feature tables with known ground-truth annotations split into seed and validation sets.

## When NOT to use

- The dataset has no known metabolite annotations to serve as seeds and validation targets, so no ground-truth correct rate or FDR can be computed
- The goal is de novo annotation of unknown metabolites (e.g. discovery of metabolites absent from databases), not tool benchmarking against reference annotations
- Data are not untargeted metabolomics feature tables (MS1/MS2 from Orbitrap-type acquisition); the seed/validation propagation framework requires feature-level MS1 and MS2 inputs

## Inputs

- Supplementary Data 2: seed and validation metabolite annotations for the 20 Orbitrap untargeted metabolomics datasets
- Source Data file for Fig. 3e–g (reported coverage and correct-rate values for tolerance checking)
- Per-fold MetDNA3 and MetDNA2 annotation result tables produced by MrnAnnoAlgo()
- Cross-validation fold assignments (30% seed / 70% validation splits of known metabolites)

## Outputs

- cv_annotation_benchmark.csv with per-fold and aggregate metrics: annotation coverage, Top 1/3/10 correct rates, Top 3 FDR
- Per-fold validation-set coverage compared against the MetDNA2 baseline (39.9%)
- Aggregate benchmark values for comparison with Source Data Fig. 3e–g

## How to apply

Load Supplementary Data 2 (seed and validation metabolite annotations) plus the Source Data file for Fig. 3e–g, then partition known metabolite annotations into 10 cross-validation folds with 30% of metabolites as seeds and 70% as validation targets. For each fold, run MetDNA3 (MrnAnnoAlgo3) via MrnAnnoAlgo(), using getMetAnno() to recursively propagate annotations from seed metabolites across the two-layer knowledge/data network, then assign confidence levels with assignConfidenceInMrn3() and remove redundant annotations with rmRedunInMrn3(). Compute per-fold annotation coverage on validation metabolites against the MetDNA2 baseline (39.9% coverage). Aggregate across all 10 folds to obtain Top 1/3/10 correct rates and Top 3 FDR, writing per-fold and aggregate metrics to cv_annotation_benchmark.csv. Success is judged by reproducing the reported values: coverage 68.1% vs 39.9%, Top 1/3/10 correct rates of 68.0%/84.4%/91.0%, and Top 3 FDR of 15.6%, matching Source Data Fig. 3e–g within reasonable tolerance.

## Related tools

- **MetDNA3** (Annotation engine under evaluation: run via MrnAnnoAlgo() per CV fold, with assignConfidenceInMrn3() and rmRedunInMrn3() post-processing before scoring coverage and Top-k rates) — github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline annotation tool whose coverage (39.9%) and Top 3 correct rate (65.6%) define the reference point the benchmark must exceed) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- Aggregated 10-fold coverage for MetDNA3 equals 68.1% (vs 39.9% MetDNA2 baseline)
- Aggregate Top 1/3/10 correct rates equal 68.0%/84.4%/91.0%
- Aggregate Top 3 FDR equals 15.6%
- Per-fold metrics written to cv_annotation_benchmark.csv reproduce Source Data Fig. 3e–g values within reasonable tolerance
- Each fold uses the specified 30% seed / 70% validation partition of known metabolite annotations

## Limitations

- Validation tools may have included validation metabolites in their training sets, potentially inflating measured accuracy
- MS2 spectra often cannot distinguish isomeric metabolites, so FDR estimates based on MS2 spectral matching may be optimistic
- Cross-tool concordance may overestimate performance because chemical standards are lacking

## Evidence

- [results] coverage and correct rate benchmark: [evidence span withheld — anchor status: not_found]
