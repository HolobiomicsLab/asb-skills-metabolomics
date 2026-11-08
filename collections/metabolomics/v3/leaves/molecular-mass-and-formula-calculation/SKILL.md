---
name: molecular-mass-and-formula-calculation
description: Use when you have a table of canonical SMILES (e.g. DeepMet-generated
  candidate metabolites) and need per-molecule calculated molecular mass and formula
  to enable m/z-based filtering or annotation of metabolomics peaks.
when_to_use_negative:
- Input SMILES have not yet been parsed/validated with RDKit and may include invalid
  structures — validate first, then annotate mass/formula
- You need experimental MS/MS spectra or retention times rather than calculated properties
  — the article notes predicting MS/MS spectra or retention times from chemical structures
  remain open problems
- Structures are already annotated with masses/formulas in the source table (e.g.
  the released 34.4-million-structure resource) — reuse rather than recompute
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RDKit
  role: Parses sampled SMILES, validates molecules, canonicalizes structures, and
    computes calculated molecular mass and formula
  repo: https://github.com/skinniderlab/CLM
- name: DeepMet
  role: Chemical language model whose autoregressively sampled SMILES are the structures
    annotated with mass and formula
  repo: https://github.com/skinniderlab/CLM
- name: Snakemake
  role: Orchestrates the end-end workflow (sampling, validation, canonicalization,
    mass/formula annotation, aggregation) via the included Snakefile
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:a867bce4c997fcbbf813977fa1b1bcd1af047c029e7a156f48018cf566792cf6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# molecular-mass-and-formula-calculation

## Summary

Compute the calculated molecular mass and molecular formula for each canonicalized SMILES so that generated candidate metabolites can be filtered and prioritized by mass-to-charge ratio against MS/MS signals. In the DeepMet workflow this step converts a table of valid canonical structures into a mass/formula-annotated structure-frequency resource.

## When to use

Use when you have a table of canonical SMILES (e.g. DeepMet-generated candidate metabolites) and need per-molecule calculated molecular mass and formula to enable m/z-based filtering or annotation of metabolomics peaks.

## When NOT to use

- Input SMILES have not yet been parsed/validated with RDKit and may include invalid structures — validate first, then annotate mass/formula
- You need experimental MS/MS spectra or retention times rather than calculated properties — the article notes predicting MS/MS spectra or retention times from chemical structures remain open problems
- Structures are already annotated with masses/formulas in the source table (e.g. the released 34.4-million-structure resource) — reuse rather than recompute

## Inputs

- Table of chemically valid, canonicalized SMILES from DeepMet sampling (per-molecule rows with sampling frequency)
- Trained DeepMet/CLM RNN model sampled structures (SMILES strings, 100 million per fold, 1 billion total)
- config.yaml specifying sample_mols and sample_seed values

## Outputs

- structure_frequencies.tsv with one row per unique canonical structure including calculated molecular mass, formula, InChIKey, and averaged sampling frequency
- Mass-annotated candidate-metabolite database suitable for m/z-based prioritization

## How to apply

First parse each sampled SMILES with RDKit and discard chemically invalid outputs, then canonicalize the valid SMILES. For each canonical structure, compute the calculated molecular mass and molecular formula from the RDKit molecule object (implicit-H-accurate monoisotopic or average mass, matching how DeepMet annotates its structure_frequency resource). Retain these annotations alongside the per-molecule sampling frequency and InChIKey, so downstream steps can filter the generated database on mass-to-charge ratio to prioritize structures most likely to account for a mass spectrometric signal. The resulting mass/formula-annotated table feeds the aggregate-frequency AUC evaluation (near-perfect AUC of 0.98 in cross-validation) and the in silico MS/MS library construction over anticipated metabolites.

## Related tools

- **RDKit** (Parses sampled SMILES, validates molecules, canonicalizes structures, and computes calculated molecular mass and formula) — https://github.com/skinniderlab/CLM
- **DeepMet** (Chemical language model whose autoregressively sampled SMILES are the structures annotated with mass and formula) — https://github.com/skinniderlab/CLM
- **Snakemake** (Orchestrates the end-end workflow (sampling, validation, canonicalization, mass/formula annotation, aggregation) via the included Snakefile) — https://github.com/skinniderlab/CLM

## Examples

```
from rdkit import Chem; from rdkit.Chem import Descriptors; mol = Chem.MolFromSmiles(canonical_smiles); mass = Descriptors.MolWt(mol); formula = rdMolDescriptors.CalcMolFormula(mol)
```

## Evaluation signals

- structure_frequencies.tsv contains one row per unique canonical structure with its averaged sampling frequency
- No training-set InChIKeys are present in the output table (training molecules were removed by InChIKey comparison)
- No chemically invalid SMILES are included in the output
- Every row carries a non-empty calculated molecular mass and molecular formula consistent with its canonical SMILES
- Calculated masses fall within the metabolite-like chemical space expected of HMDB-derived training data, enabling well-calibrated m/z prioritization

## Limitations

- Prioritization by sampling frequency and mass-to-charge ratio is trained/evaluated on human metabolites; evolutionarily distant applications (e.g. plant or bacterial metabolism) will likely require bespoke models
- The model has inherent limitations anticipating metabolites from divergent, as-of-yet undiscovered biosynthetic routes
- Mass-based prioritization is well-calibrated even without other analytical data, but more tightly integrating DeepMet with analytical data (e.g. MS/MS, retention time) remains unrealized

## Evidence

- [results] Canonicalize the valid SMILES and compute the calculated molecular mass and formula for each.: [evidence span withheld — anchor status: not_found]
