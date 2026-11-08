---
name: rdf-knowledge-graph-construction
description: Use when per-sample LC-MS/MS DDA metabolomics results (feature tables,
  MS2 spectra, taxonomic metadata) must be integrated across datasets without RT-based
  feature alignment, e.g. merging batches acquired under different conditions into
  one queryable graph.
when_to_use_negative:
- Input data is a single already-aligned feature table (N x M matrix) from a homogeneous
  batch — classical FBMN-style analysis suffices without RDF construction
- Samples lack a minimal metadata file with the originating taxon, since taxonomic
  resolution against Open Tree of Life is a mandatory enhancement step
- The dataset is not formatted in the exact same way as the reference example — per
  the emikg README, the conversion scripts 'will likely not work if your dataset to
  be treated is not formatted in the exact same way as the example dataset'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ENPKG
  role: The overall workflow whose 06_enpkg_graph_builder formats per-sample results
    as RDF .ttl triples
  repo: https://github.com/enpkg/enpkg_workflow
- name: RDF
  role: Standard graph model used to structure experimental data and public knowledge
    as linked triples
- name: GraphDB
  role: Triple store used to load .ttl files and serve the SPARQL endpoint at https://enpkg.commons-lab.org/graphdb/
- name: SPARQL
  role: Query language used to retrieve and federate data from the knowledge graph
    and external RDF databases
- name: MassIVE
  role: Repository from which external datasets (e.g. MSV000087728, MSV00008616) are
    retrieved before graph integration
- name: MZmine
  role: Performs per-sample feature detection whose exported feature lists and MS2
    scans are the primary graph nodes
- name: emikg
  role: Python package integrating parts of the enpkg_full pipeline to convert metabolomics
    datasets to Linked Open Data
  repo: https://github.com/earth-metabolome-initiative/emikg
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:3e3657aa296e5545d042319b163caa818e748ad42165f3dce014ddd38c20626d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ENPKG
  - GraphDB
  - MZmine
  - MassIVE
  - RDF
  - SPARQL
  - emikg
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# rdf-knowledge-graph-construction

## Summary

Builds an RDF knowledge graph (ENPKG) from individually processed LC-HRMS2 metabolomics samples, integrating MZmine features, ISDB-LOTUS/SIRIUS annotations, and Open Tree of Life taxonomy as .ttl triples loadable in GraphDB. Replaces classical RT-based feature alignment with sample-centric, semantically queryable integration.

## When to use

Use when per-sample LC-MS/MS DDA metabolomics results (feature tables, MS2 spectra, taxonomic metadata) must be integrated across datasets without RT-based feature alignment, e.g. merging batches acquired under different conditions into one queryable graph.

## When NOT to use

- Input data is a single already-aligned feature table (N x M matrix) from a homogeneous batch — classical FBMN-style analysis suffices without RDF construction
- Samples lack a minimal metadata file with the originating taxon, since taxonomic resolution against Open Tree of Life is a mandatory enhancement step
- The dataset is not formatted in the exact same way as the reference example — per the emikg README, the conversion scripts 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset'

## Inputs

- LC-MS/MS DDA data (.mzML) and deconvoluted feature spectra (.mgf) per sample
- Feature quantification tables (m/z, retention time, area/intensity) from MZmine
- Minimal per-sample metadata file containing the sample's originating taxon
- Per-sample SIRIUS/CSI:FingerID/CANOPUS and ISDB-LOTUS spectral-match annotation results
- Taxonomic resolutions (ottID from Open Tree of Life v3.5, Wikidata IDs)

## Outputs

- RDF .ttl files per sample following the ENPKG vocabulary
- A single knowledge graph loadable in GraphDB (reference graph contains over 161 million statements; 788,623 PI and 364,967 NI features)
- SPARQL-queryable integration supporting federated queries and cross-dataset spectral/structural analog retrieval

## How to apply

Clone enpkg/enpkg_full, configure params/user.yml with the dataset record, and process each sample individually: organize data sample-wise (01_enpkg_data_organization), resolve producing-organism binomial names against Open Tree of Life v3.5 and Wikidata (02_enpkg_taxo_enhancer), run MZmine feature detection and MN/ISDB-LOTUS/SIRIUS annotation (03/04), then format all per-sample results as RDF with 06_enpkg_graph_builder. Load the generated .ttl files into GraphDB and query via SPARQL, optionally federated over multiple RDF endpoints. Features link through properties like enpkg:has_lcms_feature_list_<hash> and annotated compounds receive Wikidata IDs, enabling cross-dataset analog matching (e.g. Tanimoto >= 0.8 structural analogs) that bypasses retention-time alignment entirely.

## Related tools

- **ENPKG** (The overall workflow whose 06_enpkg_graph_builder formats per-sample results as RDF .ttl triples) — https://github.com/enpkg/enpkg_workflow
- **RDF** (Standard graph model used to structure experimental data and public knowledge as linked triples)
- **GraphDB** (Triple store used to load .ttl files and serve the SPARQL endpoint at https://enpkg.commons-lab.org/graphdb/)
- **SPARQL** (Query language used to retrieve and federate data from the knowledge graph and external RDF databases)
- **MassIVE** (Repository from which external datasets (e.g. MSV000087728, MSV00008616) are retrieved before graph integration)
- **MZmine** (Performs per-sample feature detection whose exported feature lists and MS2 scans are the primary graph nodes)
- **emikg** (Python package integrating parts of the enpkg_full pipeline to convert metabolomics datasets to Linked Open Data) — https://github.com/earth-metabolome-initiative/emikg

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git; cd enpkg_full; sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Regenerated graph reproduces reported scale: 788,623 features in PI and 364,967 in NI modes, and over 161 million total statements
- Integrated triples are retrievable at the ENPKG SPARQL endpoint (https://enpkg.commons-lab.org/graphdb/); e.g. the Korean Pharmacopeia PI-mode feature list resolves under enpkg:has_lcms_feature_list_a137fd4a263d3587d35f61a526932c09
- SPARQL queries retrieve cross-dataset matches, e.g. scopolamine analogs shared between Datura metel and Scopolia japonica despite different acquisition conditions
- Every annotated feature carries resolved external identifiers (ottID, Wikidata QID) verifiable against Open Tree of Life v3.5 and Wikidata
- Generated .ttl files validate against the ENPKG vocabulary documented at https://enpkg.commons-lab.org/doc/index.html

## Limitations

- Datasets must be organized in a sample-wise folder architecture matching the expected schema; the emikg README warns the scripts 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset'
- No reported rate of false positives/false negatives from semantic merging of experimental data with public resources
- No benchmark against existing workflows or baselines for exploring chemodiversity is provided

## Evidence

- [intro] RDF as standard graph model: "A format used to build a KG is the resource description framework (RDF), a standard graph model for structured and semistructured data interchange on the Web."
- [intro] sample-centric processing and graph integration: "Each sample is considered individually for taxonomic metadata standardization, feature detection, structural annotation, and FBMN."
- [results] graph scale validation: "788,623 and 364,967 features in positive ionization (PI) and negative ionization (NI) modes, respectively"
- [readme] GraphDB exploration instructions: "You can use GraphDB to explore the generated Knowledge Graph. To do so, you will need to install GraphDB (https://graphdb.ontotext.com/download/) and import the generated .ttl files."
- [readme] integrated Korean Pharmacopeia dataset: "This KG also integrates data from a metabolomics study led over 337 medicinal plants of the Korean Pharmacopeia"
- [readme] Wikidata-connected graph purpose: [evidence span withheld — anchor status: external_doc]
