---
name: network-composition-summarization
description: Use when a task asks to reproduce or verify reported composition statistics
  of a metabolic module network (e.g., 171 modules, 22 super modules, 66 intermediate
  substrates in Figure 1 of the scFEA/FLUXestimator paper) from its module-gene and
  stoichiometry files.
when_to_use_negative:
- Input is not the scFEA/FLUXestimator module-gene and stoichiometry file pair — the
  method depends on the specific CSV schema (rows = intermediate metabolites, columns
  = modules)
- The goal is to run flux prediction on single-cell data rather than to summarize
  the static structure of the metabolic map
edam_operation: http://edamontology.org/operation_0222
tools:
- name: scFEA
  role: Source of the human metabolic map files (module_gene_m168.csv, cmMat_171.csv,
    cName_c70_m168.csv) that are parsed and summarized
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web server whose underlying curated metabolic networks (integrating RECON3D,
    KEGG and literature data) the composition summarization characterizes
  repo: https://github.com/changwn/scFEA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:a30ac41a95efab55afa3a602c9b54af15ae41bb43365bdfffccc9c487f974d2f
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# network-composition-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recomputes and tabulates the structural composition of a curated metabolic module network (e.g., the scFEA M171 human metabolic map) by parsing module-gene and stoichiometry-matrix files to count reaction modules, super modules, intermediate substrates, genes, enzymes, transporters, reactions, and metabolites.

## When to use

Use when a task asks to reproduce or verify reported composition statistics of a metabolic module network (e.g., 171 modules, 22 super modules, 66 intermediate substrates in Figure 1 of the scFEA/FLUXestimator paper) from its module-gene and stoichiometry files.

## When NOT to use

- Input is not the scFEA/FLUXestimator module-gene and stoichiometry file pair — the method depends on the specific CSV schema (rows = intermediate metabolites, columns = modules)
- The goal is to run flux prediction on single-cell data rather than to summarize the static structure of the metabolic map

## Inputs

- module_gene_m168.csv (module-to-gene mapping table from scFEA /data/ folder)
- cmMat_171.csv (stoichiometry matrix; rows = intermediate metabolites, columns = metabolic modules)
- cName_c70_m168.csv (compound-name table mapping compound IDs to names, containing NA values)

## Outputs

- m171_network_composition.csv (structured table of network composition counts: 171 reaction modules, 22 super modules, 66 intermediate substrates, plus gene/enzyme/transporter/reaction/metabolite totals)

## How to apply

Load module_gene_m168.csv, cmMat_171.csv, and the compound-name file (cName_c70_m168.csv, handling its documented NA values) with pandas. Parse the stoichiometry matrix — each row is an intermediate metabolite and each column is a metabolic module — to enumerate distinct reactions, metabolites, and module input/output compounds, flagging compounds that are outputs of one module and inputs of another as intermediate substrates. Build a module-to-module graph over shared intermediate substrates and partition the 171 modules into their 22 super modules. Tally genes per module from the module-gene table and classify gene products as enzymes or transporters, then assemble all counts into a single structured table (e.g., m171_network_composition.csv).

## Related tools

- **scFEA** (Source of the human metabolic map files (module_gene_m168.csv, cmMat_171.csv, cName_c70_m168.csv) that are parsed and summarized) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web server whose underlying curated metabolic networks (integrating RECON3D, KEGG and literature data) the composition summarization characterizes) — https://github.com/changwn/scFEA

## Examples

```
python src/scFEA.py --data_dir data --moduleGene_file module_gene_m168.csv --stoichiometry_matrix cmMat_171.csv --cName_file cName_c70_m168.csv
```

## Evaluation signals

- Recomputed module count equals 171 and super-module partition yields 22 super modules
- Number of intermediate substrates (compounds that are outputs of one module and inputs of another) equals 66
- Gene, enzyme, transporter, reaction, and metabolite totals match the paper-reported values (expert_review on the tallied totals)
- NA values in cName_c70_m168.csv are handled without dropping valid compound entries
- Output file m171_network_composition.csv is written with a single structured row of all counts

## Limitations

- The compound-name file cName_c70_m168.csv historically contained NA values that must be handled during parsing
- The M171 map is one of several candidate models; human uses module_gene_m168.csv and cmMat_171.csv by default, and other candidate files exist in /data/ for mouse and other networks
- The scFEA map is a manually reorganized human metabolic map of focused modules, not a full genome-scale reconstruction; context-specific network reconstruction was listed as future work in FLUXestimator

## Evidence
