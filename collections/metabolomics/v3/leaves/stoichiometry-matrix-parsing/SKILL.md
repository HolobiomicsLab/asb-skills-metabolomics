---
name: stoichiometry-matrix-parsing
description: Use when a task requires reconstructing the structure of the scFEA human
  metabolic map from its CSV model files — counting reaction modules, super modules,
  and intermediate substrates, or tallying genes/enzymes/transporters per module.
when_to_use_negative:
- The goal is to estimate cell-wise metabolic fluxes from scRNA-seq data rather than
  characterize the static network structure of the metabolic map
- The input model is the mouse model or another non-default module-gene/stoichiometry
  file set (other candidate files are provided in scFEA's /data/ folder and should
  be used instead of the m168/m171 defaults)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scFEA
  role: Provides the model files (module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv)
    whose stoichiometry and module structure are parsed in this skill
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Webserver implementation of the scFEA pipeline whose curated human metabolic
    maps (from RECON3D, KEGG and literature data) are the source of the module definitions
  repo: https://github.com/changwn/scFEA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:d33e06db66cdad21a52b7a2248cd1ab73a1e11ecee18d7f0af205019d400dfcd
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# stoichiometry-matrix-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse the scFEA stoichiometry matrix (e.g. cmMat_171.csv), module-gene table (module_gene_m168.csv), and compound-name files (e.g. cName_c70_m168.csv) with pandas to enumerate reaction modules, metabolites, and intermediate substrates, then assemble the module-to-module graph used to reproduce the M171 network composition statistics.

## When to use

Use when a task requires reconstructing the structure of the scFEA human metabolic map from its CSV model files — counting reaction modules, super modules, and intermediate substrates, or tallying genes/enzymes/transporters per module.

## When NOT to use

- The goal is to estimate cell-wise metabolic fluxes from scRNA-seq data rather than characterize the static network structure of the metabolic map
- The input model is the mouse model or another non-default module-gene/stoichiometry file set (other candidate files are provided in scFEA's /data/ folder and should be used instead of the m168/m171 defaults)

## Inputs

- cmMat_171.csv stoichiometry matrix (rows = intermediate metabolites, columns = metabolic modules)
- module_gene_m168.csv module-gene table (human model, scFEA default)
- cName_c70_m168.csv compound-name file (compound names and ids, contains NA values)

## Outputs

- m171_network_composition.csv structured table of module, super-module, intermediate-substrate, gene, enzyme, transporter, reaction, and metabolite totals
- Module-to-module connectivity graph over shared intermediate substrates

## How to apply

Load module_gene_m168.csv, cmMat_171.csv, and cName_c70_m168.csv with pandas, handling the documented NA values in the compound-name file. Parse the stoichiometry matrix where each row is an intermediate metabolite and each column is a metabolic module, to enumerate distinct reactions, metabolites, and module input/output compounds; flag intermediate substrates that are outputs of one module and inputs of another. Build the module-to-module graph over these shared intermediates and partition the 171 modules into their 22 super modules, tallying genes per module from the module-gene table and classifying gene products as enzymes or transporters. Write all counts into a single structured table (e.g. m171_network_composition.csv) and verify the recomputed composition matches the reported 171 reaction modules in 22 super modules with 66 intermediate substrates.

## Related tools

- **scFEA** (Provides the model files (module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv) whose stoichiometry and module structure are parsed in this skill) — https://github.com/changwn/scFEA
- **FLUXestimator** (Webserver implementation of the scFEA pipeline whose curated human metabolic maps (from RECON3D, KEGG and literature data) are the source of the module definitions) — https://github.com/changwn/scFEA

## Examples

```
import pandas as pd; cm = pd.read_csv('cmMat_171.csv', index_col=0); mg = pd.read_csv('module_gene_m168.csv'); cn = pd.read_csv('cName_c70_m168.csv', keep_default_na=False)
```

## Evaluation signals

- Recomputed counts reproduce the reported network composition: 171 reaction modules organized into 22 super modules with 66 intermediate substrates
- Gene, enzyme, transporter, reaction, and metabolite totals match the paper-reported values (expert review of the tallied totals)
- Every intermediate substrate flagged as connecting lies in exactly two module input/output sets (output of one module, input of another)
- The output m171_network_composition.csv is a single structured table covering all count categories without missing values from the cName NA entries
- Module count tallied from module_gene_m168.csv agrees with the number of columns in cmMat_171.csv

## Limitations

- The compound-name file cName_c70_m168.csv contains NA values that must be handled during parsing (documented bug fix in scFEA v1.1.2)
- Counts must be re-derived from the provided CSVs; the composition statistics come from the paper's Figure 1 description and should be validated against the supplementary Table S1 (reorganized human metabolic map)
- Other candidate module-gene and stoichiometry matrix files exist in scFEA's /data/ folder; the m168/m171 human defaults are not the only valid model

## Evidence

- [other] Reported network composition finding: [evidence span withheld — anchor status: not_found]
