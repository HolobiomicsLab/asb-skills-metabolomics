---
name: rdkit-molecular-property-filtering
description: Use when a large enumerated set of derivative SMILES must be reduced
  by rule-based structural filters (ring count, drug-like property criteria, synthetic-accessibility
  score) before downstream spectrum prediction.
when_to_use_negative:
- When the analyte's spectrum already matches a known reference library (NIST, Wiley,
  SWGDRUG) — filtering/enumeration is only triggered when no reference match is found
- When you need to predict spectra for a fixed, known compound list rather than prune
  a large enumerated candidate set
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: rdkit
  role: 'Molecular processing for the filter step: ring perception via smallest sets
    of smallest rings, valence/stability checks, and property calculations used to
    evaluate each drug-like rule against every derivative'
  repo: https://www.rdkit.org/docs/Install.html
- name: PS2MS
  role: Pipeline whose enumeration-and-filtering stage consumes the RDKit-based rule
    evaluation to produce the synthetic NPS database of ~422 million compounds
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:0166f9b468fa860ec22b8a859cac0141e5410bbdc6851fd3f93a4cfa69001d24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rdkit-molecular-property-filtering

## Summary

Rule-based molecular screening of an enumerated derivative set (SMILES) using RDKit to apply drug-like criteria (Virshup et al., 57 rules), messy-rings limits (>7 smallest sets of smallest rings), and ΔSCScore thresholds, reducing candidates to a manageable synthetic NPS database (~422 million compounds in PS²MS).

## When to use

Use when a large enumerated set of derivative SMILES must be reduced by rule-based structural filters (ring count, drug-like property criteria, synthetic-accessibility score) before downstream spectrum prediction.

## When NOT to use

- When the analyte's spectrum already matches a known reference library (NIST, Wiley, SWGDRUG) — filtering/enumeration is only triggered when no reference match is found
- When you need to predict spectra for a fixed, known compound list rather than prune a large enumerated candidate set

## Inputs

- Enumerated derivative SMILES set (one structure per line) from hydrogen-to-functional-group enumeration
- Filter rule definitions: 57 drug-like criteria (Virshup et al.), messy-rings threshold (>7 SSSR), ΔSCScore threshold αn
- Core drug structure (e.g., cathinone) used as the enumeration template

## Outputs

- filter_counts.csv — per-rule elimination counts
- filtered_compounds.smi — SMILES of derivatives passing all filter rules
- Ranked list of filter rules by number of compounds removed

## How to apply

Load the enumerated derivative SMILES produced by hydrogen-to-functional-group substitution on the core drug structure. For each molecule, use RDKit ring perception (smallest sets of smallest rings) to count rings and evaluate all 57 drug-like criteria from Virshup et al., plus the ΔSCScore-based rule retaining only functional groups whose average synthesis-difficulty differential falls below threshold αn. Tally the number of derivatives each rule eliminates, write the per-rule counts to filter_counts.csv, and rank rules by elimination count — in PS²MS the messy rings rule (>7 SSSR) was the most effective. Assemble survivors of all rules into filtered_compounds.smi for downstream NEIMS spectrum prediction, expecting the retained set to sit near the paper's reported ~422 million compounds.

## Related tools

- **rdkit** (Molecular processing for the filter step: ring perception via smallest sets of smallest rings, valence/stability checks, and property calculations used to evaluate each drug-like rule against every derivative) — https://www.rdkit.org/docs/Install.html
- **PS2MS** (Pipeline whose enumeration-and-filtering stage consumes the RDKit-based rule evaluation to produce the synthetic NPS database of ~422 million compounds) — https://github.com/jhhung/PS2MS

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smiles); rings = Chem.GetSymmSSSR(mol); keep = len(rings) <= 7
```

## Evaluation signals

- filter_counts.csv shows the messy rings rule (>7 smallest sets of smallest rings) removing the largest share of molecules, matching Figure 2d's reported pattern
- Size of filtered_compounds.smi is consistent with the paper's reported scale of approximately 422 million compounds after enumeration and filtering
- Every surviving SMILES passes all 57 drug-like criteria and has ≤7 SSSR rings
- Retained functional groups have ΔSCScore below threshold αn (no highly challenging-to-synthesize compounds)

## Limitations

- Thresholds are rule-specific and paper-tuned (e.g., >7 SSSR rings, ΔSCScore < αn); the exact number of enumeration levels is not specified and impacts the filtered output size
- Filtering is designed for derivative enumeration from a drug core (e.g., cathinone); it does not address identification when reference spectra exist
- RDKit must be built from source and the python package installed from conda per the PS2MS README

## Evidence

- [methods] 57 drug-like criteria and messy rings: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] most effective rule: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] SCScore threshold: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [methods] output scale: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [readme] enumeration input definition: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
