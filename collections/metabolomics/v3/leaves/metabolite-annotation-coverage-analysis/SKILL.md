---
name: metabolite-annotation-coverage-analysis
description: Use when you need to quantify how much of a known metabolite set a network-based
  annotation pipeline recovers (coverage, Top 1/3/10 correct rate, Top 3 FDR) by hiding
  a fraction of known annotations and testing their recovery from seed metabolites.
when_to_use_negative:
- The goal is annotation of a new experimental dataset (production annotation) rather
  than benchmarking recovery of known annotations — use the full MetDNA3/MetDNA2 annotation
  workflow instead.
- 'Comparing tools whose training sets may overlap with the validation metabolites:
  the article notes ''these tools may have included part of validation metabolites
  in their training, potentially contributing to the high accuracy'', so cross-tool
  coverage comparison may be inflated.'
- Cross-tool concordance without chemical standards — the article warns 'cross-tool
  concordance may overestimate performance due to the lack of chemical standards'.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3 (MrnAnnoAlgo3)
  role: Two-layer network-based annotation propagation engine; getMetAnno(), assignConfidenceInMrn3(),
    and rmRedunInMrn3() produce the benchmarked annotations per fold
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline single-layer (knowledge-guided multi-layer metabolic network) method
    whose 39.9% coverage and 65.6% Top 3 correct rate serve as the comparison point
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:78780bdaf58d61281af0417ca34f6f39977afb620de7f38df0df525c4b585be5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA2
  - MetDNA3
  - MetDNA3 (MrnAnnoAlgo3)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-annotation-coverage-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metabolite annotation coverage and correct rate by partitioning known annotations into 30% seed / 70% validation folds under 10-fold cross-validation, propagating annotations from seeds across the two-layer (knowledge + data) network with MetDNA3/MrnAnnoAlgo3, and comparing against the MetDNA2 baseline.

## When to use

Use when you need to quantify how much of a known metabolite set a network-based annotation pipeline recovers (coverage, Top 1/3/10 correct rate, Top 3 FDR) by hiding a fraction of known annotations and testing their recovery from seed metabolites.

## When NOT to use

- The goal is annotation of a new experimental dataset (production annotation) rather than benchmarking recovery of known annotations — use the full MetDNA3/MetDNA2 annotation workflow instead.
- Comparing tools whose training sets may overlap with the validation metabolites: the article notes 'these tools may have included part of validation metabolites in their training, potentially contributing to the high accuracy', so cross-tool coverage comparison may be inflated.
- Cross-tool concordance without chemical standards — the article warns 'cross-tool concordance may overestimate performance due to the lack of chemical standards'.

## Inputs

- Supplementary Data 2: seed and validation metabolite annotations from 20 Orbitrap untargeted metabolomics datasets
- Source Data file for Fig. 3e–g (published benchmark metrics)
- MS1 peak table (.csv with 'name', 'mz', 'rt' columns) and MS2 data files (.mgf or .msp) for network construction
- Metabolic reaction network (MRN) object md_mrn with GNN-predicted reaction relationships

## Outputs

- cv_annotation_benchmark.csv with per-fold and aggregate coverage, Top 1/3/10 correct rates, and Top 3 FDR
- Per-fold annotation tables with confidence levels assigned (assignConfidenceInMrn3) and redundancy removed (rmRedunInMrn3)

## How to apply

Load the seed and validation metabolite annotations (Supplementary Data 2) and the Source Data file for Fig. 3e–g, then split known metabolite annotations into 10 cross-validation folds with 30% of metabolites as seeds and 70% as validation targets. For each fold, run the MrnAnnoAlgo3 algorithm via MrnAnnoAlgo(), using getMetAnno() to recursively propagate annotations from seed metabolites across the two-layer knowledge/data network built by pre-mapping experimental features through sequential MS1 m/z matching, reaction relationship mapping, and MS2 similarity constraints. Post-process each fold's result table with assignConfidenceInMrn3() to assign confidence levels and rmRedunInMrn3() to remove redundant annotations. Compute per-fold annotation coverage on validation metabolites and, after aggregating across all 10 folds, Top 1/3/10 correct rates and Top 3 FDR, comparing against the MetDNA2 baseline. Validation succeeds if aggregated metrics reproduce the published values: MetDNA3 coverage 68.1% vs MetDNA2 39.9%, Top 1/3/10 correct rates 68.0%/84.4%/91.0%, and Top 3 FDR 15.6%, within reasonable tolerance of the Source Data for Fig. 3e–g.

## Related tools

- **MetDNA3 (MrnAnnoAlgo3)** (Two-layer network-based annotation propagation engine; getMetAnno(), assignConfidenceInMrn3(), and rmRedunInMrn3() produce the benchmarked annotations per fold) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline single-layer (knowledge-guided multi-layer metabolic network) method whose 39.9% coverage and 65.6% Top 3 correct rate serve as the comparison point) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); anno <- getMetAnno(table_anno, idx_seeds, round, obj_mrn, table_ms2_network_filter, step_mrn, topN_neighbor, lib_type_mrn, thread)
```

## Evaluation signals

- Aggregated 10-fold coverage equals 68.1% for MetDNA3 vs 39.9% for MetDNA2 (within tolerance of Source Data Fig. 3e–g).
- Aggregated Top 1/3/10 correct rates equal 68.0% / 84.4% / 91.0% (Top 3 improved from MetDNA2's 65.6%).
- Aggregated Top 3 FDR equals 15.6%, consistent with 100% − 84.4% Top 3 correct rate.
- Each fold's result table has passed assignConfidenceInMrn3() and rmRedunInMrn3() before metric computation (no redundant or unassigned-confidence rows remain).
- Fold split respects the 30% seed / 70% validation partition of known metabolite annotations in every one of the 10 folds.

## Limitations

- Validation tools may have included part of the validation metabolites in their training, potentially inflating accuracy.
- Cross-tool concordance may overestimate performance due to the lack of chemical standards.
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so MS2-matching-based FDR estimates may be optimistic.
- The MetDNA2 R package ships without the copyright-restricted library objects (zhuMetLib, zhuMetlib_orbitrap, zhuRPlib, lib_rt, lib_ccs); a benchmark requires user-supplied libraries or the webserver.

## Evidence

- [results] compared to MetDNA2, the average network-based annotation coverage in the 10-fold cross-validation increased from 39.9% to 68.1%: [evidence span withheld — anchor status: not_found]
