---
name: molecular-graph-processing-rdkit
description: Use when building a synthetic compound database by applying functional
  groups to a template molecule's graph in RDKit and computing per-derivative molecular
  scores for filtering.
when_to_use_negative:
- The analyte's chemical structure is already known and matches a reference spectrum,
  so no derivative enumeration is needed
- Predicting spectra or fingerprints of existing compounds (NEIMS/DeepEI tasks) rather
  than enumerating and filtering derivatives
- The goal is structural de novo generation of NPS (e.g., DarkNPS-style generative
  modeling) rather than template-based substitution
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: rdkit
  role: Molecular-graph toolkit used to parse SMILES, build the cathinone template,
    substitute hydrogen atoms with functional groups at recursion levels n = 2 and
    n = 3, and represent derivatives for scoring; installed by building the C++ code
    from source and installing the Python package from conda
  repo: https://www.rdkit.org/docs/Install.html
- name: SCScore
  role: Pretrained neural network applied to the RDKit-generated derivative molecules
    to compute synthetic-complexity scores, averaged per functional group into ΔSCScore
    for the retention threshold
- name: python
  role: Runtime (3.6.9 or higher) for the deep-learning tools that consume the enumerated
    derivative molecules
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:3a32f447329d44cce870b99908a341beed84f7ece8222deecb4ba6d61404289e
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# molecular-graph-processing-rdkit

## Summary

Uses RDKit to manipulate molecular graphs (SMILES parsing, hydrogen-atom substitution with functional groups on a drug template such as cathinone) so that enumerated derivatives can be scored (e.g., with SCScore) and filtered during synthetic NPS database construction.

## When to use

Use when building a synthetic compound database by applying functional groups to a template molecule's graph in RDKit and computing per-derivative molecular scores for filtering.

## When NOT to use

- The analyte's chemical structure is already known and matches a reference spectrum, so no derivative enumeration is needed
- Predicting spectra or fingerprints of existing compounds (NEIMS/DeepEI tasks) rather than enumerating and filtering derivatives
- The goal is structural de novo generation of NPS (e.g., DarkNPS-style generative modeling) rather than template-based substitution

## Inputs

- Cathinone drug template molecule (SMILES)
- Enumerated functional-group list (204 compiled functional groups from task_001)
- Pretrained SCScore neural-network model
- Recursion depth n (2 and 3) and threshold α = 1.2

## Outputs

- scscore_filter_results.csv with per-functional-group ΔSCScore values and per-level (n=2, n=3) pass flags
- Functional groups passing the threshold (131 at level 2, 97 at level 3)
- Substituted derivative compounds for downstream mass-spectrum prediction (~422 million compounds after full enumeration and filtering)

## How to apply

Load the core drug structure (e.g., cathinone) and the enumerated functional-group list (204 functional groups compiled in task_001) as RDKit molecule objects from SMILES. Generate derivatives by substituting hydrogen atoms in the template graph with each functional group, applying the group n times (n = 2 and n = 3) to produce the recursion-level compound sets. Compute the pretrained SCScore of the cathinone template as the synthetic-difficulty baseline, then per functional group average the SCScore over its substituted compounds to obtain the difficulty differential ΔSCScore. Retain only groups with ΔSCScore < α^n using α = 1.2 per recursion level, and combine with rule-based filters (e.g., Messy rings > 7 smallest sets of smallest rings, 57 drug-like criteria per Virshup et al.). Write the per-group score table with pass/fail flags to scscore_filter_results.csv.

## Related tools

- **rdkit** (Molecular-graph toolkit used to parse SMILES, build the cathinone template, substitute hydrogen atoms with functional groups at recursion levels n = 2 and n = 3, and represent derivatives for scoring; installed by building the C++ code from source and installing the Python package from conda) — https://www.rdkit.org/docs/Install.html
- **SCScore** (Pretrained neural network applied to the RDKit-generated derivative molecules to compute synthetic-complexity scores, averaged per functional group into ΔSCScore for the retention threshold)
- **python** (Runtime (3.6.9 or higher) for the deep-learning tools that consume the enumerated derivative molecules)

## Examples

```
python -c "from rdkit import Chem; tmpl = Chem.MolFromSmiles('CC(=O)C(N)Cc1ccccc1'); print(Chem.MolToSmiles(tmpl))"
```

## Evaluation signals

- The number of functional groups passing the ΔSCScore < α^n threshold equals 131 at recursion level n = 2 and 97 at n = 3 with α = 1.2
- scscore_filter_results.csv contains one row per functional group (204 groups) with a ΔSCScore value and pass flags for both levels
- Every ΔSCScore is computed relative to the SCScore baseline of the cathinone template before substitution
- The final enumeration-and-filtering output totals approximately 422 million compounds

## Limitations

- SCScore filtering depends on a pretrained model whose version is not stated in the paper, and threshold α = 1.2 was tuned for cathinone-template derivatives
- The number of enumeration levels has a similar (unquantified) impact on results, so deeper recursion changes the retained functional-group set
- RDKit must be built from C++ source with the Python package installed from conda; the pipeline requires GNU g++-10+, CMake 3.16.0+, and Python 3.6.9+

## Evidence

- [methods] We then calculated the average synthetic complexity score of the resulting compounds using SCScore.: "We then calculated the average synthetic complexity score of the resulting compounds using SCScore."
- [methods] retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [readme] PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups.: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [methods] We further introduced 57 drug-like criteria according to Virshup et al.: "We further introduced 57 drug-like criteria according to Virshup et al."
- [methods] the process of enumeration and filtering resulted in a total of approximately 422 million compounds.: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
