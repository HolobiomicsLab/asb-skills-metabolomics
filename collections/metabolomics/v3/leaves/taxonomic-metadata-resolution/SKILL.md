---
name: taxonomic-metadata-resolution
description: Use when processing LC-MS/MS DDA metabolomics samples sample-wise and
  the input metadata only contains free-text organism binomial names that must be
  standardized to stable taxonomic identifiers before RDF/SPARQL integration.
when_to_use_negative:
- Input metadata already carries resolved OTT/Wikidata identifiers, so re-resolution
  would add no value
- Samples have no associated organism metadata (e.g., purely spectral or environmental
  feature lists), since the enhancer resolves producing-organism binomial names
- Data are not LC-MS/MS DDA metabolomics samples processed in the ENPKG sample-wise
  architecture
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ENPKG
  role: Workflow framework whose 02_enpkg_taxo_enhancer scripts perform the per-sample
    taxonomic resolution
  repo: https://github.com/enpkg/enpkg_workflow
- name: enpkg_full
  role: Repository hosting the 02_enpkg_taxo_enhancer scripts invoked between data
    organization (01) and molecular networking/ISDB (03)
  repo: https://github.com/enpkg/enpkg_full
- name: Wikidata
  role: Knowledge base against which organism taxa are linked, enabling connection
    to LOTUS structure–organism pairs in the knowledge graph
- name: Open Tree of Life (v3.5)
  role: Taxonomy reference against which each producing organism's binomial name is
    resolved to an ottID
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:35c8e14f8c76538f3a6da53026140b691b29f665925a632a2d351eec89ae3754
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ENPKG
  - GraphDB
  - Open Tree of Life (v3.5)
  - RDF
  - Wikidata
  - enpkg_full
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# taxonomic-metadata-resolution

## Summary

Resolves the producing-organism binomial names attached to each metabolomics sample against the Open Tree of Life (v3.5) taxonomy (ottID) and Wikidata, using the ENPKG 02_enpkg_taxo_enhancer scripts, so taxonomic metadata becomes semantically linkable in the RDF knowledge graph.

## When to use

Use when processing LC-MS/MS DDA metabolomics samples sample-wise and the input metadata only contains free-text organism binomial names that must be standardized to stable taxonomic identifiers before RDF/SPARQL integration.

## When NOT to use

- Input metadata already carries resolved OTT/Wikidata identifiers, so re-resolution would add no value
- Samples have no associated organism metadata (e.g., purely spectral or environmental feature lists), since the enhancer resolves producing-organism binomial names
- Data are not LC-MS/MS DDA metabolomics samples processed in the ENPKG sample-wise architecture

## Inputs

- Sample-wise minimal metadata files containing the sample's originating taxon (binomial organism name)
- Sample-wise folder architecture produced by 01_enpkg_data_organization (mzML, mgf, feature quantification tables, metadata)

## Outputs

- Taxonomically standardized per-sample records carrying Open Tree of Life ottIDs and Wikidata identifiers
- Enriched per-sample data ready for taxonomic-distance reranking (ISDB-LOTUS) and RDF graph building (06_enpkg_graph_builder)

## How to apply

Run this step per sample, right after the 01_enpkg_data_organization step has arranged the .mzML/.mgf files, feature quantification tables and metadata in a sample-wise folder architecture. Each sample's minimal metadata file supplies the originating taxon; the 02_enpkg_taxo_enhancer scripts resolve that binomial name against the Open Tree of Life taxonomy v3.5 to obtain an ottID and against Wikidata, grounding the organism in identifiers shared with the LOTUS structure–organism pairs. The resolved taxonomy feeds downstream steps (ISDB-LOTUS spectral matching with taxonomic distance reranking in 03_enpkg_mn_isdb_isdb_taxo and the 06_enpkg_graph_builder RDF generation), which is the rationale for standardizing it early. The same resolution was applied to the 337 Korean Pharmacopoeia plant extracts (MSV00008616/MSV000093464) so their producing organisms were linkable with the 1,600-plant-extract graph under a single SPARQL endpoint.

## Related tools

- **ENPKG** (Workflow framework whose 02_enpkg_taxo_enhancer scripts perform the per-sample taxonomic resolution) — https://github.com/enpkg/enpkg_workflow
- **enpkg_full** (Repository hosting the 02_enpkg_taxo_enhancer scripts invoked between data organization (01) and molecular networking/ISDB (03)) — https://github.com/enpkg/enpkg_full
- **Wikidata** (Knowledge base against which organism taxa are linked, enabling connection to LOTUS structure–organism pairs in the knowledge graph)
- **Open Tree of Life (v3.5)** (Taxonomy reference against which each producing organism's binomial name is resolved to an ottID)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git; edit params/user.yml with the MassIVE/Zenodo record_id and record_name; sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Every sample's originating taxon resolves to a non-empty OTT ottID (v3.5) and Wikidata identifier with no unmatched binomial names left in the metadata
- Resolved taxa are present as ontology-linked metadata in the generated .ttl files loaded into GraphDB, retrievable via SPARQL at the ENPKG endpoint
- Cross-dataset SPARQL queries over the integrated graph (1,600 extracts + 337 Korean Pharmacopoeia extracts) can group features by resolved producing organism, e.g., scopolamine analogs in Datura metel and Scopolia japonica
- Taxonomic distance reranking in the ISDB-LOTUS step runs on all samples, confirming the resolved taxonomy propagated to 03_enpkg_mn_isdb_isdb_taxo

## Limitations

- Resolution depends on correct binomial names in the minimal metadata file; misspelled or synonymized taxa may fail or misresolve against Open Tree of Life v3.5
- Taxonomic reranking of spectral candidates depends on this resolution, so annotation confidence propagates any upstream taxonomic errors
- The workflow is designed for LC-MS/MS DDA metabolomics with organism metadata; datasets lacking taxon information cannot use this step

## Evidence

- [readme] the workflow automatically resolves the species taxonomy against Open Tree of Life (ott3.5) (ottID): [evidence span withheld — anchor status: external_doc]
- [readme] minimal metadata file containing the sample's originating taxon: "For each sample, the required input data are 
1) A minimal metadata file containing the sample's originating taxon."
- [methods] scripts used for this taxonomic resolution are available in 02_enpkg_taxo_enhancer: "The scripts used for this taxonomic resolution are available in https://github.com/enpkg/enpkg_full/tree/main/02_enpkg_taxo_enhancer."
- [intro] Each sample is considered individually for taxonomic metadata standardization, feature detection, structural annotation, and FBMN: "Each sample is considered individually for taxonomic metadata standardization, feature detection, structural annotation, and FBMN."
