---
name: sssr-ring-perception-analysis
description: Use when a rule-based drug-like filter must be applied to an enumerated
  derivative SMILES set and per-ring-count screening (messy rings, >7 smallest sets
  of smallest rings) is one of the filtering criteria.
when_to_use_negative:
- The goal is spectrum or fingerprint prediction (NEIMS/DeepEI) or analyte ranking
  rather than structure filtering of the enumerated database
- Input structures are not enumerated derivatives of a drug core (e.g., raw experimental
  spectra or already-curated reference libraries such as NIST/SWGDRUG)
- The filtering task only involves ΔSCScore synthetic-difficulty screening or mass-weight
  (M+ peak tolerance) filtering and no ring-count criterion is required
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: rdkit
  role: Molecular processing toolkit used for ring perception (smallest sets of smallest
    rings) and evaluation of the 57 drug-like criteria against each derivative SMILES
  repo: https://www.rdkit.org/docs/Install.html
- name: PS2MS
  role: Pipeline whose enumeration-and-filtering stage contains the messy rings rule;
    the enumeration step builds the synthetic database by substituting hydrogens in
    the core structure
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:c055e2327c6d922286ff15d0249d0000f5e6faa3d56812b302826ebdd1e6c166
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sssr-ring-perception-analysis

## Summary

Computes smallest sets of smallest rings (SSSR) for enumerated derivative SMILES in the PS²MS pipeline and applies the 'messy rings' rule (>7 SSSR) — the single most effective drug-like filter — to prune the cathinone-derivative set down to the ~422-million-compound synthetic NPS database.

## When to use

Use when a rule-based drug-like filter must be applied to an enumerated derivative SMILES set and per-ring-count screening (messy rings, >7 smallest sets of smallest rings) is one of the filtering criteria.

## When NOT to use

- The goal is spectrum or fingerprint prediction (NEIMS/DeepEI) or analyte ranking rather than structure filtering of the enumerated database
- Input structures are not enumerated derivatives of a drug core (e.g., raw experimental spectra or already-curated reference libraries such as NIST/SWGDRUG)
- The filtering task only involves ΔSCScore synthetic-difficulty screening or mass-weight (M+ peak tolerance) filtering and no ring-count criterion is required

## Inputs

- Enumerated derivative SMILES set from the hydrogen-to-functional-group enumeration step (one SMILES per candidate)
- Core drug template structure (e.g., cathinone) used to generate the derivatives

## Outputs

- Per-rule elimination counts written to filter_counts.csv (including the messy-rings count)
- Filtered compound set filtered_compounds.smi (SMILES passing all 57 drug-like criteria including messy rings)
- Synthetic NPS database of approximately 422 million compounds feeding NEIMS spectrum prediction

## How to apply

Load the enumerated derivative SMILES produced by hydrogen-to-functional-group substitution on the core drug structure (e.g., cathinone). For each molecule, use RDKit ring perception to obtain the smallest sets of smallest rings (Chem.GetSymmSSSR) and flag any molecule whose SSSR count exceeds 7 as 'messy rings'. Evaluate this rule alongside the 57 drug-like criteria from Virshup et al. and the ΔSCScore threshold (αn) on synthetic-difficulty screening, tallying the number of derivatives each individual rule eliminates (written to filter_counts.csv). Rank rules by elimination count to confirm that messy rings is the most effective filter, as reported in Figure 2d, then assemble the survivors that pass all criteria into filtered_compounds.smi for downstream NEIMS spectrum prediction. Rationale: derivatives with more than 7 smallest rings are structurally implausible as drug-like NPS candidates, so this single rule removes the largest share of enumerated structures.

## Related tools

- **rdkit** (Molecular processing toolkit used for ring perception (smallest sets of smallest rings) and evaluation of the 57 drug-like criteria against each derivative SMILES) — https://www.rdkit.org/docs/Install.html
- **PS2MS** (Pipeline whose enumeration-and-filtering stage contains the messy rings rule; the enumeration step builds the synthetic database by substituting hydrogens in the core structure) — https://github.com/jhhung/PS2MS

## Examples

```
from rdkit import Chem; n_sssr = len(Chem.GetSymmSSSR(Chem.MolFromSmiles(smi))); keep = n_sssr <= 7
```

## Evaluation signals

- filter_counts.csv shows the messy rings rule (>7 SSSR) removes the largest share of derivatives among all 57 drug-like criteria, matching the paper's Figure 2d pattern
- The surviving filtered set is consistent with the paper's reported scale of approximately 422 million compounds after enumeration and filtering
- Every SMILES in filtered_compounds.smi has an SSSR count ≤ 7 when re-computed with RDKit
- Compounds removed by the messy rings rule all exceed 7 smallest sets of smallest rings under RDKit ring perception (no false eliminations)
- The filtered set is a strict subset of the enumerated derivative input SMILES, with no molecules added or altered

## Limitations

- The threshold is a fixed drug-like heuristic (>7 SSSR) from Virshup et al.; it may remove structurally exotic but synthetically feasible polycyclic candidates
- The article does not specify the exact number of enumeration levels, which similarly impacts the size of the set this filter operates on
- Validation depends on reproducing the paper's qualitative per-rule pattern rather than an exact per-rule count, so the messy-rings dominance is a relative check

## Evidence

- [methods] Messy rings is the most effective filter rule: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] 57 drug-like criteria introduced from Virshup et al.: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] Enumeration and filtering yields ~422 million compounds: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [readme] Enumeration builds the synthetic database the filter operates on: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
