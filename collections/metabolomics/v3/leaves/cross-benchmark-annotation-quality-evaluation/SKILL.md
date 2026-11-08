---
name: cross-benchmark-annotation-quality-evaluation
description: Use when benchmarking an LC-MS annotation tool (e.g. mWISE, xMSannotator,
  mummichog) against alternatives on multiple feature peak sets, and you need to quantify
  performance (sensitivity, F1), computation time, and structural closeness of proposed
  compounds via Tanimoto similarity.
when_to_use_negative:
- The input is already an annotated compound list with no reference peaks or known
  original identifiers — there is no ground truth to compute TP/FN/FP or Tanimoto
  similarity against
- The annotator only offers databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB)
  while the evaluation pipeline is built around KEGG-compound prioritization, as mWISE
  only offers KEGG data
- A single study with one assay is available — cross-benchmark aggregation across
  the three benchmark studies is required for the sensitivity/SD and Tanimoto distribution
  comparisons
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: mWISE
  role: Annotator under evaluation; provides KEGG-compound prioritized lists per LC-MS
    peak via matching, cluster-based filter, and diffusion scoring (FELLA or RClass
    configurations)
  repo: https://github.com/b2slab/mWISE
- name: xMSannotator
  role: State-of-the-art baseline annotator run on the same benchmark peak sets for
    head-to-head comparison of sensitivity, F1, computation time, and Tanimoto similarity
- name: mummichog
  role: Alternative annotator benchmarked in the results (best F1 0.29 on Assay 6)
- name: FELLA
  role: R package providing the biological network used in one of the four mWISE diffusion
    configurations evaluated
- name: Diffu Stats
  role: R package used by mWISE's set.diffusion function to diffuse the label vector
    in the given network during diffusion scoring
- name: Genomic Ranges
  role: R package used by mWISE's matching Stage to rapidly match LC-MS peaks to the
    KEGG database considering a set of adducts and fragments
- name: R
  role: Runtime environment for running mWISE, xMSannotator, and the evaluation scripts
    (rmarkdown-based)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:bf7338700de14938d42a91d56caaf5928a4c30fb894348f3e4739acb81cb5666
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
  - mummichog
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cross-benchmark-annotation-quality-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate and compare metabolite annotation algorithms across multiple benchmark LC-MS feature peak sets using sensitivity, F1-score, computation time, and Tanimoto structural similarity of proposed compounds.

## When to use

Use when benchmarking an LC-MS annotation tool (e.g. mWISE, xMSannotator, mummichog) against alternatives on multiple feature peak sets, and you need to quantify performance (sensitivity, F1), computation time, and structural closeness of proposed compounds via Tanimoto similarity.

## When NOT to use

- The input is already an annotated compound list with no reference peaks or known original identifiers — there is no ground truth to compute TP/FN/FP or Tanimoto similarity against
- The annotator only offers databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) while the evaluation pipeline is built around KEGG-compound prioritization, as mWISE only offers KEGG data
- A single study with one assay is available — cross-benchmark aggregation across the three benchmark studies is required for the sensitivity/SD and Tanimoto distribution comparisons

## Inputs

- Public benchmark metabolomics LC-MS feature peak sets (e.g. three assay peak lists with positive and negative acquisition mode objects, as in the Trypanosoma dataset)
- Per-compound structure InChIKey representations
- Known original compound identifiers for reference peaks
- Annotation tool outputs: prioritized lists of KEGG compounds per LC-MS peak (top three candidates per peak)

## Outputs

- Per-peak Tanimoto chemical-structure similarity values between original and proposed compound structures
- Aggregated per-tool Tanimoto similarity statistics and comparison table (e.g. tanimoto_similarity_comparison.csv)
- Per-assay confusion-matrix counts (TP, FN, FP) and derived sensitivity, specificity, and F1-score
- Computation time per tool and per configuration

## How to apply

Retrieve the public benchmark metabolomics feature peak sets together with each compound's structure InChIKey and known original compound identifier. Run each annotator on the same peak sets to obtain the top three proposed candidates per peak — for mWISE this means the matching stage against the KEGG database with a set of adducts and fragments, the cluster-based filter to reduce false positives, and diffusion-based scoring with either the FELLA network (raw score) or RClass network (z score). For each annotated peak, calculate the Tanimoto chemical-structure similarity between the original compound structure and each proposed candidate using their InChIKey representations, then aggregate per-tool similarity distributions across the three benchmark studies. Compare tools on TP/FN/FP counts (F1 up to 0.49 for mWISE FELLA on Assay 6 vs 0.22 for xMSannotator), sensitivity (mWISE mean 0.63 SD 0.07 vs xMSannotator 0.55 SD 0.19), and computation time, and write aggregated per-tool Tanimoto statistics and per-peak values to a structured comparison table.

## Related tools

- **mWISE** (Annotator under evaluation; provides KEGG-compound prioritized lists per LC-MS peak via matching, cluster-based filter, and diffusion scoring (FELLA or RClass configurations)) — https://github.com/b2slab/mWISE
- **xMSannotator** (State-of-the-art baseline annotator run on the same benchmark peak sets for head-to-head comparison of sensitivity, F1, computation time, and Tanimoto similarity)
- **mummichog** (Alternative annotator benchmarked in the results (best F1 0.29 on Assay 6))
- **FELLA** (R package providing the biological network used in one of the four mWISE diffusion configurations evaluated)
- **Diffu Stats** (R package used by mWISE's set.diffusion function to diffuse the label vector in the given network during diffusion scoring)
- **Genomic Ranges** (R package used by mWISE's matching Stage to rapidly match LC-MS peaks to the KEGG database considering a set of adducts and fragments)
- **R** (Runtime environment for running mWISE, xMSannotator, and the evaluation scripts (rmarkdown-based))

## Examples

```
Rscript run_mWISE_xMSannotator_benchmark.R --assays assay1,assay2,assay6 --configs FELLA_raw,RClass_z --out tanimoto_similarity_comparison.csv
```

## Evaluation signals

- Per-peak Tanimoto hit ratios across assays fall in the reported range, e.g. 0.59–0.76 for mWISE's FELLA and RClass configurations, with values such as 0.69 for Assay 1 (FELLA raw score) and 0.65 for Assay 6 (RClass z score)
- Sensitivity mean (SD) across datasets for mWISE reaches 0.63 (0.07) versus xMSannotator's 0.55 (0.19) under the FELLA-network-plus-raw-scores configuration
- Assay 1 matching stage yields 144 TP, 13 FN, and 157 references, giving sensitivity 0.92 — verify counts against reference peak totals
- F1-scores per assay and tool should reproduce the reported ordering: mWISE FELLA up to 0.49, mWISE RClass up to 0.39, xMSannotator 0.15–0.22, mummichog 0.08–0.29
- Comparison table (e.g. tanimoto_similarity_comparison.csv) contains aggregated per-tool statistics plus per-peak Tanimoto values for every annotated peak

## Limitations

- mWISE only offers the data to annotate in the KEGG database — an important limitation with respect to xMSannotator, which supports KEGG, HMDB, Lipid Maps, and T3DB
- The repository on GitHub is no longer active; the active repository is hosted on Forgejo at https://dev.b2s.club/b2slab/mWISE, so update remotes and bookmarks
- Evaluation depends on reference peaks having a known original compound identifier and an InChIKey; peaks without these cannot contribute to Tanimoto or TP/FN counts

## Evidence

- [methods] benchmark pipeline configuration: [evidence span withheld — anchor status: not_found]
