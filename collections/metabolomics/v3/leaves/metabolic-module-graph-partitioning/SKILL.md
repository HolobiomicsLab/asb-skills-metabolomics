---
name: metabolic-module-graph-partitioning
description: 'Use when you need to reproduce or verify the composition of a curated
  metabolic module map (e.g. scFEA''s M171 human metabolic map): counting reaction
  modules, super modules, intermediate substrates, genes, enzymes, transporters, reactions
  and metabolites.'
when_to_use_negative:
- The goal is flux estimation on single-cell data rather than characterizing the structure
  of the module map itself
- You need a context-specific (disease, tissue, or cell type specific) metabolic network
  reconstruction — the article lists this as a future capability, not an existing
  one
- Input data are raw scRNA-seq expression matrices rather than the scFEA model files
  (module-gene table, stoichiometry matrix, compound names)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: scFEA
  role: Provides the curated human metabolic map artifacts (module_gene_m168.csv,
    cmMat_171.csv, cName_c70_m168.csv) that are parsed and partitioned into super
    modules
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Webserver exposing the same curated metabolic networks (15 manually curated
    networks integrating RECON3D, KEGG and literature data) whose module structure
    this skill characterizes
  repo: https://github.com/changwn/scFEA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:56a0fc296500288dd38c4bf54b1d0a20c501ce6b4a6cfabad0d0212bb953af6f
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolic-module-graph-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the scFEA M171 metabolic map by parsing the stoichiometry matrix (cmMat_171.csv) and module-gene table (module_gene_m168.csv) to enumerate reaction modules, identify intermediate substrates shared between modules, and partition the 171 modules into 22 super modules.

## When to use

Use when you need to reproduce or verify the composition of a curated metabolic module map (e.g. scFEA's M171 human metabolic map): counting reaction modules, super modules, intermediate substrates, genes, enzymes, transporters, reactions and metabolites.

## When NOT to use

- The goal is flux estimation on single-cell data rather than characterizing the structure of the module map itself
- You need a context-specific (disease, tissue, or cell type specific) metabolic network reconstruction — the article lists this as a future capability, not an existing one
- Input data are raw scRNA-seq expression matrices rather than the scFEA model files (module-gene table, stoichiometry matrix, compound names)

## Inputs

- module_gene_m168.csv (module-gene table listing genes per metabolic module)
- cmMat_171.csv (stoichiometry matrix: rows = intermediate metabolites, columns = metabolic modules)
- cName_c70_m168.csv (compound names and corresponding ids, contains NA values)

## Outputs

- m171_network_composition.csv (structured table of counts: reaction modules, super modules, intermediate substrates, genes, enzymes, transporters, reactions, metabolites)

## How to apply

Load the module-gene table (module_gene_m168.csv), the stoichiometry matrix (cmMat_171.csv), and the compound-name files (e.g. cName_c70_m168.csv, handling documented NA values) with pandas. Parse the stoichiometry matrix — where each row is an intermediate metabolite and each column a metabolic module — to enumerate distinct reactions, metabolites, and module input/output compounds, flagging intermediate substrates that are outputs of one module and inputs of another. Build a module-to-module graph over these shared intermediate substrates and partition the 171 modules into their 22 super modules (connected components of the graph). Tally genes per module from the module-gene table and classify gene products as enzymes or transporters, then assemble all counts into a single structured table written to m171_network_composition.csv.

## Related tools

- **scFEA** (Provides the curated human metabolic map artifacts (module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv) that are parsed and partitioned into super modules) — https://github.com/changwn/scFEA
- **FLUXestimator** (Webserver exposing the same curated metabolic networks (15 manually curated networks integrating RECON3D, KEGG and literature data) whose module structure this skill characterizes) — https://github.com/changwn/scFEA

## Examples

```
import pandas as pd; cm = pd.read_csv('cmMat_171.csv', index_col=0); mg = pd.read_csv('module_gene_m168.csv'); cn = pd.read_csv('cName_c70_m168.csv')
```

## Evaluation signals

- Recomputed module-to-module graph yields exactly 171 connected reaction modules
- Graph partitioning recovers 22 super modules from the 171 modules
- Intermediate-substrate tally (outputs of one module that are inputs of another) equals 66
- Gene, enzyme, transporter, reaction, and metabolite totals match the paper-reported values after expert review of the tallied table
- Output file m171_network_composition.csv contains all counts in a single structured table with no parsing errors from NA values in cName_c70_m168.csv

## Limitations

- The cName_c70_m168.csv compound-name file historically contained NA values (fixed in scFEA v1.1.2) and must be handled explicitly during parsing
- Counts are tied to the specific M171 human map shipped with scFEA; other candidate moduleGene and stoichiometry files in the /data/ folder will yield different compositions
- Biochemical constraints such as redox, pH, or energy balance are not part of the map structure; the partitioning reflects module connectivity only

## Evidence
