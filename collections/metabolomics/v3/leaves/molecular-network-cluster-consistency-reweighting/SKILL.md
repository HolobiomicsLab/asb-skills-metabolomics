---
name: molecular-network-cluster-consistency-reweighting
description: Use when candidate MS2 annotation lists (e.g., ISDB-LOTUS spectral matches)
  need to be reweighted using the chemical consistency of the molecular-network cluster
  a feature belongs to.
when_to_use_negative:
- Features lack associated MS2 scans or have no molecular-network edges, so no cluster
  consistency signal exists (the ENPKG feature filtering keeps only features with
  an associated MS2 scan)
- The goal is reranking by taxonomic distance of the producing organism rather than
  chemical cluster consistency — that is a separate reranking mechanism in the same
  workflow
- Only a single annotation method is desired without cross-validation, e.g., when
  combining ISDB and SIRIUS/CSI:FingerID annotations for confidence is the actual
  goal
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Computes modified cosine molecular networking edges and is the Python package
    in which the spectral matching and reranking are implemented (v0.20.0 in the paper)
  repo: https://github.com/enpkg/enpkg_workflow
- name: MZmine
  role: Provides the feature detection output (feature MS2 spectra) that form the
    nodes of the molecular network
- name: ISDB-LOTUS
  role: In silico natural products database supplying the candidate structures whose
    annotations get reweighted by cluster consistency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:cdf3c09f6bd33df315f1278462da6af4bc15a2acf62e405a6b3fdf444792b3a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ISDB-LOTUS
  - MZmine
  - Open Tree of Life
  - matchms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# molecular-network-cluster-consistency-reweighting

## Summary

Reranks candidate structural annotations by propagating annotation confidence across a feature-based molecular network: features connected by high modified-cosine edges inherit support from the annotations of their spectral neighbors, following the Network Annotation Propagation principle.

## When to use

Use when candidate MS2 annotation lists (e.g., ISDB-LOTUS spectral matches) need to be reweighted using the chemical consistency of the molecular-network cluster a feature belongs to.

## When NOT to use

- Features lack associated MS2 scans or have no molecular-network edges, so no cluster consistency signal exists (the ENPKG feature filtering keeps only features with an associated MS2 scan)
- The goal is reranking by taxonomic distance of the producing organism rather than chemical cluster consistency — that is a separate reranking mechanism in the same workflow
- Only a single annotation method is desired without cross-validation, e.g., when combining ISDB and SIRIUS/CSI:FingerID annotations for confidence is the actual goal

## Inputs

- MZmine-exported feature MS2 spectra for each sample (sample-wise directory architecture)
- Candidate structure annotations from ISDB-LOTUS spectral matching (cosine score >= 0.2, >= 6 matched fragments, 0.01 m/z tolerance)
- Taxonomic reranking scores resolved via Open Tree of Life/Wikidata IDs

## Outputs

- Reranked candidate-annotation table with one best-ranked candidate per feature
- Molecular network edges (modified cosine scores) for feature clusters

## How to apply

Within the matchms-based annotation step of the ENPKG workflow, first build a molecular network from the MZmine-exported feature MS2 spectra using modified cosine scoring with a cutoff of 0.7, max_links set to 10, and top_n set to 15. Then, following the Network Annotation Propagation principle, reweight each feature's candidate structures according to how chemically consistent the candidate is with the annotations of spectrally connected neighbors in the cluster. Combine this with (or after) taxonomic distance reranking, select the best-ranked candidate per feature, and write the reranked candidate-annotation table. The scripts implementing this are in the enpkg_full 03_enpkg_mn_isdb_isdb_taxo module.

## Related tools

- **matchms** (Computes modified cosine molecular networking edges and is the Python package in which the spectral matching and reranking are implemented (v0.20.0 in the paper)) — https://github.com/enpkg/enpkg_workflow
- **MZmine** (Provides the feature detection output (feature MS2 spectra) that form the nodes of the molecular network)
- **ISDB-LOTUS** (In silico natural products database supplying the candidate structures whose annotations get reweighted by cluster consistency)

## Examples

```
sh workflow/00_workflow_all.sh
```

## Evaluation signals

- The reranked candidate-annotation table is produced with exactly one annotated candidate per feature
- The MN build parameters are respected: modified cosine score cutoff 0.7, max_links 10, top_n 15
- The scripts used (03_enpkg_mn_isdb_isdb_taxo) run to completion without error
- Reranked top candidates show improved chemical consistency within clusters (neighbors in a cluster converge on structurally consistent candidates)
- On the README test dataset (Zenodo 10018590), the full workflow including this step completes in about 10 minutes

## Limitations

- Reranking confidence is only as good as the underlying ISDB-LOTUS spectral matches (minimal cosine 0.2, 6 matched fragments), so weak matches propagate weakly
- The paper reports no validation metric or ground-truth comparison for downstream biological identifications, and no false positive/false negative rate for merging experimental annotations with public resources
- Sample-centric processing means the MN is built per sample from MZmine outputs; no RT-based feature alignment is used, so cross-batch consistency relies on the propagation itself

## Evidence

- [full_text] Reweight candidates by the chemical consistency of the molecular network cluster (matchms molecular networking: modified cosine score cutoff 0.7, max_links 10, top_n 15) following the Network Annotation Propagation principle.: [evidence span withheld — anchor status: not_found]
- [intro] the quantitative, spectral, and structural data can then be integrated and visualized through feature-based molecular networking (FBMN): "the quantitative, spectral, and structural data can then be integrated and visualized through feature-based molecular networking (FBMN)"
- [readme] generates a Molecular Network from fragmentation spectra (MN) and annotates features using two different methods (spectral matching to in silico DB coupled to taxonomical reweighting: [evidence span withheld — anchor status: not_found]
- [methods] The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo.: "The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo."
- [results] Comparing structural annotations results from two different approaches...allows the user to strengthen confidence in spectral annotation: [evidence span withheld — anchor status: not_found]
