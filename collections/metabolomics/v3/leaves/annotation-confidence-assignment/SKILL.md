---
name: annotation-confidence-assignment
description: Use when a MetDNA3/MrnAnnoAlgo3 result table from recursive network annotation
  propagation (getMetAnno()) must be turned into a confidence-assigned, redundancy-filtered
  annotation table before computing coverage or Top 1/3/10 correct rates and Top 3
  FDR.
when_to_use_negative:
- The input is raw LC-MS/MS data not yet processed into an MS1 feature table and MS2
  (.mgf/.msp) spectra — confidence assignment operates on propagation result tables,
  not raw spectra
- The task is discovering metabolites absent from human metabolome databases (no seed/database
  annotations exist for network propagation to start from)
- Annotations come from a single-layer tool (e.g., GNPS FBMN, IIMN, NetID) whose own
  confidence scoring scheme is being used — do not apply MetDNA3 confidence levels
  to foreign pipelines
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3 (MrnAnnoAlgo3)
  role: Provides assignConfidenceInMrn3(), calculateRedundancyInMrn3(), and rmRedunInMrn3()
    used to assign confidence levels and remove redundant annotations from propagated
    result tables
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline single-layer MRN pipeline whose confidence assignment/output format
    MetDNA3's results are benchmarked against (39.9% coverage, 65.6% Top 3 correct
    rate)
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:97ea0fa30d4cddda7329df73c28d9e23fa6fa25c48c4356a4c1d68470bfc3607
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# annotation-confidence-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns per-candidate annotation confidence levels to network-propagated metabolite annotations (MetDNA3's two-layer knowledge/data network output) and removes redundant N-to-1 annotations, so that coverage and Top-N correct-rate/FDR benchmarks are computed on a de-duplicated, confidence-ranked result table.

## When to use

Use when a MetDNA3/MrnAnnoAlgo3 result table from recursive network annotation propagation (getMetAnno()) must be turned into a confidence-assigned, redundancy-filtered annotation table before computing coverage or Top 1/3/10 correct rates and Top 3 FDR.

## When NOT to use

- The input is raw LC-MS/MS data not yet processed into an MS1 feature table and MS2 (.mgf/.msp) spectra — confidence assignment operates on propagation result tables, not raw spectra
- The task is discovering metabolites absent from human metabolome databases (no seed/database annotations exist for network propagation to start from)
- Annotations come from a single-layer tool (e.g., GNPS FBMN, IIMN, NetID) whose own confidence scoring scheme is being used — do not apply MetDNA3 confidence levels to foreign pipelines

## Inputs

- MetDNA3/MrnAnnoAlgo3 annotation result table (result_df) from getMetAnno() network propagation, one per cross-validation fold or sample run
- Seed metabolite annotations (e.g., from Supplementary Data 2: seed and validation metabolite annotations)
- MS1 peak table (.csv with name, mz, rt as first three columns) and MS2 spectra (.mgf or .msp) used during propagation
- MRN model data (md_mrn) and adduct/RT prediction artifacts from predictRtInMrn3() and getAdductAnno()

## Outputs

- Confidence-assigned, redundancy-removed result_df annotation table with per-candidate confidence levels
- Per-fold and aggregated benchmark metrics (coverage, Top 1/3/10 correct rates, Top 3 FDR) written to cv_annotation_benchmark.csv

## How to apply

After recursive propagation from seed metabolites across the two-layer network, pass each fold's result table to assignConfidenceInMrn3(result_df) to attach confidence levels to every propagated metabolite annotation. Then run calculateRedundancyInMrn3(result_df) and rmRedunInMrn3(result_df, is_only_confidence_assignment) to collapse redundant annotations where multiple features map to the same metabolite (N-to-1 cases), optionally deferring full redundancy cleanup with rmRedunInMetdna3(result_df, wd_output, num_candi, is_known_prioritization, is_clean_Nto1, md_mrn, column) until the full MetDNA3 run. Candidate scoring upstream relies on getErrorAndScore() with m/z ppm threshold, RT tolerance (rt_tol_rel) and CCS tolerance (ccs_tol_rel) parameters, and MS2 spectral matching parameters set via getMatchParam(). In the paper's 10-fold cross-validation benchmark (30% seed / 70% validation on 20 Orbitrap datasets), this confidence-assignment step yields Top 3 correct rates of 84.4% and Top 3 FDR of 15.6%, which are the reference values for judging correct application.

## Related tools

- **MetDNA3 (MrnAnnoAlgo3)** (Provides assignConfidenceInMrn3(), calculateRedundancyInMrn3(), and rmRedunInMrn3() used to assign confidence levels and remove redundant annotations from propagated result tables) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline single-layer MRN pipeline whose confidence assignment/output format MetDNA3's results are benchmarked against (39.9% coverage, 65.6% Top 3 correct rate)) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
result_df <- assignConfidenceInMrn3(result_df); result_df <- rmRedunInMrn3(result_df, is_only_confidence_assignment)
```

## Evaluation signals

- Aggregated 10-fold cross-validation (30% seed / 70% validation) coverage reaches ~68.1% for MetDNA3 vs the MetDNA2 baseline of 39.9%
- Top 1/3/10 correct rates reproduce ~68.0%/84.4%/91.0% and Top 3 FDR ~15.6%, matching Source Data values for Fig. 3e–g within reasonable tolerance
- Every propagated annotation in the result table carries a confidence level after assignConfidenceInMrn3() — no rows left unassigned
- Redundant N-to-1 annotations are reduced after rmRedunInMrn3(): the number of candidate rows per feature decreases relative to the pre-filter table
- Per-fold metrics and aggregates are written to cv_annotation_benchmark.csv with all 10 folds present

## Limitations

- Isomeric metabolites may be indistinguishable based on MS2 spectra alone, so FDR estimates from MS2-dependent validation steps may be optimistic
- No FDR is reported for the curated MRN itself due to the lack of clearly defined non-RPs (negative examples) under screening conditions
- Cross-tool concordance may overestimate performance because chemical standards are lacking; validation tools may also have included validation metabolites in their training sets

## Evidence

- [abstract] Overall, this strategy significantly improves the coverage, accuracy, and efficiency of metabolite annotation and is freely available as MetDNA3.: "Overall, this strategy significantly improves the coverage, accuracy, and efficiency of metabolite annotation and is freely available as MetDNA3."
- [readme] An efficient topology-based annotation propagation algorithm leveraging both network layers to enhance annotation coverage and accuracy.: "An efficient topology-based annotation propagation algorithm leveraging both network layers to enhance annotation coverage and accuracy."
- [discussion] isomeric metabolites may be indistinguishable based on MS2 spectra alone. Consequently, both network-based annotation propagation and validation: "isomeric metabolites may be indistinguishable based on MS2 spectra alone. Consequently, both network-based annotation propagation and validation"
- [discussion] cross-tool concordance may overestimate performance due to the lack of chemical standards: "cross-tool concordance may overestimate performance due to the lack of chemical standards"
