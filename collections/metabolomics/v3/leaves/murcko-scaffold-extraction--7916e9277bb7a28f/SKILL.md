---
name: murcko-scaffold-extraction
description: 'Use when evaluating de-novo structure recommendations from MS/MS spectra:
  to verify that generated SMILES recover the key molecular substructures (Murcko
  scaffolds and functional groups) of the ground-truth test compounds.'
when_to_use_negative:
- When the recommended SMILES fail RDKit sanitization or cannot be parsed, since HasSubstructMatch
  requires substructure-searchable molecules
- When evaluating retrieval-style spectrum-to-structure performance (e.g. Tanimoto
  similarity against known references) rather than de-novo substructure recovery
- When input spectra are not in the four csv format expected by Spec2Mol (pos/neg
  low/high NCE files with m/z and intensity columns)
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Parses and canonicalizes SMILES, extracts Bemis-Murcko scaffolds via the MurckoScaffold
    module, and performs HasSubstructMatch queries for scaffolds and functional-group
    SMARTS patterns
  repo: https://www.rdkit.org/
- name: PyTorch
  role: Implements the Spec2Mol encoder-decoder architecture that generates the recommended
    SMILES evaluated by this skill
  repo: github.com/KavrakiLab/Spec2Mol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:9aa064348099385238c29a4dc019c2657aee0544552b043dcc9f5fae13f8d908
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# murcko-scaffold-extraction

## Summary

A substructure-identification skill for evaluating whether Spec2Mol's recommended SMILES structures contain the Bemis-Murcko scaffolds and functional-group patterns of the ground-truth test molecules, using RDKit's MurckoScaffold and HasSubstructMatch modules.

## When to use

Use when evaluating de-novo structure recommendations from MS/MS spectra: to verify that generated SMILES recover the key molecular substructures (Murcko scaffolds and functional groups) of the ground-truth test compounds.

## When NOT to use

- When the recommended SMILES fail RDKit sanitization or cannot be parsed, since HasSubstructMatch requires substructure-searchable molecules
- When evaluating retrieval-style spectrum-to-structure performance (e.g. Tanimoto similarity against known references) rather than de-novo substructure recovery
- When input spectra are not in the four csv format expected by Spec2Mol (pos/neg low/high NCE files with m/z and intensity columns)

## Inputs

- Recommended SMILES strings produced by the Spec2Mol decoder from input MS/MS spectra (four csv files of m/z and intensity pairs: [M+H]+ 35% and 130% NCE, [M-H]- 35% and 130% NCE)
- Ground-truth SMILES structures of the test compounds

## Outputs

- Per-test-compound binary flags for scaffold presence and each functional-group SMARTS pattern
- Fraction of recommendations containing the ground-truth Murcko scaffold
- Fraction of recommendations containing each predefined functional group

## How to apply

Load the Spec2Mol recommended SMILES from the similarity evaluation stage together with the corresponding ground-truth test structures, and canonicalize all SMILES with RDKit, sanitizing the molecules so they are substructure-searchable. Apply RDKit's MurckoScaffold module to extract the Bemis-Murcko scaffold of each ground-truth molecule. Run RDKit HasSubstructMatch queries on each recommended SMILES to check for the presence of the ground-truth scaffold and of predefined functional-group SMARTS patterns. Tabulate the fraction of recommendations that contain the ground-truth scaffold and each functional group, storing per-compound counts across the test set. A nonzero substructure-recovery rate across the test set supports the paper's claim that Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum.

## Related tools

- **RDKit** (Parses and canonicalizes SMILES, extracts Bemis-Murcko scaffolds via the MurckoScaffold module, and performs HasSubstructMatch queries for scaffolds and functional-group SMARTS patterns) — https://www.rdkit.org/
- **PyTorch** (Implements the Spec2Mol encoder-decoder architecture that generates the recommended SMILES evaluated by this skill) — github.com/KavrakiLab/Spec2Mol

## Examples

```
python -c "from rdkit import Chem; from rdkit.Chem.Scaffolds import MurckoScaffold; mol=Chem.MolFromSmiles(smi); scaffold=MurckoScaffold.GetScaffoldForMol(mol)"
```

## Evaluation signals

- Nonzero substructure-recovery rate: a nonzero fraction of recommendations contain the ground-truth Murcko scaffold, consistent with the claim 'Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum'
- Every ground-truth and recommended SMILES canonicalizes and sanitizes successfully with RDKit before substructure queries
- Per-compound counts are tabulated for each predefined functional-group SMARTS pattern, covering the full test set
- The fraction of recommendations containing each functional group is reported alongside the scaffold-recovery fraction

## Limitations

- The recommended structures may not reproduce the full ground-truth molecule, so recovery is assessed at the level of scaffolds and functional groups rather than exact structures
- The spectra encoder is trained on the commercial NIST Tandem Mass Spectral Library 2020, which may limit generalization to spectra outside that distribution
- Nonzero recovery does not guarantee correct identification of every key substructure; recovery rates should be interpreted alongside similarity-based evaluation results

## Evidence

- [abstract] Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum: "Spec2Mol is able to identify the presence of key molecular substructures from its mass spectrum"
- [intro] Processing of the chemical data is based on the RDKit software: "Processing of the chemical data is based on the RDKit software"
- [intro] The implementation of the Spec2Mol architecture is based on the Pytorch library: "The implementation of the Spec2Mol architecture is based on the Pytorch library"
- [readme] The spectra encoder has been trained on the NIST Tandem Mass Spectral Library 2020: "The spectra encoder has been trained on the NIST Tandem Mass Spectral Library
2020 which is a commercial dataset"
- [readme] Each csv file has the m/z values in the first column and the intensity values in the second column: "Each csv file has the m/z values in the first column and the intensity values in the second
column"
