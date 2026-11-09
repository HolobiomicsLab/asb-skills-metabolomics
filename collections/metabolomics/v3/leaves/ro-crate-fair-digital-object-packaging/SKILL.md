---
name: ro-crate-fair-digital-object-packaging
description: Use when a computational workflow (e.g. a Nextflow .nf pipeline) must
  be deposited or shared with machine-understandable semantic metadata conforming
  to RO-Crate 1.1, rather than just published as code.
when_to_use_negative:
- The goal is merely to publish workflow source code under version control (plain
  GitHub hosting suffices; no semantic metadata packaging is required)
- The workflow steps have no available ontology terms and ontology annotation cannot
  be curated — the crate would lack the machine-understandable semantic metadata that
  motivates this skill
- The data are potentially personally identifiable omics or clinical data — privacy
  regulations prohibit sharing them, so the crate can only carry a synthetic dataset
  as in this case study
edam_topics:
- http://edamontology.org/topic_3068
tools:
- name: ro-crate-py
  role: Python library used to initialize the RO-Crate, register the workflow root
    entity, add and annotate data entities, and write the JSON-LD metadata descriptor
- name: Nextflow
  role: Workflow manager whose main script (action.nf) is registered as the crate's
    workflow root entity
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: WorkflowHub
  role: Registry where the packaged RO-Crate is registered and from which it can be
    downloaded as a zip (workflow 402)
  repo: https://workflowhub.eu/workflows/402
- name: GitHub
  role: Hosts the workflow repository and the generate_ro-crate.ipynb Jupyter Notebook
    demonstrating how the RO-Crate was made
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:8ed97bdff7952d005d2fae34e8008d506cb3d7c04e294634a9580fa1d0d0dc4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - GitHub
  - Nextflow
  - RO-Crate
  - WorkflowHub
  - ro-crate-py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3068
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# ro-crate-fair-digital-object-packaging

## Summary

Package a Nextflow multi-omics analysis workflow as a FAIR Digital Object by building an RO-Crate with ro-crate-py that embeds ontology-annotated (EDAM, NCIT, OBI, SIO) semantic metadata for the workflow, its data entities, and its CreateAction analysis steps, then register it on WorkflowHub so the workflow is findable and machine-readable.

## When to use

Use when a computational workflow (e.g. a Nextflow .nf pipeline) must be deposited or shared with machine-understandable semantic metadata conforming to RO-Crate 1.1, rather than just published as code.

## When NOT to use

- The goal is merely to publish workflow source code under version control (plain GitHub hosting suffices; no semantic metadata packaging is required)
- The workflow steps have no available ontology terms and ontology annotation cannot be curated — the crate would lack the machine-understandable semantic metadata that motivates this skill
- The data are potentially personally identifiable omics or clinical data — privacy regulations prohibit sharing them, so the crate can only carry a synthetic dataset as in this case study

## Inputs

- Nextflow workflow script (action.nf)
- Synthetic example data files in Synthetic_data/ (synthetic_metabolomics.csv, synthetic_epigenomics.csv, synthetic_epigenomics_meta.csv, synthetic_cbcl_data.csv, synthetic_phenotype_covariates_data.csv, cbcl_labels.csv, ACTIONdemonstrator_XOmics_IDS_synthetic.csv)
- ISA-Tab experimental metadata files
- Ontology term assignments from EBI's Ontology Lookup Service (EDAM, NCIT, OBI, SIO, eNanoMapper)

## Outputs

- ro-crate-metadata. (JSON-LD crate descriptor at the crate root per RO-Crate 1.1)
- RO-Crate zip archive containing the workflow, data files, ISA-Tab files, and annotated metadata
- Registered workflow page on WorkflowHub (workflow 402) with downloadable RO-Crate

## How to apply

Clone or stage the workflow repository (github.com/Xomics/ACTIONdemonstrator_workflow), locate the main Nextflow script action.nf and the Synthetic_data/ directory (synthetic_metabolomics.csv, synthetic_epigenomics.csv, synthetic_epigenomics_meta.csv, synthetic_cbcl_data.csv, synthetic_phenotype_covariates_data.csv, cbcl_labels.csv, and the IDS file) plus the ISA-Tab experimental metadata. Initialize an RO-Crate with the Python package ro-crate-py, registering action.nf as the crate's workflow root entity and adding each omics/phenotype file as an input data entity. Annotate each entity with ontology terms found via EBI's Ontology Lookup Service (EDAM, NCIT, OBI, SIO), falling back to the eNanoMapper Ontology for unsupervised-learning steps (MOFA, MCA, SNF) where EDAM has no matching term. Add the analysis steps as CreateAction entities linked to the mainEntity workflow, then write the JSON-LD crate descriptor ro-crate-metadata. at the crate root with the @context and conformance properties required by RO-Crate 1.1, and package everything (workflow, data, ISA-Tab files, metadata descriptor) into a distributable zip for WorkflowHub registration (workflow 402).

## Related tools

- **ro-crate-py** (Python library used to initialize the RO-Crate, register the workflow root entity, add and annotate data entities, and write the JSON-LD metadata descriptor)
- **Nextflow** (Workflow manager whose main script (action.nf) is registered as the crate's workflow root entity) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **WorkflowHub** (Registry where the packaged RO-Crate is registered and from which it can be downloaded as a zip (workflow 402)) — https://workflowhub.eu/workflows/402
- **GitHub** (Hosts the workflow repository and the generate_ro-crate.ipynb Jupyter Notebook demonstrating how the RO-Crate was made) — https://github.com/Xomics/ACTIONdemonstrator_workflow

## Examples

```
jupyter notebook generate_ro-crate.ipynb
```

## Evaluation signals

- ro-crate-metadata. exists at the crate root and is parseable JSON-LD loadable by ro-crate-py
- The crate contains the annotated workflow (action.nf as mainEntity), all input data entities, and CreateAction entities for the analysis steps (MOFA, MCA, SNF, and remaining steps with supported EDAM terms)
- Each entity carries valid ontology annotations, with eNanoMapper terms only for unsupervised-learning steps lacking EDAM matches
- The crate zip archive can be re-opened and all its entities resolved
- The packaged crate is downloadable from the registered WorkflowHub page (workflow 402)

## Limitations

- No exact ontology match for the ISA term 'sample collection' exists in EBI's Ontology Lookup Service, requiring manual curation choices
- eNanoMapper Ontology had to be used for unsupervised-learning steps because no matching EDAM term was available in the recommended ontology
- Manual workflow documentation/annotation is effortful; automated generation tooling was only potential future work

## Evidence

- [abstract] the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub.: "the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub."
- [readme] The Jupyter Notebook `generate_ro-crate.ipynb` demonstrates how the RO-Crate was made.: "The Jupyter Notebook [ `generate_ro-crate.ipynb`](generate_ro-crate.ipynb) demonstrates how the RO-Crate was made."
- [discussion] To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology.: "To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology [79] as no matching term was available in the recommended EDAM ontology."
- [methods] The workflow RO-Crate can be downloaded from the WorkflowHub page.: "The workflow RO-Crate can be downloaded from the [WorkflowHub page](https://workflowhub.eu/workflows/402)"
