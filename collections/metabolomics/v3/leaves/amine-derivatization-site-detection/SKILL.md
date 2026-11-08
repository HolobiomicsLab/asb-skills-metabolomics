---
name: amine-derivatization-site-detection
description: Use when predicting m/z values of amine-derivatized metabolites (e.g.
  dopamine, GABA, spermidine with AMPP) where reactive sites and the resulting mass
  shifts must be computed from structural SMILES before annotation.
when_to_use_negative:
- When metabolites lack derivatization-reactive amine functional groups (substructure
  matching will find zero sites and no derivative m/z can be computed)
- When the analysis target is pixel-level annotation of raw MALDI-MSI m/z features
  (use Met-ID's identify() workflow instead; this skill predicts candidate derivative
  masses upstream)
- When the derivatizing reagent is already the preconfigured FMP-10 matrix covered
  by Met-ID's built-in database and setup
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_1312
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: Met-ID
  role: Provides the local server (MetIDServer), the searchable HMDB/LIPID MAPS-derived
    database with SMILES (search_by_name), and the derivatization-aware annotation
    model (possible_derivs) that this site-detection workflow extends to AMPP
  repo: https://github.com/pbjarterot/Met-ID/releases
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:eb5bbd957655140ee3fc057317c244b03ea40520a41198c33a856acdf2b7109c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_1312
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# amine-derivatization-site-detection

## Summary

Detects derivatization-reactive amine functional groups in metabolite SMILES and computes per-site derivative m/z values by applying one AMPP reagent mass shift per reactive site, extending Met-ID's derivatization-aware annotation beyond the preconfigured in-house FMP-10 matrix.

## When to use

Use when predicting m/z values of amine-derivatized metabolites (e.g. dopamine, GABA, spermidine with AMPP) where reactive sites and the resulting mass shifts must be computed from structural SMILES before annotation.

## When NOT to use

- When metabolites lack derivatization-reactive amine functional groups (substructure matching will find zero sites and no derivative m/z can be computed)
- When the analysis target is pixel-level annotation of raw MALDI-MSI m/z features (use Met-ID's identify() workflow instead; this skill predicts candidate derivative masses upstream)
- When the derivatizing reagent is already the preconfigured FMP-10 matrix covered by Met-ID's built-in database and setup

## Inputs

- Metabolite names for a test panel (e.g. dopamine, GABA, spermidine) resolvable in the Met-ID database
- Metabolite SMILES strings retrieved via search_by_name(name, ccs=False)
- AMPP reagent mass shift applied once per derivatization-reactive amine site

## Outputs

- ampp_derivatization_sites.csv with columns: metabolite name, SMILES, derivatization-site count, derivative m/z values
- Per-molecule amine site annotations from RDKit substructure matching

## How to apply

Start the local Met-ID server via the MetIDServer context manager and retrieve database records (including SMILES) for each target metabolite with search_by_name(name, ccs=False). Use RDKit substructure matching to detect derivatization-reactive amine functional groups in each SMILES, mirroring how Met-ID 'utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent'. Count the derivatization sites per molecule (analogous to Met-ID's possible_derivs field) and compute each derivative m/z by adding one AMPP reagent mass shift per reactive site to the metabolite's protonated mass [M+H]+. Assemble a table (metabolite name, SMILES, derivatization-site count, derivative m/z values) saved as ampp_derivatization_sites.csv. This follows Met-ID's design principle that it is 'extendable to use any derivatizing matrix with the tools to do local version changes right from inside the software'.

## Related tools

- **Met-ID** (Provides the local server (MetIDServer), the searchable HMDB/LIPID MAPS-derived database with SMILES (search_by_name), and the derivatization-aware annotation model (possible_derivs) that this site-detection workflow extends to AMPP) — https://github.com/pbjarterot/Met-ID/releases

## Examples

```
with MetIDServer() as srv: recs = [srv.search_by_name(n, ccs=False) for n in ['dopamine','GABA','spermidine']]
```

## Evaluation signals

- Read ampp_derivatization_sites.csv back with pandas and verify all three panel metabolites (dopamine, GABA, spermidine) are present
- Each metabolite row has at least one annotated derivatization site (site count >= 1)
- Each derivative m/z is finite and consistent with one AMPP mass shift per annotated reactive site added to the protonated mass
- The derivatization-site count equals the number of RDKit amine substructure matches in the metabolite SMILES
- Derivative m/z values for multi-amine metabolites (e.g. spermidine) increase by the AMPP shift for each additional site

## Limitations

- The FMP-10 derivatizing matrix was developed in-house; its availability/distribution to other users is not stated, so preconfigured MS2 support does not transfer to AMPP derivatives
- Met-ID ships a preconfigured finite number of MS2 spectra (FMP-10 standards); there is no built-in AMPP MS2 spectral library for similarity comparison
- RDKit installation is nontrivial (often needs anaconda) and no pinned dependency versions are specified; on macOS there are known issues with adding functional groups
- All changes to the Met-ID database only affect the specific local install, not other users

## Evidence

- [abstract] structural-functionality-filter: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [abstract] multiple-derivatization-sites: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [readme] extendable-to-any-matrix: [evidence span withheld — anchor status: external_doc]
- [methods] server-start-method: "Import ``MetIDServer`` and use it as a context manager."
- [methods] database-search-method: "``search_by_name(name, ccs=False)`` searches by name."
