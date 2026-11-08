---
name: lcms-feature-annotation-table-generation
description: Use when each sample has a minimal metadata file with its originating
  taxon plus MZmine-processed LC-MS/MS DDA feature MS2 spectra, and you need per-feature
  structural annotations reranked by taxonomy and chemical consistency.
when_to_use_negative:
- The goal is de novo structural elucidation rather than database matching — use Sirius/CSI:FingerID,
  the second annotation method in the ENPKG workflow, instead of ISDB spectral matching.
- Input spectra have not been processed by MZmine (no feature detection, no exported
  MS2 per feature), since the workflow expects feature MS2 spectra from the sample-wise
  folder architecture.
- The samples lack taxonomic metadata for their originating organism, because the
  reranking depends on taxonomic distance via Open Tree of Life/Wikidata IDs.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: matchms
  role: Python package (v0.20.0) used for both spectral matching against ISDB-LOTUS
    and molecular networking (modified cosine) of the feature spectra
- name: MZmine
  role: Upstream feature detection and MS2 export step producing the sample-wise files
    that feed the annotation
- name: ISDB-LOTUS
  role: In silico natural products structural database that candidate structures are
    matched against
- name: Open Tree of Life
  role: Taxonomy reference (ottID) used to resolve the sample species and compute
    taxonomic distance for candidate reranking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:31a7eaf9aa2fbc4e66e1e221e131f1224fb94745236b14756ff2b58004077c35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ISDB-LOTUS
  - MZmine
  - Open Tree of Life
  - matchms
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lcms-feature-annotation-table-generation

## Summary

Generates a reranked candidate-annotation table for each LC-MS2 feature by spectral matching MZmine-exported MS2 spectra against the ISDB-LOTUS in silico natural products database with matchms, then reweighting candidates by taxonomic distance (Open Tree of Life IDs) and chemical-consistency propagation through the modified-cosine molecular network.

## When to use

Use when each sample has a minimal metadata file with its originating taxon plus MZmine-processed LC-MS/MS DDA feature MS2 spectra, and you need per-feature structural annotations reranked by taxonomy and chemical consistency.

## When NOT to use

- The goal is de novo structural elucidation rather than database matching — use Sirius/CSI:FingerID, the second annotation method in the ENPKG workflow, instead of ISDB spectral matching.
- Input spectra have not been processed by MZmine (no feature detection, no exported MS2 per feature), since the workflow expects feature MS2 spectra from the sample-wise folder architecture.
- The samples lack taxonomic metadata for their originating organism, because the reranking depends on taxonomic distance via Open Tree of Life/Wikidata IDs.

## Inputs

- MZmine-exported feature MS2 spectra per sample (sample-wise directory architecture)
- Minimal metadata file containing the sample's originating taxon
- ISDB-LOTUS in silico natural products database

## Outputs

- Reranked candidate-annotation table with one annotated candidate selected per feature

## How to apply

Load the MZmine-exported feature MS2 spectra for each sample from its sample-wise directory and run spectral matching in Python with matchms (v0.20.0) against ISDB-LOTUS, using an m/z tolerance of 0.01 for parent mass and fragments, a minimal cosine score of 0.2, and at least 6 matching fragments; also consider potential adducts of compounds reported in the sample's species for MS1-only annotation at 10 ppm tolerance. Resolve the sample's organism to Open Tree of Life/Wikidata IDs and rerank matching candidates by the taxonomic distance between the producing organism of each candidate structure and the sample's organism. Reweight the candidates by the chemical consistency of the molecular network cluster (matchms molecular networking with a modified cosine score cutoff of 0.7, max_links 10, top_n 15) following the Network Annotation Propagation principle. Finally, select the best-ranked candidate for each feature and write the reranked candidate-annotation table.

## Related tools

- **matchms** (Python package (v0.20.0) used for both spectral matching against ISDB-LOTUS and molecular networking (modified cosine) of the feature spectra)
- **MZmine** (Upstream feature detection and MS2 export step producing the sample-wise files that feed the annotation)
- **ISDB-LOTUS** (In silico natural products structural database that candidate structures are matched against)
- **Open Tree of Life** (Taxonomy reference (ottID) used to resolve the sample species and compute taxonomic distance for candidate reranking)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git
```

## Evaluation signals

- The reranked candidate-annotation table is produced with one best-ranked annotated candidate per feature.
- All features retained after the feature-filtering module carry an associated MS2 scan, so every row of the annotation table has a spectrum to match.
- Spectral matching parameters are verifiably set: m/z tolerance 0.01, minimal cosine score 0.2, minimum 6 matched fragments; MS1-only adduct annotation uses 10 ppm.
- Molecular networking parameters are verifiably set: modified cosine score cutoff 0.7, max_links 10, top_n 15.
- Taxonomic reranking is grounded in resolved ottID/Wikidata IDs for both the sample organism and the candidate producing organism.

## Limitations

- Feature alignment steps based on RT in current workflows hinder comparison of samples from different batches or instrumental setups, which motivates the sample-centric processing.
- Annotations from spectral matching to in silico databases are approximate; the workflow proposes comparing structural annotations from two different approaches (ISDB taxo and Sirius/CSI:FingerID) to strengthen confidence in spectral annotation.
- Workflow requires input datasets formatted in a specific way (e.g. the sample-wise folder architecture and minimal metadata file); the emikg README notes it will likely not work if the dataset is not formatted exactly like the example.

## Evidence

- [readme] spectral matching with taxonomic reranking: [evidence span withheld — anchor status: external_doc]
- [readme] workflow inputs: [evidence span withheld — anchor status: external_doc]
- [readme] taxonomy resolution: [evidence span withheld — anchor status: external_doc]
- [methods] MN parameters: "The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo."
- [methods] taxo enhancer scripts: "The scripts used for this taxonomic resolution are available in https://github.com/enpkg/enpkg_full/tree/main/02_enpkg_taxo_enhancer."
- [results] annotation confidence: [evidence span withheld — anchor status: not_found]
- [full_text] source card: matching parameters: [evidence span withheld — anchor status: not_found]
