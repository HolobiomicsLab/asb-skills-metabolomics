---
name: tanimoto-nearest-neighbor-similarity-vs-pubchem-controls
description: 'Decision support for evaluating whether generated SMILES resemble known
  metabolites: use when a generative chemical model''s samples must be judged for
  metabolite-likeness against matched PubChem controls.'
when_to_use_negative:
- When the goal is to assess novelty rather than metabolite-likeness — nearest-neighbor
  Tanimoto similarity vs PubChem controls measures proximity to the known metabolome,
  not how new the generated structures are.
- When generated molecules must be compared against arbitrary synthetic compounds
  rather than formula-matched PubChem controls — an unformula-matched control set
  would confound structural proximity with mass/formula differences.
- When the language model was trained on non-human metabolites (e.g. plant or bacterial
  metabolism) — the article notes its model was trained/evaluated only on human metabolites,
  so the metabolite-likeness reference set may not transfer.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3047
tools:
- name: DeepMet
  role: Chemical language model used to generate the candidate metabolite structures
    that are then evaluated for metabolite-likeness via nearest-neighbor Tanimoto
    similarity.
  repo: github.com/skinniderlab/clm
- name: RDKit
  role: Provides SMILES canonicalization, InChIKey generation, fingerprint computation,
    and Tanimoto similarity calculation for both generated molecules and PubChem controls.
- name: PyTorch
  role: Trains the RNN model whose sampled outputs are subsequently filtered and compared
    via Tanimoto similarity to PubChem controls.
- name: PubChem
  role: Source of formula-matched control molecules used as the structural reference
    set for nearest-neighbor Tanimoto similarity.
- name: Snakemake
  role: Orchestrates the end-to-end CLM workflow (preprocess, sampling, filtering,
    evaluation) in which this similarity step is embedded.
  repo: github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:66c4e0daba128b34d8644493e8675e7a1b5cc07ecfe27ba921beeaf934fb44a3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepMet
  - PubChem
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3047
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tanimoto-nearest-neighbor-similarity-vs-pubchem-controls

## Summary

Quantifies how structurally close a language model's generated molecules are to the known metabolome by comparing each generated molecule's nearest-neighbor Tanimoto similarity (RDKit fingerprints) against formula-matched PubChem control molecules, rather than against an unstructured chemical universe.

## When to use

Decision support for evaluating whether generated SMILES resemble known metabolites: use when a generative chemical model's samples must be judged for metabolite-likeness against matched PubChem controls.

## When NOT to use

- When the goal is to assess novelty rather than metabolite-likeness — nearest-neighbor Tanimoto similarity vs PubChem controls measures proximity to the known metabolome, not how new the generated structures are.
- When generated molecules must be compared against arbitrary synthetic compounds rather than formula-matched PubChem controls — an unformula-matched control set would confound structural proximity with mass/formula differences.
- When the language model was trained on non-human metabolites (e.g. plant or bacterial metabolism) — the article notes its model was trained/evaluated only on human metabolites, so the metabolite-likeness reference set may not transfer.

## Inputs

- Sampled SMILES from a trained DeepMet/CLM RNN model checkpoint (e.g. 500,000 samples per fold), canonicalized with RDKit and deduplicated against the fold's training InChIKeys
- Withheld test-fold metabolites (known metabolite SMILES, e.g. from HMDB) for the 'metabolite-like' reference set
- PubChem control molecules matched on molecular formula to each generated molecule
- RDKit fingerprint definitions (e.g. Morgan/circular fingerprints) used for Tanimoto comparison

## Outputs

- Per-molecule nearest-neighbor Tanimoto similarity values (generated vs withheld metabolites; generated vs formula-matched PubChem controls)
- Summary metrics written to metabolite_likeness_evaluation.csv (nearest-neighbor Tanimoto distribution, random-forest AUC ~0.48, BioTransformer recapitulation ~77.5%)
- Optional UMAP plot of chemical-space overlap between generated molecules and withheld metabolites

## How to apply

