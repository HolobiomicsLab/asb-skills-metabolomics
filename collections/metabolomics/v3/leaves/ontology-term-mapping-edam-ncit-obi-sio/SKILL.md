---
name: ontology-term-mapping-edam-ncit-obi-sio
description: Use when packaging a computational workflow as an RO-Crate that needs
  semantic annotation of data entities and workflow steps, and ontology lookups in
  EBI's OLS are required (with fallback handling when EDAM has no matching term).
when_to_use_negative:
- When the task is annotating ISA-Tab experimental metadata itself rather than packaging
  an executable workflow as an RO-Crate — the ISA term 'sample collection' has no
  ontology match in EBI's OLS, so this skill cannot supply one.
- When the workflow will not be shared or registered (e.g., WorkflowHub) and human-readable
  documentation suffices; the article notes that manual semantic annotation is effortful.
- When EDAM or the recommended ontologies do contain suitable terms but the goal is
  free-text description — do not invent custom terms outside the chosen ontologies.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: ro-crate-py
  role: Python package used to initialize the RO-Crate, register the workflow root
    entity, add CreateAction entities, and write the JSON-LD metadata descriptor
- name: EBI Ontology Lookup Service (OLS)
  role: Used to determine whether suitable EDAM/NCIT/OBI/SIO terms exist for each
    entity before choosing a fallback ontology
- name: eNanoMapper Ontology
  role: Fallback vocabulary for unsupervised-learning workflow steps (MOFA, MCA, SNF)
    where no matching EDAM term is available
- name: WorkflowHub
  role: Registry where the annotated RO-Crate is registered and from which the crate
    zip can be downloaded
  repo: https://workflowhub.eu/workflows/402
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:97314300de8b5a832bc851939988cb2a856bd62aada3c98b2fe21c1f48568605
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EBI Ontology Lookup Service (OLS)
  - GitHub
  - WorkflowHub
  - eNanoMapper Ontology
  - ro-crate-py
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ontology-term-mapping-edam-ncit-obi-sio

## Summary

Annotates RO-Crate entities (workflow, input data, CreateAction analysis steps) with machine-readable ontology terms drawn from EDAM, NCIT, OBI, and SIO via EBI's Ontology Lookup Service, falling back to the eNanoMapper Ontology where EDAM lacks suitable terms. This is what turns a packaged computational workflow into a findable, interoperable FAIR Digital Object.

## When to use

Use when packaging a computational workflow as an RO-Crate that needs semantic annotation of data entities and workflow steps, and ontology lookups in EBI's OLS are required (with fallback handling when EDAM has no matching term).

## When NOT to use

- When the task is annotating ISA-Tab experimental metadata itself rather than packaging an executable workflow as an RO-Crate — the ISA term 'sample collection' has no ontology match in EBI's OLS, so this skill cannot supply one.
- When the workflow will not be shared or registered (e.g., WorkflowHub) and human-readable documentation suffices; the article notes that manual semantic annotation is effortful.
- When EDAM or the recommended ontologies do contain suitable terms but the goal is free-text description — do not invent custom terms outside the chosen ontologies.

## Inputs

- Nextflow workflow script (action.nf) to be registered as the crate's mainEntity
- Input data entities: synthetic_metabolomics.csv, synthetic_epigenomics.csv, synthetic_epigenomics_meta.csv, synthetic_cbcl_data.csv, synthetic_phenotype_covariates_data.csv, cbcl_labels.csv, ACTIONdemonstrator_XOmics_IDS_synthetic.csv
- ISA-Tab experimental metadata files
- Workflow analysis steps to classify (MOFA, MCA, SNF, normalization, correlation, cluster testing)

## Outputs

- ro-crate-metadata. (JSON-LD crate descriptor with @context and conformance-to-standard properties per RO-Crate 1.1)
- Annotated RO-Crate with ontology-typed data entities and CreateAction entities linked to the workflow mainEntity
- Distributable crate zip archive suitable for registration on WorkflowHub (workflow 402)

