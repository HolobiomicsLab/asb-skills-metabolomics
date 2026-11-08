---
name: sample-wise-metabolomics-data-organization
description: Use when ingesting an externally published LC-MS2 dataset (e.g., MSV000093464
  from MassIVE) whose .mzML, .mgf, feature quantification tables, and metadata must
  be organized per sample before ENPKG graph building.
when_to_use_negative:
- The dataset is already a single consolidated feature table or aligned feature matrix
  across samples — the sample-centric organization would be redundant and the sample-level
  metadata mapping unavailable.
- The input is LC-MS data lacking per-sample taxonomic metadata (originating taxon),
  since the downstream Open Tree of Life (ott3.5) taxonomic enhancer cannot resolve
  sample identities.
- The goal is classical RT/m/z feature alignment across a single homogeneous batch
  — use a dataset-centric metabolomics workflow instead of reorganizing into a knowledge
  graph structure.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ENPKG data-organization scripts (01_enpkg_data_organization)
  role: Scripts that place exported feature quantification tables, MS2 spectra, and
    metadata into the sample-wise folder architecture consumed by the ENPKG workflow.
  repo: https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization
- name: MassIVE
  role: Public repository from which the LC-MS2 (.mzML) and deconvoluted feature spectra
    (.mgf) files of the external dataset (e.g., MSV000093464 / MSV00008616) are retrieved.
- name: MZmine
  role: Tool that produces the per-sample feature detection outputs (quantification
    tables and deconvoluted .mgf spectra) that this organization step packages.
  repo: http://mzmine.github.io/
- name: Open Tree of Life (ott3.5) taxo enhancer
  role: Downstream step (02_enpkg_taxo_enhancer) that resolves the producing-organism
    names carried in the organized sample metadata to ottIDs, validating that metadata
    was organized correctly.
  repo: https://github.com/enpkg/enpkg_full/tree/main/02_enpkg_taxo_enhancer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:4eb7a5283ae16963845a4c564d1e55dfa6a1917a515aea047314d7d133023837
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENPKG
  - ENPKG data-organization scripts (01_enpkg_data_organization)
  - GraphDB
  - MZmine
  - MassIVE
  - Open Tree of Life (ott3.5) taxo enhancer
  - RDF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# sample-wise-metabolomics-data-organization

## Summary

Restructures an external LC-MS/MS metabolomics dataset (e.g., .mzML and deconvoluted .mgf files retrieved from MassIVE) into a sample-wise folder architecture using the ENPKG data-organization scripts (01_enpkg_data_organization), so each sample can be processed individually and later integrated into the ENPKG knowledge graph.

## When to use

Use when ingesting an externally published LC-MS2 dataset (e.g., MSV000093464 from MassIVE) whose .mzML, .mgf, feature quantification tables, and metadata must be organized per sample before ENPKG graph building.

## When NOT to use

- The dataset is already a single consolidated feature table or aligned feature matrix across samples — the sample-centric organization would be redundant and the sample-level metadata mapping unavailable.
- The input is LC-MS data lacking per-sample taxonomic metadata (originating taxon), since the downstream Open Tree of Life (ott3.5) taxonomic enhancer cannot resolve sample identities.
- The goal is classical RT/m/z feature alignment across a single homogeneous batch — use a dataset-centric metabolomics workflow instead of reorganizing into a knowledge graph structure.

## Inputs

- LC-MS2 raw/converted data files (.mzML) retrieved from a MassIVE repository record
- Deconvoluted feature spectra (.mgf) exported from MZmine feature detection
- Feature quantification tables (m/z, retention time, area/intensity) per sample
- Sample metadata (producing organism / originating taxon, experimental conditions)

## Outputs

- Sample-wise folder architecture containing, per sample: quantification table, .mgf MS2 spectra, and metadata file
- Organized dataset ready for downstream ENPKG steps (taxo_enhancer, graph_builder) and RDF (.ttl) generation

## How to apply