Sample molecules from a trained chemical language model (e.g. DeepMet/CLM, RNN trained per cross-validation fold via `train_models_RNN` in PyTorch) and canonicalize each SMILES with RDKit, removing any sample whose InChIKey matches the training file for that fold. Pull a set of control molecules from PubChem that match the molecular formula of each generated molecule, so the comparison is formula-stratified. Compute nearest-neighbor Tanimoto similarity (using RDKit fingerprints) between each generated molecule and the withheld metabolite set, and separately between each generated molecule and its formula-matched PubChem control set. The rationale is that structural proximity to known metabolites supports the claim that generated structures are metabolite-like rather than arbitrary synthetic compounds; write all metrics (e.g. to `metabolite_likeness_evaluation.csv`) together with the random forest AUC (~0.48 vs withheld metabolites) and BioTransformer one-step recapitulation (~77.5%) as joint evidence.

## Related tools

- **DeepMet** (Chemical language model used to generate the candidate metabolite structures that are then evaluated for metabolite-likeness via nearest-neighbor Tanimoto similarity.) — github.com/skinniderlab/clm
- **RDKit** (Provides SMILES canonicalization, InChIKey generation, fingerprint computation, and Tanimoto similarity calculation for both generated molecules and PubChem controls.)
- **PyTorch** (Trains the RNN model whose sampled outputs are subsequently filtered and compared via Tanimoto similarity to PubChem controls.)
- **PubChem** (Source of formula-matched control molecules used as the structural reference set for nearest-neighbor Tanimoto similarity.)
- **Snakemake** (Orchestrates the end-to-end CLM workflow (preprocess, sampling, filtering, evaluation) in which this similarity step is embedded.) — github.com/skinniderlab/CLM

## Examples

```
python -c "from clm.functions import similarity_to_controls; similarity_to_controls(sampled_smiles='sampled_fold1.csv', controls='pubchem_formula_matched.csv', fingerprint='rdkit', out='metabolite_likeness_evaluation.csv')"
```

## Evaluation signals

- Random forest classifier distinguishing filtered generated molecules from withheld metabolites should perform at chance level (AUC ≈ 0.48), confirming generated molecules are structurally indistinguishable from real metabolites.
- Nearest-neighbor Tanimoto similarities to formula-matched PubChem controls should show structural proximity of generated molecules to the known metabolome, not just chance overlap.
- BioTransformer one-step recapitulation should reach approximately 77.5% of generated molecules, confirming transformation-product consistency with known metabolite chemistry.
- Canonicalized sampled SMILES must be deduplicated against the fold's training InChIKeys so that 'metabolite-likeness' is not simply memorized training molecules.
- All metrics should be written to metabolite_likeness_evaluation.csv with a reproducible Snakemake-driven pipeline (e.g. `train_models_RNN` on one fold, sampling, filtering, evaluation).

## Limitations

- The DeepMet/CLM language model was trained and evaluated on the structures of human metabolites; applying the same metabolite-likeness evaluation to evolutionarily distant organisms (e.g. plant or bacterial metabolism) likely requires a bespoke model trained on that metabolome.
- The model's generative prior may be inherently limited in anticipating metabolites that arise from divergent, as-of-yet undiscovered biosynthetic routes, so high structural proximity to known metabolites may not capture truly novel chemistry.
- This similarity-based evaluation does not address the open problems of predicting MS/MS spectra or retention times from chemical structures, so metabolite-likeness alone does not guarantee that generated structures would produce matching analytical signals.

## Evidence

- [other] Computed nearest-neighbor Tanimoto similarity of each sampled molecule against formula-matched PubChem control molecules.: [evidence span withheld — anchor status: not_found]
- [other] Structural similarity to formula-matched controls was confirmed via random-forest AUC at chance level and BioTransformer recapitulation.: [evidence span withheld — anchor status: not_found]
- [readme] README documents that the CLM repository contains Python source code for training and evaluating chemical language models.: [evidence span withheld — anchor status: external_doc]
