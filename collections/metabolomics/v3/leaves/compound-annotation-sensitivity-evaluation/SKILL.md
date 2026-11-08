---
name: compound-annotation-sensitivity-evaluation
description: Use when you have per-peak top-N compound annotations from an LC-MS annotation
  pipeline (e.g. mWISE top three candidates) plus a reference set of true compounds
  per assay, and you need entity-level sensitivity/F1/Tanimoto metrics to compare
  configurations or algorithms.
when_to_use_negative:
- You only have peak-to-compound mass matching output (mass-to-charge to KEGG) and
  no top-N prioritized proposals with which to compute TP/FP/FN.
- Your reference set lacks compound chemical structures, so the Tanimoto ratio cannot
  be computed and only count-based metrics are meaningful.
- You need database-coverage comparison beyond KEGG (e.g. HMDB, Lipid Maps, T3DB),
  since mWISE only annotates against KEGG and the metric reflects that limitation.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: mWISE
  role: Source of the top-N per-peak candidate annotations being evaluated; provides
    the FELLA or RClass diffusion prioritization outputs the metrics are computed
    over.
  repo: github.com/b2slab/mWISE
- name: FELLA
  role: Supplies the biological network used in the evaluated diffusion configuration
    (raw score, binary input), whose entity-level sensitivity is being measured.
- name: Diffu Stats
  role: R package used by mWISE to compute the diffusion scores underlying the ranked
    candidate lists that the sensitivity evaluation consumes.
- name: R
  role: Implementation environment in which the metric recomputation script and mWISE
    pipeline run.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:07e20928a4237e0b25de92c8f4510bd52401038b4721b1f3e8040241d70cc92d
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
  - R
  - mWISE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# compound-annotation-sensitivity-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates the entity-level sensitivity (and F1, specificity, precision, Tanimoto ratio) of an LC-MS compound-annotation algorithm such as mWISE by recomputing TP/FP/FN counts from top-N candidate annotations against reference compound sets, as demonstrated for mWISE's FELLA-network + raw-score diffusion configuration.

## When to use

Use when you have per-peak top-N compound annotations from an LC-MS annotation pipeline (e.g. mWISE top three candidates) plus a reference set of true compounds per assay, and you need entity-level sensitivity/F1/Tanimoto metrics to compare configurations or algorithms.

## When NOT to use

- You only have peak-to-compound mass matching output (mass-to-charge to KEGG) and no top-N prioritized proposals with which to compute TP/FP/FN.
- Your reference set lacks compound chemical structures, so the Tanimoto ratio cannot be computed and only count-based metrics are meaningful.
- You need database-coverage comparison beyond KEGG (e.g. HMDB, Lipid Maps, T3DB), since mWISE only annotates against KEGG and the metric reflects that limitation.

## Inputs

- Per-assay top-N candidate annotation lists from the mWISE pipeline (top three candidates per peak, chosen diffusion configuration e.g. FELLA network + raw score)
- Reference compound sets per assay (true annotated compounds, with reference-level false positives)
- Chemical structures (e.g. SMILES/InChI) of proposed and original compounds for Tanimoto ratio computation
- Candidate/background compound counts per assay for TN estimation

## Outputs

- Per-assay entity-metric table: TP, TN, FP, FN, reference FP, number of references, sensitivity, specificity, precision, NPV, F1
- Per-assay Tanimoto ratio between structures of proposed and original compounds
- Cross-assay aggregate summary (e.g. sensitivity mean and SD across datasets, comparable to mWISE 0.63 (0.07) vs xMSannotator 0.55 (0.19))

## How to apply

Load the top-N (mWISE default: top three candidates per peak) annotation proposals for each assay and the reference compound lists, then derive per-assay TP, FP, FN, TN, reference FP and reference counts (e.g. the FELLA + raw score Assay 6 case: 53 TP, 90 FP, 7852 TN, 21 FN, 1 reference FP, 74 references). From these counts compute sensitivity, specificity, precision, NPV and F1 per assay, and additionally compute the Tanimoto structural-similarity ratio between proposed and original compounds to capture chemically close false positives (e.g. Assay 1 ratio 0.69; Assay 3 ratio 0.61). Aggregate all assays into a single table mirroring the article's columns (TP, TN, FP, FN, ref FP, references, sensitivity, specificity, precision, NPV, F1, Tanimoto ratio) so configurations (FELLA vs RClass networks, raw score vs z-normalized scores) or algorithms (mWISE vs xMSannotator vs mummichog) can be compared. Validate by checking that recomputed per-assay F1 and sensitivity match the reported values (e.g. Assay 6 F1 = 0.49, sensitivity = 0.72 under FELLA + raw score) and flagging any deviating row.

## Related tools

- **mWISE** (Source of the top-N per-peak candidate annotations being evaluated; provides the FELLA or RClass diffusion prioritization outputs the metrics are computed over.) — github.com/b2slab/mWISE
- **FELLA** (Supplies the biological network used in the evaluated diffusion configuration (raw score, binary input), whose entity-level sensitivity is being measured.)
- **Diffu Stats** (R package used by mWISE to compute the diffusion scores underlying the ranked candidate lists that the sensitivity evaluation consumes.)
- **R** (Implementation environment in which the metric recomputation script and mWISE pipeline run.)

## Examples

```
# R: recompute entity metrics from an mWISE FELLA+raw-score annotation table
metrics <- cbind(TP=53, TN=7852, FP=90, FN=21); sens <- 53/(53+21); f1 <- 2*sens*53/(sens*(53+21)+53)
```

## Evaluation signals

- Recomputed per-assay F1 matches the article's reported value for the chosen configuration (e.g. FELLA + raw score Assay 6 F1 = 0.49; mWISE cluster-filter case Assay 2 specificity = 0.91).
- Recomputed per-assay sensitivity matches the reported value (e.g. FELLA + raw score Assay 6 sensitivity = 0.72; Assay 1 matching sensitivity = 0.92 with 144 TP and 13 FN).
- TP + FN equals the total number of reference compounds per assay (e.g. Assay 1: 144 + 13 = 157 references; Assay 6: 53 + 21 = 74).
- Cross-assay sensitivity mean and SD are in the expected range for the configuration (e.g. mWISE FELLA + raw score: mean 0.63, SD 0.07, versus xMSannotator 0.55, SD 0.19).
- Tanimoto ratios fall in the plausible 0–1 range and are consistent with the article (e.g. Assay 1: 0.69; Assay 3: 0.61), indicating that many false positives are structurally close true-like compounds.

## Limitations

- mWISE only annotates against the KEGG database, so the metrics reflect KEGG-only coverage and cannot assess performance on HMDB, Lipid Maps or T3DB entries — an acknowledged limitation relative to xMSannotator.
- The Tanimoto ratio partially credits false positives that are structurally close to the true compound, so sensitivity alone overstates entity-level agreement; both must be read together.
- Metric values are configuration-dependent: the same algorithm with RClass network and raw score yields much lower F1 (best 0.39, Assay 6) than with FELLA (best 0.49, Assay 6), so comparisons must fix the diffusion configuration.

## Evidence