Retrieve the LC-MS2 data (.mzML) and deconvoluted feature spectra (.mgf) for the target dataset from its MassIVE repository record (e.g., MSV000093464 for 337 Korean Pharmacopoeia plant extracts profiled in PI mode on a Q-ToF). Run the 01_enpkg_data_organization scripts to place feature quantification tables, MS2 spectra, and metadata (producing organism, etc.) in a sample-wise folder architecture, which is the structure the ENPKG workflow expects because each sample is processed individually. Ensure each sample carries a minimal metadata file with its originating taxon so the downstream 02_enpkg_taxo_enhancer can resolve names against Open Tree of Life v3.5 (ottID). Confirm the PI-mode feature list is linked through the property enpkg:has_lcms_feature_list_a137fd4a263d3587d35f61a526932c09 when the 06_enpkg_graph_builder scripts (pinned version d625e1a7a3365dd2fd70c66b00f8a59a80fd97a3) consume the organized folders. The rationale is that sample-centric organization lets heterogeneous datasets — differing in platform, run time, and solvent — join a shared knowledge graph without retention-time-based feature alignment.

## Related tools

- **ENPKG data-organization scripts (01_enpkg_data_organization)** (Scripts that place exported feature quantification tables, MS2 spectra, and metadata into the sample-wise folder architecture consumed by the ENPKG workflow.) — https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization
- **MassIVE** (Public repository from which the LC-MS2 (.mzML) and deconvoluted feature spectra (.mgf) files of the external dataset (e.g., MSV000093464 / MSV00008616) are retrieved.)
- **MZmine** (Tool that produces the per-sample feature detection outputs (quantification tables and deconvoluted .mgf spectra) that this organization step packages.) — http://mzmine.github.io/
- **Open Tree of Life (ott3.5) taxo enhancer** (Downstream step (02_enpkg_taxo_enhancer) that resolves the producing-organism names carried in the organized sample metadata to ottIDs, validating that metadata was organized correctly.) — https://github.com/enpkg/enpkg_full/tree/main/02_enpkg_taxo_enhancer

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && cd enpkg_full && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Every sample folder contains the complete trio: a feature quantification table, its MS2 spectra (.mgf), and a metadata file with the producing organism.
- All producing-organism names in the organized metadata resolve to ottIDs via the Open Tree of Life v3.5 taxonomy enhancer.
- The 06_enpkg_graph_builder scripts accept the organized folders and generate .ttl files in which the PI-mode feature list is resolvable under enpkg:has_lcms_feature_list_a137fd4a263d3587d35f61a526932c09.
- The generated .ttl files import into GraphDB and the integrated triples are retrievable at the ENPKG SPARQL endpoint (https://enpkg.commons-lab.org/graphdb/sparql).
- SPARQL queries over the combined graph retrieve cross-dataset spectral analog matches (e.g., scopolamine analogs shared between Datura metel / Scopolia japonica in the Korean dataset and the in-house extracts).

## Limitations

- The downstream tooling expects the dataset to be formatted in the exact same sample-wise folder structure as the reference example dataset; deviations will break processing (the emikg README explicitly warns it 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset').
- Requires LC-MS/MS DDA data in positive and/or negative ionization mode plus a minimal metadata file with the sample's originating taxon; other acquisition types are out of scope.
- Because processing is per-sample, there is no cross-batch RT-based feature alignment — this is intentional but means batch-level alignment artifacts are not corrected here.

## Evidence

- [methods] organize feature tables and metadata per sample: "organized in a sample-wise folder architecture using scripts available at https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization"
- [readme] sample-centric rationale: [evidence span withheld — anchor status: external_doc]
- [readme] per-sample input requirements: "For each sample, the required input data are 
1) A minimal metadata file containing the sample's originating taxon.
2) The LC-MS/MS DDA data (positive and/or negative ionization modes)."
- [results] origin of the files being organized: "the data for each sample are processed to perform feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine."
