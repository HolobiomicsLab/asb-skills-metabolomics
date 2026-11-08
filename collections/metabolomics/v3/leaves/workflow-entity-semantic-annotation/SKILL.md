---
name: workflow-entity-semantic-annotation
description: Use when a Nextflow (or similar) workflow has been packaged as an RO-Crate
  and its data entities and CreateAction steps need machine-readable semantic metadata
  to make the crate findable and FAIR-compliant.
when_to_use_negative:
- The workflow is already registered on WorkflowHub with a generated RO-Crate — re-annotating
  manually is redundant
- You only need human-readable workflow documentation (e.g. a Documentation.md) rather
  than machine-understandable semantic metadata
- The entities are real, personally identifiable omics/clinical data — these cannot
  be packaged and shared; only synthetic data was distributable in this case study
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ro-crate-py
  role: Python library used to initialize the RO-Crate, register action.nf as the
    workflow root entity, add data entities, and write the JSON-LD ro-crate-metadata.
    descriptor
- name: Nextflow
  role: Workflow manager whose main script (action.nf) and analysis steps become the
    annotated entities of the crate
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
- name: WorkflowHub
  role: Registry where the packaged, semantically annotated RO-Crate is published
    and made findable (workflow 402)
  repo: https://workflowhub.eu/workflows/402
- name: EBI Ontology Lookup Service
  role: Used to determine appropriate ontology terms (EDAM, NCIT, OBI, SIO) for each
    entity and step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:6295c6ec8a306e8c01106a717ba4e7d20382b21fb0c0cd7f8efb5f13bd53eaa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
  tools:
  - EBI Ontology Lookup Service
  - GitHub
  - Nextflow
  - WorkflowHub
  - ro-crate-py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# workflow-entity-semantic-annotation

## Summary

Annotate the entities of an RO-Crate packaged computational workflow (workflow root, input data files, analysis steps) with ontology terms (EDAM, NCIT, OBI, SIO, eNanoMapper) so the crate is a machine-readable FAIR Digital Object. Terms are selected via EBI's Ontology Lookup Service and written as JSON-LD metadata with ro-crate-py.

## When to use

Use when a Nextflow (or similar) workflow has been packaged as an RO-Crate and its data entities and CreateAction steps need machine-readable semantic metadata to make the crate findable and FAIR-compliant.

## When NOT to use

- The workflow is already registered on WorkflowHub with a generated RO-Crate — re-annotating manually is redundant
- You only need human-readable workflow documentation (e.g. a Documentation.md) rather than machine-understandable semantic metadata
- The entities are real, personally identifiable omics/clinical data — these cannot be packaged and shared; only synthetic data was distributable in this case study

## Inputs

- Nextflow workflow script (action.nf) and its repository
- Input data entities (CSV omics/phenotype files, IDS file, ISA-Tab experimental metadata)
- Ontology term candidates from EBI's Ontology Lookup Service (EDAM, NCIT, OBI, SIO, eNanoMapper)
- ro-crate-py Python package

## Outputs

- ro-crate-metadata. (JSON-LD crate descriptor conforming to RO-Crate 1.1)
- Annotated RO-Crate with workflow root, data entities, and CreateAction step entities
- Distributable crate zip archive registered on WorkflowHub (workflow 402)

## How to apply

Initialize an RO-Crate with ro-crate-py, registering the main Nextflow script (action.nf) as the crate's workflow root entity and adding each input file (e.g. synthetic_metabolomics.csv, synthetic_epigenomics.csv, synthetic_epigenomics_meta.csv, synthetic_cbcl_data.csv, synthetic_phenotype_covariates_data.csv, cbcl_labels.csv, the IDS file) as data entities. For each entity, search EBI's Ontology Lookup Service for a suitable EDAM term (plus NCIT, OBI, or SIO where appropriate) and attach it; when no EDAM term exists — as with the MOFA, MCA, and SNF unsupervised-learning steps — fall back to the eNanoMapper Ontology. Model workflow analysis steps as CreateAction entities linked to the workflow via mainEntity, assigning semantic types to each step. Write the descriptor as ro-crate-metadata. (JSON-LD) at the crate root conforming to RO-Crate 1.1, then package workflow, data, ISA-Tab files, and descriptor into the distributable crate zip for WorkflowHub registration.

## Related tools

- **ro-crate-py** (Python library used to initialize the RO-Crate, register action.nf as the workflow root entity, add data entities, and write the JSON-LD ro-crate-metadata. descriptor)
- **Nextflow** (Workflow manager whose main script (action.nf) and analysis steps become the annotated entities of the crate) — https://github.com/Xomics/ACTIONdemonstrator_workflow.git
- **WorkflowHub** (Registry where the packaged, semantically annotated RO-Crate is published and made findable (workflow 402)) — https://workflowhub.eu/workflows/402
- **EBI Ontology Lookup Service** (Used to determine appropriate ontology terms (EDAM, NCIT, OBI, SIO) for each entity and step)

## Examples

```
from rocrate.rocrate import ROCrate; crate = ROCrate(); crate.add_workflow("action.nf", mainEntity=True); crate.write("ro-crate-metadata.")
```

## Evaluation signals

- ro-crate-metadata. exists at the crate root and is parseable JSON-LD loadable by ro-crate-py
- The crate contains the workflow root entity (action.nf), all input data entities, and CreateAction step entities linked via mainEntity
- Every analysis step carries a semantic type — EDAM where available, eNanoMapper for unsupervised-learning steps (MOFA, MCA, SNF)
- The crate zip archive can be re-opened and its entities resolved programmatically
- The crate downloads correctly from the WorkflowHub page (workflowhub.eu/workflows/402)

## Limitations

- No matching EDAM term existed for unsupervised-learning workflow steps, forcing use of the eNanoMapper Ontology
- No exact ontology match exists in EBI's OLS for the generic ISA term 'sample collection'
- Choosing appropriate ontology terms when recommended ontologies lack suitable terms is challenging and effortful; the crate was generated manually via the generate_ro-crate.ipynb notebook, with automated tooling only potential future work

## Evidence

- [abstract] the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub.: "the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub."
- [discussion] To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology [79] as no matching term was available in the recommended EDAM ontology.: "To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology [79] as no matching term was available in the recommended EDAM ontology."
- [discussion] no exact match to the generic term sample collection that is part of the ISA schema can be found in any ontology available in EBI’s Ontology Lookup Serivce (OLS): "no exact match to the generic term sample collection that is part of the ISA schema can be found in any ontology available in EBI’s Ontology Lookup Serivce (OLS)"
- [readme] The Jupyter Notebook [ `generate_ro-crate.ipynb`](generate_ro-crate.ipynb) demonstrates how the RO-Crate was made.: "The Jupyter Notebook [ `generate_ro-crate.ipynb`](generate_ro-crate.ipynb) demonstrates how the RO-Crate was made."
