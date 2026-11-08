---
name: ms2-spectral-similarity-scoring
description: Use when you have MS1 features with MS2 spectra and a reaction-linked
  feature-pair table, and must filter network edges by spectral similarity before
  metabolite annotation propagation.
when_to_use_negative:
- 'Input lacks MS2 spectra (MS1-only features): similarity cannot be scored and only
  MS1 matching plus reaction mapping constraints apply.'
- Goal is library spectral matching against reference standards (e.g., GNPS library
  search) rather than filtering edges in a reaction-mapped feature network.
- Feature pairs are not pre-constrained by reaction relationships; scoring all-vs-all
  feature pairs would not reproduce the data-constrained MRN.
edam_operation: http://edamontology.org/operation_0368
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetDNA3
  role: Implements the pre-mapping pipeline whose MS2 similarity constraint this scoring
    refines; available at http://metdna.zhulab.cn/
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: SpectraTools
  role: Provides getMatchParam and spectral similarity computation for the modified
    dot-product score between reaction-linked feature pairs
  repo: https://github.com/ZhuMetLab/MetDNA2
- name: igraph
  role: Handles the network structures (feature network and MRN) over which similarity-filtered
    edges are applied
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:a26637fc1ce656b092c1b706101132c8c0de31c99b9a6efd815f44cf5de4c62c
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
  - MetDNA3
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_0368
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# MS2 spectral similarity scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a modified dot-product MS2 spectral similarity score between reaction-linked feature pairs and applies a ≥ 0.5 cutoff to convert the knowledge-layer metabolic reaction network into a data-constrained feature network (MRN) in MetDNA3's pre-mapping stage.

## When to use

Use when you have MS1 features with MS2 spectra and a reaction-linked feature-pair table, and must filter network edges by spectral similarity before metabolite annotation propagation.

## When NOT to use

- Input lacks MS2 spectra (MS1-only features): similarity cannot be scored and only MS1 matching plus reaction mapping constraints apply.
- Goal is library spectral matching against reference standards (e.g., GNPS library search) rather than filtering edges in a reaction-mapped feature network.
- Feature pairs are not pre-constrained by reaction relationships; scoring all-vs-all feature pairs would not reproduce the data-constrained MRN.

## Inputs

- MS1 feature table (e.g., NIST urine HILIC-MS(+) feature table loaded with getMs1Data4Mrn3)
- MS2 spectra (loaded with getMs2Data4Mrn3)
- Reaction-linked feature-pair table from the GNN-curated knowledge-layer MRN (getMrnNeighbor)

## Outputs

- Filtered pair table of reaction-linked features with modified dot-product similarity ≥ 0.5 (data-constrained feature network)
- Edge table exported as data_constrained_mrn_edges.csv
- table_ms2_network_filter input for getMetAnno

## How to apply

First retrieve the reaction-linked feature pairs from the GNN-curated knowledge-layer MRN (e.g., via getMrnNeighbor), which constrains which feature pairs are compared. Then compute the modified dot-product MS2 spectral similarity for each pair using getMatchParam from the SpectraTools package, with parameters such as mz_tol_ms2, mz_ppm_thr, includePrecursor, ppmPrecursorFilter, thrIntensityAbs, thrIntensityRel, and methodScore controlling the fragment matching. Retain only edges with similarity ≥ 0.5, forming the data-constrained feature network — the second pre-mapping constraint after MS1 matching and reaction relationship mapping. Finally, pass the filtered pair table to getMetAnno as table_ms2_network_filter so the two-layer topology can propagate seed annotations (e.g., >1,600 seed metabolites with chemical standards) across the refined network.

## Related tools

- **MetDNA3** (Implements the pre-mapping pipeline whose MS2 similarity constraint this scoring refines; available at http://metdna.zhulab.cn/) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **SpectraTools** (Provides getMatchParam and spectral similarity computation for the modified dot-product score between reaction-linked feature pairs) — https://github.com/ZhuMetLab/MetDNA2
- **igraph** (Handles the network structures (feature network and MRN) over which similarity-filtered edges are applied) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
params <- getMatchParam(mz_tol_ms2, mz_ppm_thr, includePrecursor, ppmPrecursorFilter, thrIntensityAbs, thrIntensityRel, methodScore); filtered_edges <- pairs[pairs$ms2_similarity >= 0.5, ]
```

## Evaluation signals

- For NIST urine HILIC-MS(+), the resulting data-constrained MRN must contain exactly 2,993 metabolites and 55,674 reaction pairs, matching the Source Data file for Fig. 2c–d.
- Every retained edge must carry a modified dot-product similarity score ≥ 0.5; edges below the cutoff are removed.
- Retained edges must be a subset of the reaction-linked feature pairs returned by getMrnNeighbor (no edges absent from the knowledge-layer pair list).
- The exported data_constrained_mrn_edges.csv must be structurally compatible as table_ms2_network_filter for getMetAnno (correct columns for pair-wise similarity feeding annotation rounds).

## Limitations

- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so similarity-based filtering may retain indistinguishable isomers.
- The GNN-predicted reaction pairs feeding the network may include a considerable number of false positives, and no FDR is reported for the curated MRN because non-RPs (negative examples) are not clearly defined.
- Similarity scores are computed only for reaction-linked pairs, so the threshold's effect depends on the quality of the upstream knowledge-layer curation.

## Evidence
