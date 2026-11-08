---
name: cross-feeding-dependency-verification
description: Use when you have gap-filled genome-scale community models (e.g. Matlab
  workspaces from COMMIT gap filling) and need to enumerate extracellular exchanges
  and validate which members depend on which secreted metabolites via uptake-blocking
  FBA.
when_to_use_negative:
- The models have not yet been gap-filled (draft reconstructions lack the exchange
  reactions needed for dependency testing)
- The goal is to build or gap-fill community reconstructions rather than to verify
  cross-feeding interactions in existing functional models
- Only individual isolate models are available without community composition, since
  SMETANA requires a collection of models in SBML format
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SMETANA
  role: Downstream directed interaction prediction on gap-filled community models
    to compute cross-feeding metrics
  repo: https://github.com/cdanielmachado/smetana
- name: COBRA Toolbox
  role: Matlab LP framework for sink-reaction maximization and uptake-blocking FBA
    with CPLEX/Gurobi
  repo: https://github.com/opencobra/cobratoolbox
- name: COMMIT
  role: Produces the community-dependent gap-filled reconstructions whose exchanges
    are analyzed
  repo: https://github.com/pwendering/COMMIT
- name: pheatmap
  role: R package for clustered heatmap visualization of the biomass-reduction dependency
    matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:85b6162cd8aaed4bdfb5f396a3dc932434d8922479c320fb3ab7adf10e8ca2c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cdanielmachado/smetana
  tools:
  - COBRA Toolbox
  - COMMIT
  - SMETANA
  - pheatmap
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Cross-feeding dependency verification

## Summary

Verifies predicted cross-feeding interactions in microbial community GEMs by combining SMETANA-directed interaction prediction with explicit COBRA-toolbox FBA experiments (sink-reaction maximization at suboptimal biomass factor f=0.9, uptake-blocking biomass-reduction assays) on gap-filled community models, producing helper/beneficiary roles and member-by-metabolite dependency matrices for the Bulgarelli and Schlaeppi soil communities.

## When to use

Use when you have gap-filled genome-scale community models (e.g. Matlab workspaces from COMMIT gap filling) and need to enumerate extracellular exchanges and validate which members depend on which secreted metabolites via uptake-blocking FBA.

## When NOT to use

- The models have not yet been gap-filled (draft reconstructions lack the exchange reactions needed for dependency testing)
- The goal is to build or gap-fill community reconstructions rather than to verify cross-feeding interactions in existing functional models
- Only individual isolate models are available without community composition, since SMETANA requires a collection of models in SBML format

## Inputs

- Gap-filled community metabolic models for the two soil communities (Matlab workspaces from run_iterative_gap_filling.m)
- SBML genome-scale metabolic models as accepted by SMETANA
- MNXref-normalized metabolite/reaction identifiers with PubChem-derived permeability properties (HBD, HBA, MW, TPSA, RB, XLogP3)

## Outputs

- Per-member maximal secretion/leakage flux contributions to the exchanged-metabolite pool
- Member-by-uptake-reaction biomass-reduction dependency matrix
- Helper/beneficiary community role assignments
- Clustered pheatmap visualization of the dependency matrix

## How to apply

Load the gap-filled community models for the Bulgarelli and Schlaeppi soil communities into the COBRA toolbox v3.0 and enumerate extracellular exchange/uptake reactions. For each exchange reaction, compute each member's maximal secretion/leakage flux with sink-reaction maximization constrained to suboptimal biomass (factor f = 0.9, as used in the COMMIT study). Then, for each uptake reaction, run an LP solved with CPLEX or Gurobi in which that uptake is blocked and record the resulting biomass flux reduction for every community member. Cross-check computed dependencies against the reported mineral dependencies (Fe2+, Fe3+, SO42-), which appear across most bacterial families and are also provided by members of most families, and against the community-specific amino-acid and within-family exchange patterns. Finally render the member-by-uptake-reaction biomass-reduction matrix as a clustered heatmap with pheatmap, and accept the result only if helper/beneficiary roles and mineral/amino-acid dependencies match the published pattern.

## Related tools

- **SMETANA** (Downstream directed interaction prediction on gap-filled community models to compute cross-feeding metrics) — https://github.com/cdanielmachado/smetana
- **COBRA Toolbox** (Matlab LP framework for sink-reaction maximization and uptake-blocking FBA with CPLEX/Gurobi) — https://github.com/opencobra/cobratoolbox
- **COMMIT** (Produces the community-dependent gap-filled reconstructions whose exchanges are analyzed) — https://github.com/pwendering/COMMIT
- **pheatmap** (R package for clustered heatmap visualization of the biomass-reduction dependency matrix)

## Examples

```
git clone https://github.com/pwendering/COMMIT
```

## Evaluation signals

- Reproduced heatmap and dependency calls match the published helper/beneficiary interaction pattern for both soil communities
- Most models across bacterial families show putative dependencies for Fe2+, Fe3+, and SO42-, which are simultaneously provided by members of most families
- Sink-reaction maximization uses suboptimal biomass factor f = 0.9 (per S1 Text protocol)
- Amino-acid and specialized exchanges are limited within families and consistent per community (Bulgarelli vs Schlaeppi)
- LP solutions are reproducible with an installed solver (CPLEX v12.9 tested with COBRA toolbox v3.0)

## Limitations

- Only sensitivity and precision could be computed for model quality since true negatives could not be determined
- 16S rRNA abundance data were only available by personal provision (Ruben Garrido Oter), not a public accession
- Gap-filling penalty weights feeding into the analyzed models were tuned ad hoc without a systematic selection procedure
- Full COMMIT procedure specification (ordering criteria, S1 Text details) is not in the main text

## Evidence

- [abstract] Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
- [discussion] In this study, f = 0.9 has been used (please see S1 Text for more detail).: "In this study, f = 0.9 has been used (please see S1 Text for more detail)."
- [intro] we conducted a downstream analysis of the gap-filled models using the SMETANA tool: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [readme] This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al. 2012, Schlaeppi et al. 2014).: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al."
