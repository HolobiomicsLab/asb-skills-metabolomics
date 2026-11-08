---
name: recursive-derivative-enumeration-from-drug-template
description: Use when a GC/EI-MS analyte spectrum matches no reference library and
  you need to enumerate plausible derivatives of a known drug core to populate a searchable
  synthetic database.
when_to_use_negative:
- The analyte's mass spectrum already matches an established reference spectrum (NIST,
  Wiley, or SWGDRUG) — conventional screening suffices and enumeration is unnecessary
- The target chemical structure is already known and you only need predicted EI mass
  spectra (NEIMS handles that case directly without derivative enumeration)
- Only functional groups with very high synthetic difficulty are of interest — SCScore
  screening deliberately excludes highly challenging-to-synthesize compounds from
  enumeration
edam_operation: http://edamontology.org/operation_3676
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: PS2MS
  role: Provides the enumeration and rule-based filtering module that generates the
    synthetic NPS database by substituting hydrogen atoms in the core structure with
    functional groups
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Performs the hydrogen-to-R-group replacement on the template SMILES during
    recursive enumeration
  repo: https://www.rdkit.org/docs/Install.html
- name: SCScore
  role: Computes average synthetic complexity scores used to screen functional groups
    whose ΔSCScore exceeds alpha*n, excluding hard-to-synthesize derivatives
- name: python
  role: Runtime (3.6.9 or higher) for the deep learning and enumeration tooling
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:03a242a3609d6e6318ae372f10f3df1077a191c59f357390be44d89c5cb3b596
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3676
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# recursive-derivative-enumeration-from-drug-template

## Summary

Builds a synthetic NPS database by recursively substituting hydrogen atoms in a drug's core structure (e.g., cathinone) with functional groups using RDKit, then filtering derivatives by SCScore-based synthetic-difficulty thresholds and Virshup drug-like criteria to keep only plausibly synthesizable compounds.

## When to use

Use when a GC/EI-MS analyte spectrum matches no reference library and you need to enumerate plausible derivatives of a known drug core to populate a searchable synthetic database.

## When NOT to use

- The analyte's mass spectrum already matches an established reference spectrum (NIST, Wiley, or SWGDRUG) — conventional screening suffices and enumeration is unnecessary
- The target chemical structure is already known and you only need predicted EI mass spectra (NEIMS handles that case directly without derivative enumeration)
- Only functional groups with very high synthetic difficulty are of interest — SCScore screening deliberately excludes highly challenging-to-synthesize compounds from enumeration

## Inputs

- Cathinone (or other drug) core template SMILES
- Functional-group R-group definitions (204 compiled groups; 30 UNODC groups for the restricted run)
- SCScore model for computing synthetic complexity scores
- RDKit Python installation (built from source, installed via conda)
- Enumeration depth parameter n=3 and threshold alpha=1.2

## Outputs

- Derivative SMILES sets per configuration (204-group and 30-group runs)
- Synthetic NPS database of ~422 million (full) and ~2.3 million (UNODC subset) enumerated cathinone derivatives
- Summary log recording compound count per configuration and enumeration runtime

## How to apply

Load the cathinone template SMILES and functional-group definitions (204 groups for the full configuration, or the 30 UNODC groups for the restricted configuration), preserving the heavy-atom core while iteratively replacing hydrogens with R-groups via RDKit. Recurse to depth n=3; at each depth n>1 compute the average SCScore of n-fold modified compounds per functional group and retain only groups whose average ΔSCScore falls below alpha*n with alpha=1.2. Filter the enumerated derivatives with the 57 drug-like criteria of Virshup et al., applying the most effective rule first (Messy rings: more than 7 smallest sets of smallest rings) to save computational resources. Run the enumeration twice — once with all 204 groups and once with only the 30 UNODC groups — writing derivative SMILES per configuration and logging compound counts (~422 million for the 204-group run, ~2.3 million for the 30-group run) and runtime. Validation requires that the full-parameter run (alpha=1.2, n=3) recovers all known illicit cathinone-type drugs in the UNODC database, with functional-group retention of 131 groups at the second and 97 at the third recursion level.

## Related tools

- **PS2MS** (Provides the enumeration and rule-based filtering module that generates the synthetic NPS database by substituting hydrogen atoms in the core structure with functional groups) — https://github.com/jhhung/PS2MS
- **rdkit** (Performs the hydrogen-to-R-group replacement on the template SMILES during recursive enumeration) — https://www.rdkit.org/docs/Install.html
- **SCScore** (Computes average synthetic complexity scores used to screen functional groups whose ΔSCScore exceeds alpha*n, excluding hard-to-synthesize derivatives)
- **python** (Runtime (3.6.9 or higher) for the deep learning and enumeration tooling) — https://www.python.org/downloads/

## Examples

```
cd enumeration && mkdir build && cd build && cmake .. && make   # build and run the enumeration step per jhhung/PS2MS enumeration/README.md
```

## Evaluation signals

- The full-parameter run (alpha=1.2, n=3) must recover all known illicit cathinone-type drugs listed in the UNODC database (Figure S2)
- Functional-group retention should reach 131 groups at the second and 97 at the third recursion level
- The 204-group run should yield approximately 422 million compounds and the 30-group UNODC run approximately 2.3 million
- Filtered derivatives should respect the Messy rings rule (no more than 7 smallest sets of smallest rings) and the 57 Virshup et al. drug-like criteria
- Summary log must record compound count and enumeration runtime for each configuration

## Limitations

- Enumeration depth strongly affects database size — 'The number of enumeration levels also has a similar impact' on resources and results
- SCScore screening excludes highly challenging-to-synthesize compounds, so genuinely novel but synthetically difficult derivatives may be missed
- The validation shown is specific to cathinone-type drugs; generalization to other drug templates is asserted but demonstrated only for the 30 UNODC cathinone functional groups

## Evidence

- [methods] recursive enumeration procedure: [evidence span withheld — anchor status: not_found]
- [methods] SCScore thresholding: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [methods] filtering rules: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] scale of output: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [readme] README description of enumeration: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
