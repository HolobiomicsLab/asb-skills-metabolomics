---
name: workflowhub-workflow-registration
description: Use when a computational workflow (e.g. a Nextflow pipeline like action.nf)
  must be made findable and reusable by packaging it with annotated metadata into
  an RO-Crate and publishing it on WorkflowHub.
when_to_use_negative:
- The data accompanying the workflow are potentially personally identifiable omics
  measurements or clinical information — privacy regulations prohibit sharing them,
  so only a synthetic dataset should be packaged.
- The goal is only to run the workflow locally — cloning the git repository suffices
  and no RO-Crate registration is needed.
- Required ontology terms cannot be identified even via fallback ontologies — semantic
  annotation quality would be too low for a meaningful FAIR Digital Object.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ro-crate-py
  role: Python package used to initialize the RO-Crate, register the workflow and
    data entities, and write the ro-crate-metadata. JSON-LD descriptor
- name: WorkflowHub
  role: Registry (part of EOSC) where the workflow RO-Crate is registered and from
    which the crate zip is downloadable
  repo: https://workflowhub.eu/workflows/402
- name: RO-Crate
  role: Open standard for packaging the workflow with rich human- and machine-readable
    semantic metadata
- name: GitHub
  role: Hosts the workflow repository and the generate_ro-crate.ipynb notebook that
    demonstrates how the RO-Crate was made
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:44dd220474207d090a0fc6b730c437c128f6d9d30117cb019b7afe3dbb378805
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - RO-Crate
  - WorkflowHub
  - ro-crate-py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# workflowhub-workflow-registration

## Summary

Package a scientific workflow as an RO-Crate FAIR Digital Object with machine-readable semantic metadata and register it on WorkflowHub so it is findable and downloadable as a single crate zip archive.

## When to use

Use when a computational workflow (e.g. a Nextflow pipeline like action.nf) must be made findable and reusable by packaging it with annotated metadata into an RO-Crate and publishing it on WorkflowHub.

## When NOT to use

- The data accompanying the workflow are potentially personally identifiable omics measurements or clinical information — privacy regulations prohibit sharing them, so only a synthetic dataset should be packaged.
- The goal is only to run the workflow locally — cloning the git repository suffices and no RO-Crate registration is needed.
- Required ontology terms cannot be identified even via fallback ontologies — semantic annotation quality would be too low for a meaningful FAIR Digital Object.

## Inputs

- Workflow root script (e.g. action.nf Nextflow main script)
- Example/synthetic dataset files (synthetic_metabolomics.csv, synthetic_epigenomics.csv, synthetic_epigenomics_meta.csv, synthetic_cbcl_data.csv, synthetic_phenotype_covariates_data.csv, cbcl_labels.csv, IDS file)
- ISA-Tab experimental metadata files describing the multi-omics (meta)data
- Ontology terms from EBI's Ontology Lookup Service (EDAM, NCIT, OBI, SIO, eNanoMapper)

## Outputs

- ro-crate-metadata. (JSON-LD crate descriptor conforming to RO-Crate 1.1)
- RO-Crate zip archive containing workflow, data, ISA-Tab files, and metadata descriptor
- WorkflowHub registration page hosting the downloadable crate (e.g. workflowhub.eu/workflows/402)

## How to apply

Clone or stage the workflow repository and identify the root workflow script (e.g. action.nf) plus all example data such as the Synthetic_data/ directory with synthetic_metabolomics.csv, synthetic_epigenomics.csv, synthetic_cbcl_data.csv, and ISA-Tab files. Initialize the RO-Crate with the Python package ro-crate-py via the provided Jupyter Notebook (generate_ro-crate.ipynb), registering action.nf as the crate's workflow root entity and the data files as input data entities. Annotate each entity with ontology terms from EBI's Ontology Lookup Service (EDAM, NCIT, OBI, SIO; fall back to the eNanoMapper Ontology for unsupervised-learning steps like MOFA, MCA, and SNF where no EDAM term exists), and link analysis steps to the workflow as CreateAction entities with a mainEntity relation. Write the JSON-LD descriptor ro-crate-metadata. conforming to the RO-Crate 1.1 specification at the crate root, then package the whole crate (workflow, data, ISA-Tab, descriptor) into a zip archive and register it on WorkflowHub so it can be downloaded from the workflow's page.

## Related tools

- **ro-crate-py** (Python package used to initialize the RO-Crate, register the workflow and data entities, and write the ro-crate-metadata. JSON-LD descriptor)
- **WorkflowHub** (Registry (part of EOSC) where the workflow RO-Crate is registered and from which the crate zip is downloadable) — https://workflowhub.eu/workflows/402
- **RO-Crate** (Open standard for packaging the workflow with rich human- and machine-readable semantic metadata)
- **GitHub** (Hosts the workflow repository and the generate_ro-crate.ipynb notebook that demonstrates how the RO-Crate was made) — https://github.com/Xomics/ACTIONdemonstrator_workflow.git

## Examples

```
jupyter notebook generate_ro-crate.ipynb  # builds RO-Crate with ro-crate-py for upload to https://workflowhub.eu/workflows/402
```

## Evaluation signals

- ro-crate-metadata. exists at the crate root and is parseable JSON-LD loadable by ro-crate-py
- The crate contains the workflow entity (action.nf) as mainEntity/workflow root, all annotated input data entities, and CreateAction entities for each analysis step
- All entity annotations resolve to ontology terms in EBI's OLS (or the documented eNanoMapper fallback for unsupervised-learning steps)
- The crate zip can be re-opened and its entities resolved
- The workflow appears on WorkflowHub with an accessible page offering the RO-Crate download (e.g. workflowhub.eu/workflows/402)

## Limitations

- No matching EDAM term was available for unsupervised-learning workflow steps, so the eNanoMapper Ontology had to be employed instead
- No exact ontology match exists for the generic ISA term 'sample collection' in any ontology in EBI's Ontology Lookup Service
- Privacy regulations prohibit sharing potentially personally identifiable data such as omics measurements or clinical information, so only a synthetic dataset can be included in the public crate
- Manual workflow documentation for the crate is effortful; automated generation tooling is only potential future work

## Evidence

- [abstract] semantic metadata via WorkflowHub registration: "the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub."
- [methods] crate creation with ro-crate-py: "The workflow RO-Crate can be downloaded from the [WorkflowHub page](https://workflowhub.eu/workflows/402)"
- [discussion] eNanoMapper fallback: "To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology [79] as no matching term was available in the recommended EDAM ontology."
- [readme] README crate contents: "The Jupyter Notebook [ `generate_ro-crate.ipynb`](generate_ro-crate.ipynb) demonstrates how the RO-Crate was made."
