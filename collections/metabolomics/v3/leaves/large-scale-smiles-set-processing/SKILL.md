---
name: large-scale-smiles-set-processing
description: Use when a hydrogen-to-functional-group enumeration step yields a very
  large SMILES set that must be reduced by per-molecule rule evaluation (drug-like
  filters, messy rings >7 SSSR, ΔSCScore) before downstream prediction.
when_to_use_negative:
- Input is a small curated compound list where full medicinal-chemistry filtering
  is unnecessary and would discard valid candidates
- The analyte spectrum already matches a reference library (NIST, Wiley, SWGDRUG),
  so no synthetic NPS database needs to be built
- You need spectra prediction or similarity ranking (NEIMS/DeepEI steps), not derivative
  filtering
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: rdkit
  role: 'Molecular processing for evaluating filter rules: ring perception via smallest
    sets of smallest rings, valence/stability checks, and property calculations on
    each derivative SMILES'
  repo: https://www.rdkit.org/docs/Install.html
- name: PS2MS
  role: Provides the enumeration-then-filter pipeline and C++/CMake enumeration step
    whose output SMILES set is filtered here
  repo: https://github.com/jhhung/PS2MS
- name: SCScore
  role: Supplies the synthetic complexity score used as the ΔSCScore threshold (αn)
    to drop functional groups that are highly challenging to synthesize
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:383dba828895716af1a3948bf9b62376fdea9503193cd860aba86d7fabdd76de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# large-scale-smiles-set-processing

## Summary

Rule-based screening of very large enumerated SMILES sets (here ~422 million cathinone derivatives) using RDKit ring perception and 57 drug-like criteria from Virshup et al., plus ΔSCScore thresholds, to reduce derivatives to a manageable synthetic NPS database.

## When to use

Use when a hydrogen-to-functional-group enumeration step yields a very large SMILES set that must be reduced by per-molecule rule evaluation (drug-like filters, messy rings >7 SSSR, ΔSCScore) before downstream prediction.

## When NOT to use

- Input is a small curated compound list where full medicinal-chemistry filtering is unnecessary and would discard valid candidates
- The analyte spectrum already matches a reference library (NIST, Wiley, SWGDRUG), so no synthetic NPS database needs to be built
- You need spectra prediction or similarity ranking (NEIMS/DeepEI steps), not derivative filtering

## Inputs

- Enumerated derivative SMILES set from hydrogen-to-functional-group substitution on a core drug structure (e.g. cathinone)
- List of 57 drug-like criteria (Virshup et al.)
- SCScore thresholds (αn) per functional group

## Outputs

- Filtered compound set (filtered_compounds.smi) of derivatives passing all criteria, scaled to ~422 million compounds
- Per-rule elimination count table (filter_counts.csv) ranking rules by number of molecules removed

## How to apply

Load the enumerated derivative SMILES set produced upstream by substituting hydrogen atoms in the core drug structure with functional groups. For each molecule, use RDKit ring perception (smallest sets of smallest rings), valence/stability checks, and property calculations to evaluate the 57 drug-like criteria from Virshup et al.; tally per-rule elimination counts (e.g. filter_counts.csv) and rank rules by removal count. Confirm the Messy rings rule (more than 7 smallest sets of smallest rings) removes the largest share, as reported in Figure 2d. Retain only derivatives passing all 57 criteria (and functional groups whose ΔSCScore falls below threshold αn) into the filtered set (e.g. filtered_compounds.smi) feeding NEIMS spectrum prediction. Validation requires the per-rule counts to reproduce the paper's pattern that Messy rings is the most effective filter and that the surviving set is on the order of approximately 422 million compounds.

## Related tools

- **rdkit** (Molecular processing for evaluating filter rules: ring perception via smallest sets of smallest rings, valence/stability checks, and property calculations on each derivative SMILES) — https://www.rdkit.org/docs/Install.html
- **PS2MS** (Provides the enumeration-then-filter pipeline and C++/CMake enumeration step whose output SMILES set is filtered here) — https://github.com/jhhung/PS2MS
- **SCScore** (Supplies the synthetic complexity score used as the ΔSCScore threshold (αn) to drop functional groups that are highly challenging to synthesize)

## Examples

```
from rdkit import Chem; mols = [Chem.MolFromSmiles(s) for s in open('enumerated.smi')]; survivors = [s for s, m in zip(open('enumerated.smi'), mols) if m is not None and len(Chem.GetSymmSSSR(m)) <= 7]
```

## Evaluation signals

- Per-rule counts (filter_counts.csv) must reproduce the paper's reported pattern that the Messy rings rule (>7 smallest sets of smallest rings) removes the largest share of molecules (Figure 2d)
- The surviving filtered set must be consistent with the paper's reported scale of approximately 422 million compounds after enumeration and filtering
- Every surviving derivative must pass all 57 Virshup et al. drug-like criteria — spot-check random SMILES with RDKit against the rules
- Retained functional groups must have average ΔSCScore below the αn threshold
- Output file (filtered_compounds.smi) must contain valid, parseable SMILES consumable by the downstream NEIMS spectrum prediction step

## Limitations

- The exact number of enumeration levels is unspecified in the paper and has a similar impact on the resulting database scale
- Screening by SCScore avoids highly challenging-to-synthesize compounds but may exclude derivatization routes that are nonetheless feasible
- Building the pipeline requires building RDKit's C++ code from source and installing the python package from conda; python 3.6.9 or higher and CMake 3.16.0+ are needed

## Evidence

- [methods] Scale of filtered database: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [methods] Drug-like criteria source: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] Most effective filter rule: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [discussion] SCScore-based functional-group screening: "We utilize these scores to screen functional groups, thus avoiding the inclusion of highly challenging-to-synthesize compounds in the enumeration process"
- [readme] RDKit installation requirement: "rdkit](https://www.rdkit.org/docs/Install.html)  - build the c++ code from the source and install python package from conda"
