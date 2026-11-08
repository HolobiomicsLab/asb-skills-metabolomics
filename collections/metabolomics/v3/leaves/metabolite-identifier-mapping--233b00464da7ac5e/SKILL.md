---
name: metabolite-identifier-mapping
description: Use when working with scFEA model files such as cmMat_171.csv, module_gene_m168.csv,
  or cName_c70_m168.csv and compound IDs must be joined to names or classified as
  intermediate substrates connecting modules.
when_to_use_negative:
- The input is raw scRNA-seq expression data rather than scFEA model/compound files
  — flux estimation, not identifier mapping, is needed
- A context-specific metabolic network must be reconstructed; the article lists this
  as a future direction not yet available
- Mapping is needed for organisms other than the supplied human or mouse models —
  only human (module_gene_m168.csv) and mouse complete metabolic flux modules are
  released
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2259
tools:
- name: scFEA
  role: Supplies the reorganized human metabolic map files (module_gene_m168.csv,
    cmMat_171.csv, cName_c70_m168.csv) whose compound identifiers are mapped and validated
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web implementation of the scFEA pipeline over the same curated metabolic networks
    (RECON3D-, KEGG-, and literature-based), providing the module/compound nomenclature
    being mapped
  repo: https://github.com/changwn/scFEA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:b9aae7312d9aabc9c8c204f62ac8a5677fa4b54eec0a891c0ec445e75b7eb905
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/changwn/scFEA
  repo_url: https://github.com/changwn/scFEA
  tools:
  - FLUXestimator
  - scFEA
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metabolite-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps compound identifiers in scFEA/FLUXestimator metabolic model files to human-readable metabolite names and to their roles (module inputs, outputs, intermediate substrates) so that the reorganized human metabolic map (e.g. the 171-module M171 model) can be interpreted, tallied, and validated.

## When to use

Use when working with scFEA model files such as cmMat_171.csv, module_gene_m168.csv, or cName_c70_m168.csv and compound IDs must be joined to names or classified as intermediate substrates connecting modules.

## When NOT to use

- The input is raw scRNA-seq expression data rather than scFEA model/compound files — flux estimation, not identifier mapping, is needed
- A context-specific metabolic network must be reconstructed; the article lists this as a future direction not yet available
- Mapping is needed for organisms other than the supplied human or mouse models — only human (module_gene_m168.csv) and mouse complete metabolic flux modules are released

## Inputs

- cmMat_171.csv stoichiometry matrix (rows = intermediate metabolites, columns = metabolic modules)
- cName_c70_m168.csv compound-name file (one row compound names, one row compound ids, contains NA values)
- module_gene_m168.csv module-to-gene table for the human M171 model

## Outputs

- Compound id-to-name mapping table with NA-flagged unmapped ids
- Classification of compounds as module inputs, outputs, or intermediate substrates
- m171_network_composition.csv with module, super module, intermediate substrate, gene, enzyme, transporter, reaction, and metabolite counts

## How to apply

Load the stoichiometry matrix (cmMat_171.csv), where each row is an intermediate metabolite and each column is a metabolic module, with pandas, and load the compound-name file (e.g. cName_c70_m168.csv), which stores compound names in one row and corresponding compound ids in another. Join the compound ids from the stoichiometry matrix to the name table by id, explicitly handling the documented NA values in cName_c70_m168.csv (fixed as of scFEA v1.1.2) so unmapped ids are flagged rather than silently dropped. Then classify each compound by its column profile: compounds appearing as an output of one module and an input of another are intermediate substrates that link modules in the module-to-module graph used to form super modules. Assemble the mapped identifiers into a structured table (e.g. m171_network_composition.csv) whose counts — 171 reaction modules, 22 super modules, 66 intermediate substrates — are cross-checked against the published network composition.

## Related tools

- **scFEA** (Supplies the reorganized human metabolic map files (module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv) whose compound identifiers are mapped and validated) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web implementation of the scFEA pipeline over the same curated metabolic networks (RECON3D-, KEGG-, and literature-based), providing the module/compound nomenclature being mapped) — https://github.com/changwn/scFEA

## Examples

```
import pandas as pd; sto = pd.read_csv('cmMat_171.csv', index_col=0); cnames = pd.read_csv('cName_c70_m168.csv'); mapped = sto.index.to_series().map(dict(zip(cnames.iloc[1], cnames.iloc[0])))
```

## Evaluation signals

- Recomputed counts reproduce the reported composition: 171 reaction modules, 22 super modules, 66 intermediate substrates
- Every compound id in cmMat_171.csv resolves to a name or is explicitly flagged NA, matching the documented NA fix in v1.1.2
- Number of modules in module_gene_m168.csv equals the number of columns in cmMat_171.csv
- Gene products classified as enzymes vs transporters sum to the gene total per module, matching paper-reported totals (expert_review of tallied totals)
- Output CSV (e.g. m171_network_composition.csv) has one row per module/super-module tally and no dropped intermediate substrates

## Limitations

- cName_c70_m168.csv historically contained NA values; users must handle these explicitly when joining ids to names
- Context-specific (disease, tissue, cell type specific) network reconstructions are not yet available, so mapping is limited to the released general human/mouse maps
- Perturbation analysis and newly curated modules (e.g. methionine, copper ion) exist only in the stand-alone version pending validation

## Evidence
