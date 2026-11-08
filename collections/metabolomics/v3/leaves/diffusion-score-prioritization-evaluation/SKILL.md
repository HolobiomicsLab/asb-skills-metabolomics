---
name: diffusion-score-prioritization-evaluation
description: Use when LC-MS feature annotation has been produced with a diffusion-prioritization
  setup (e.g. mWISE over FELLA or RClass networks) and you need entity-level sensitivity/F1
  metrics and structure-based Tanimoto comparisons to judge prioritization quality.
when_to_use_negative:
- When the annotation candidates have not yet been generated — this skill evaluates
  a completed prioritized list, not the matching/clustering/diffusion pipeline itself.
- When the database used is not KEGG (mWISE only offers annotation data for the KEGG
  database), since the candidate/reference mapping would not be comparable.
- When evaluating the cluster-based filter in isolation (Assay 2 FP reduced from 17598
  to 1671, specificity 0.91) rather than the final diffusion prioritization.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: mWISE
  role: produces the top-three candidates per peak for the evaluated diffusion configuration;
    evaluation metrics are computed against these candidates
  repo: https://github.com/b2slab/mWISE
- name: Diffu Stats
  role: generates the diffusion score vector that is then evaluated in terms of prioritization
    performance
  repo: https://github.com/b2slab/mWISE
- name: FELLA
  role: provides the biological network over which diffusion scores are computed and
    then evaluated (e.g. FELLA + raw score reaches F1 0.49 for Assay 6)
  repo: https://github.com/b2slab/mWISE
- name: xMSannotator
  role: baseline annotation tool whose per-assay F1 (0.15-0.22) is compared against
    mWISE diffusion results
  repo: https://github.com/b2slab/mWISE
- name: mummichog
  role: alternative pathway-based baseline with per-assay F1 0.08-0.29 used for benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:4f51083a34d9d1ca7ad0bbd84c9b946070a7fe88a9802ac5f6bcfef441b5aca0
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
  - R
  - mWISE
  - mummichog
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# diffusion-score-prioritization-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate mWISE-style diffusion-based annotation by reproducing per-assay entity metrics (TP, FP, FN, TN, sensitivity, specificity, precision, NPV, F1) for a given diffusion configuration (e.g. FELLA network with raw scores) and complementing them with Tanimoto structure-similarity ratios that quantify how close proposed candidates are to the original compounds.

## When to use

Use when LC-MS feature annotation has been produced with a diffusion-prioritization setup (e.g. mWISE over FELLA or RClass networks) and you need entity-level sensitivity/F1 metrics and structure-based Tanimoto comparisons to judge prioritization quality.

## When NOT to use

- When the annotation candidates have not yet been generated — this skill evaluates a completed prioritized list, not the matching/clustering/diffusion pipeline itself.
- When the database used is not KEGG (mWISE only offers annotation data for the KEGG database), since the candidate/reference mapping would not be comparable.
- When evaluating the cluster-based filter in isolation (Assay 2 FP reduced from 17598 to 1671, specificity 0.91) rather than the final diffusion prioritization.

## Inputs

- Per-assay mWISE top-three candidate annotations per LC-MS peak for a given diffusion configuration (network + input type + score type, e.g. FELLA with binary input, raw score, unique annotation)
- Reference compound sets per assay defining true positives, false positives and false negatives
- Chemical structures (e.g. SMILES/InChI) for proposed and original compounds to compute Tanimoto similarity

## Outputs

- Per-assay table with TP, TN, FP, FN, reference FP, references, sensitivity, specificity, precision, NPV, F1 (e.g. Assay 6: 53, 7852, 90, 21, 1, 74, 0.72, 0.99, 0.37, 0.99, 0.49)
- Per-assay Tanimoto ratio between chemical structures of proposed and original compounds
- Cross-assay comparison against alternative configurations (RClass, xMSannotator, mummichog) for benchmarking

## How to apply

Load the mWISE top-three candidates per peak from the chosen diffusion configuration (e.g. FELLA network with binary input, raw score, unique annotation option), plus the reference compound sets per assay, and derive TP, FP, FN and TN counts per assay (e.g. Assay 6 under FELLA + raw score: 53 TP, 90 FP, 7852 TN, 21 FN against 74 references). From these counts compute sensitivity, specificity, precision, NPV and F1, and check that the recomputed values match the article's reported tables (Assay 6: sensitivity 0.72, F1 0.49). Additionally compute the Tanimoto ratio between chemical structures of proposed and original compounds — mWISE's FELLA + raw score configuration gave Tanimoto ratios of 0.69 (Assay 1) and 0.61 (Assay 3) — since many false positives are structurally close to the true compounds. Finally, compare against alternative configurations (RClass network: Assay 6 F1 0.39) and baselines (xMSannotator F1 up to 0.22; mummichog up to 0.29) to position the diffusion configuration's performance.

## Related tools

- **mWISE** (produces the top-three candidates per peak for the evaluated diffusion configuration; evaluation metrics are computed against these candidates) — https://github.com/b2slab/mWISE
- **Diffu Stats** (generates the diffusion score vector that is then evaluated in terms of prioritization performance) — https://github.com/b2slab/mWISE
- **FELLA** (provides the biological network over which diffusion scores are computed and then evaluated (e.g. FELLA + raw score reaches F1 0.49 for Assay 6)) — https://github.com/b2slab/mWISE
- **xMSannotator** (baseline annotation tool whose per-assay F1 (0.15-0.22) is compared against mWISE diffusion results) — https://github.com/b2slab/mWISE
- **mummichog** (alternative pathway-based baseline with per-assay F1 0.08-0.29 used for benchmarking)

## Examples

```
Rscript -e 'mWISE::run_metrics(assay="Assay6", config="FELLA_raw", candidates=readRDS("mwise_fella_raw.rds"), refs=readRDS("refs_assay6.rds"))'
```

## Evaluation signals

- Recomputed F1 and sensitivity per assay must match the article's reported values for the corresponding configuration (e.g. FELLA + raw score, Assay 6: F1 0.49, sensitivity 0.72); flag any row that deviates.
- TP + FP + TN + FN must equal the total number of candidate peaks evaluated per assay (e.g. Assay 6: 53 + 7852 + 90 + 21 totals correctly against 74 references).
- Tanimoto ratio should be reproduced within tolerance (Assay 1: 0.69; Assay 3: 0.61 for FELLA + raw score).
- Cross-configuration ordering should hold: all four mWISE configurations outperform xMSannotator in performance and computation time.
- Reference count per assay must match the reported reference columns (e.g. Assay 6: 74 references, 1 reference FP).

## Limitations

- mWISE only offers annotation data for the KEGG database, so entity evaluation against other databases (HMDB, Lipid Maps, T3DB) is not directly possible.
- Diffusion prioritization performance varies considerably by network choice (FELLA F1 0.49 vs RClass 0.39 for Assay 6), so results are configuration-specific.
- Many false positives are structurally close to the true compounds — high Tanimoto ratio does not guarantee annotation correctness, but indicates prioritization is chemically reasonable.

## Evidence