## How to apply

Query EBI's Ontology Lookup Service (OLS) for candidate terms for each entity type, preferring EDAM terms for workflow steps and NCIT/OBI/SIO terms for input data entities (e.g., the synthetic_metabolomics.csv, synthetic_epigenomics.csv, and cbcl_labels.csv files of the ACTION demonstrator). In ro-crate-py, register the main Nextflow script (action.nf) as the crate's workflow root entity, then add each analysis step as a CreateAction entity linked to the mainEntity workflow and assign its semantic type. When no matching EDAM term exists — as for the unsupervised-learning steps MOFA, MCA, and SNF — fall back to the eNanoMapper Ontology rather than omitting the annotation. Note that some ISA-schema terms cannot be mapped at all: no ontology in EBI's OLS contains an exact match for the generic term 'sample collection'. Finally, write the annotated crate descriptor as JSON-LD (ro-crate-metadata.) conforming to the RO-Crate 1.1 specification.

## Related tools

- **ro-crate-py** (Python package used to initialize the RO-Crate, register the workflow root entity, add CreateAction entities, and write the JSON-LD metadata descriptor)
- **EBI Ontology Lookup Service (OLS)** (Used to determine whether suitable EDAM/NCIT/OBI/SIO terms exist for each entity before choosing a fallback ontology)
- **eNanoMapper Ontology** (Fallback vocabulary for unsupervised-learning workflow steps (MOFA, MCA, SNF) where no matching EDAM term is available)
- **WorkflowHub** (Registry where the annotated RO-Crate is registered and from which the crate zip can be downloaded) — https://workflowhub.eu/workflows/402

## Examples

```
import rocrate; crate = rocrate.ROCrate(); crate.add_workflow("action.nf", "action.nf", mainEntity=True); crate.write("ro-crate-metadata.")
```

## Evaluation signals

- ro-crate-metadata. parses as valid JSON-LD loadable by ro-crate-py
- The crate contains the annotated workflow (mainEntity = action.nf) plus all listed input data entities, each carrying ontology annotations
- CreateAction entities for the workflow steps are linked to the mainEntity workflow and carry semantic types (e.g., MOFA/MCA/SNF via eNanoMapper, remaining steps via supported EDAM terms)
- Every assigned term can be resolved in EBI's OLS (or is a documented eNanoMapper fallback)
- The crate zip re-opens and all entities resolve; the descriptor conforms to RO-Crate 1.1 with embedded @context

## Limitations

- No matching EDAM term exists for unsupervised-learning workflow steps, forcing use of the non-recommended eNanoMapper Ontology.
- The generic ISA term 'sample collection' has no exact match in any ontology available in EBI's OLS, so some ISA-schema entities cannot be fully annotated.
- Choosing appropriate terms when recommended ontologies lack suitable entries is challenging, and manual annotation of workflow documentation is effortful; automated term suggestion is only potential future work.

## Evidence

- [abstract] the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub.: "the workflow was described with rich semantic metadata, packaged as a Research Object Crate, and shared via WorkflowHub."
- [discussion] To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology [79] as no matching term was available in the recommended EDAM ontology.: "To describe workflow steps in RO-Crate with unsupervised learning, we had to employ the eNanoMapper Ontology [79] as no matching term was available in the recommended EDAM ontology."
- [discussion] no exact match to the generic term sample collection that is part of the ISA schema can be found in any ontology available in EBI’s Ontology Lookup Serivce (OLS) [78].: "no exact match to the generic term sample collection that is part of the ISA schema can be found in any ontology available in EBI’s Ontology Lookup Serivce (OLS) [78]."
- [readme] The Jupyter Notebook `generate_ro-crate.ipynb` demonstrates how the RO-Crate was made.: "The Jupyter Notebook [ `generate_ro-crate.ipynb`](generate_ro-crate.ipynb) demonstrates how the RO-Crate was made."
