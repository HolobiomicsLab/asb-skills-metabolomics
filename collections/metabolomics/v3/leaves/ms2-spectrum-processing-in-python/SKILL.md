---
name: ms2-spectrum-processing-in-python
description: Use when each sample's LC-MS/MS DDA features (from MZmine feature detection)
  need structural annotation in Python via spectral matching to a large in silico
  MS2 database with organism-level reweighting.
when_to_use_negative:
- Input is not MZmine-processed LC-MS/MS DDA features (e.g. already-aligned feature
  tables from a different pipeline) — the sample-centric workflow expects per-sample
  MZmine exports
- The sample has no resolved taxonomic metadata (no Open Tree of Life ottID / Wikidata
  ID), so taxonomic reweighting cannot be applied
- Only library spectral matching without organism-based reranking or molecular-network
  chemical-consistency propagation is needed — the article's reweighting steps would
  be superfluous
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Python package (v0.20.0) used for MS2 spectral matching against ISDB-LOTUS
    and for matchms molecular networking (modified cosine, cutoff 0.7)
  repo: https://github.com/enpkg/enpkg_workflow
- name: ISDB-LOTUS
  role: In silico database of natural products providing the candidate spectra and
    structure–organism pairs used for matching and taxonomic reweighting
- name: MZmine
  role: Upstream tool that performs feature detection and exports the feature MS2
    spectra consumed by this Python matching step
- name: Open Tree of Life
  role: Taxonomy backbone used to compute taxonomic distance between the candidate's
    producing organism and the sample's organism
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:2da2e1d9c10ab864d04275ee47c4bd793962069694072cdc8cb615ebc8fd90b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ISDB-LOTUS
  - MZmine
  - Open Tree of Life
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# MS2 spectrum processing in Python

## Summary

Process MZmine-exported feature MS2 spectra in Python with the matchms package to perform spectral matching against the ISDB-LOTUS in silico natural products database, followed by taxonomic and chemical-consistency reranking of candidate annotations.

## When to use

Use when each sample's LC-MS/MS DDA features (from MZmine feature detection) need structural annotation in Python via spectral matching to a large in silico MS2 database with organism-level reweighting.

## When NOT to use

- Input is not MZmine-processed LC-MS/MS DDA features (e.g. already-aligned feature tables from a different pipeline) — the sample-centric workflow expects per-sample MZmine exports
- The sample has no resolved taxonomic metadata (no Open Tree of Life ottID / Wikidata ID), so taxonomic reweighting cannot be applied
- Only library spectral matching without organism-based reranking or molecular-network chemical-consistency propagation is needed — the article's reweighting steps would be superfluous

## Inputs

- MZmine-exported feature MS2 spectra for the sample (sample-wise folder architecture)
- Minimal metadata file with the sample's originating taxon (with ottID/Wikidata ID resolved via Open Tree of Life)
- ISDB-LOTUS in silico spectral database
- Molecular network (MN) cluster assignments from matchms molecular networking

## Outputs

- Reranked candidate-annotation table with the best-ranked candidate selected for each feature
- MS1-only annotations for compounds reported in the sample's species (adducts considered, 10 ppm tolerance)

## How to apply

Load the MZmine-exported feature MS2 spectra for the sample from its sample-wise directory, then run spectral matching in Python with matchms (v0.20.0) against the ISDB-LOTUS database using an m/z tolerance of 0.01 for both parent mass and fragment matching, a minimal cosine score of 0.2, and at least 6 matching fragments; also consider potential adducts of compounds reported in the sample's species for MS1-only annotation with a 10 ppm tolerance. Then rerank candidates by taxonomic distance between the candidate's producing organism (resolved via Open Tree of Life/Wikidata IDs) and the sample's organism, and reweight them by chemical consistency of the molecular network cluster (modified cosine score cutoff 0.7, max_links 10, top_n 15) following Network Annotation Propagation. Finally select the best-ranked candidate per feature and write the reranked candidate-annotation table; this is implemented in the scripts under 03_enpkg_mn_isdb_isdb_taxo.

## Related tools

- **matchms** (Python package (v0.20.0) used for MS2 spectral matching against ISDB-LOTUS and for matchms molecular networking (modified cosine, cutoff 0.7)) — https://github.com/enpkg/enpkg_workflow
- **ISDB-LOTUS** (In silico database of natural products providing the candidate spectra and structure–organism pairs used for matching and taxonomic reweighting)
- **MZmine** (Upstream tool that performs feature detection and exports the feature MS2 spectra consumed by this Python matching step)
- **Open Tree of Life** (Taxonomy backbone used to compute taxonomic distance between the candidate's producing organism and the sample's organism)

## Examples

```
uv run python 03_enpkg_mn_isdb_isdb_taxo/03_enpkg_mn_isdb_isdb_taxo.py --params params/user.yml
```

## Evaluation signals

- The reranked candidate-annotation table is produced with one annotated candidate per feature
- Scripts under 03_enpkg_mn_isdb_isdb_taxo run to completion without error
- Matching respects the configured thresholds: m/z tolerance 0.01, minimal cosine score 0.2, and ≥6 matching fragments
- Taxonomic reweighting orders candidates consistently with the Open Tree of Life distance between producing organism and sample taxon
- Chemical-consistency reweighting uses the MN cluster (modified cosine score cutoff 0.7, max_links 10, top_n 15) without cross-cluster leakage

## Limitations

- MS1-only annotations rely on adducts of compounds reported in the sample's species and a 10 ppm tolerance, so confidence depends on the completeness of species-level structure reporting
- Taxonomic reranking requires correct resolution of the sample's organism to an Open Tree of Life/Wikidata ID; unresolved taxa degrade reweighting
- The workflow processes each sample individually and does not perform RT-based feature alignment, so results are not directly comparable across batches without later KG-level alignment

## Evidence

- [methods] spectral matching in Python with the matchms (v0.20.0) package against the ISDB-LOTUS database: [evidence span withheld — anchor status: not_found]
- [methods] m/z tolerance for parent mass and fragment matching of 0.01, a minimal cosine score of 0.2, and 6 minimal matching fragments: [evidence span withheld — anchor status: not_found]
- [methods] rerank the matching candidates according to the taxonomic distance between the producing organism of the candidate structure and the sample's organism: [evidence span withheld — anchor status: not_found]
- [methods] The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo: "The scripts used for Molecular Networking are available at https://github.com/enpkg/enpkg_full/tree/main/03_enpkg_mn_isdb_isdb_taxo."
- [readme] annotates features using two different methods (spectral matching to in silico DB coupled to taxonomical reweighting and Sirius/CSI:FingerID): [evidence span withheld — anchor status: external_doc]
- [results] the data for each sample are processed to perform feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine: "the data for each sample are processed to perform feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine"
