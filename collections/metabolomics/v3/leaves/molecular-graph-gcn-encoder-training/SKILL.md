---
name: molecular-graph-gcn-encoder-training
description: Use when you must jointly embed molecular structures (molecular graphs
  + Morgan fingerprints) with MS/MS and consensus spectra into one contrastive embedding
  space for candidate ranking on MassSpecGym-style retrieval tasks.
when_to_use_negative:
- When you only need inference-time candidate ranking — use MVP's pretrained model
  path (python test.py --param_pth params_formSpec.yaml) instead of training from
  scratch
- When the task is de novo molecule generation or spectrum simulation rather than
  the MassSpecGym molecule retrieval challenge
- When only a single spectral view is available (no molecular graph or fingerprint
  candidates), since the joint four-view contrastive objective requires all views
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MVP
  role: Multiview contrastive framework providing the training objective that aligns
    the four encoders in one joint embedding space
  repo: github.com/HassounLab/MVP
- name: DGL
  role: Key package for building the GCN encoder over molecular graphs
- name: PyTorch Geometric
  role: Alternative GNN library for the molecular-graph encoder
- name: RDKit
  role: Computes Morgan fingerprints fed to the fingerprint MLP encoder
- name: Lightning
  role: Training loop and per-epoch loss logging for the contrastive model
- name: massspecgym
  role: Key package providing the MassSpecGym dataset, splits, and retrieval evaluation
    metrics
  repo: https://github.com/pluskal-lab/MassSpecGym
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:e1331484236b3e7bc9884309197e4ec8bcabd71b5bdaf205ca6efdac9c22b94b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGL
  - Lightning
  - MVP
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - massspecgym
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# molecular-graph-gcn-encoder-training

## Summary

Trains a GCN encoder over molecular graphs as one of four jointly aligned encoders (graph, fingerprint, spectrum, consensus spectrum) in MVP's multiview contrastive framework on MassSpecGym, producing a shared embedding space for molecule retrieval from MS/MS spectra.

## When to use

Use when you must jointly embed molecular structures (molecular graphs + Morgan fingerprints) with MS/MS and consensus spectra into one contrastive embedding space for candidate ranking on MassSpecGym-style retrieval tasks.

## When NOT to use

- When you only need inference-time candidate ranking — use MVP's pretrained model path (python test.py --param_pth params_formSpec.yaml) instead of training from scratch
- When the task is de novo molecule generation or spectrum simulation rather than the MassSpecGym molecule retrieval challenge
- When only a single spectral view is available (no molecular graph or fingerprint candidates), since the joint four-view contrastive objective requires all views

## Inputs

- MassSpecGym spectra table (data.tsv)
- MassSpecGym candidate molecules file (candidates_mass.json)
- Subformula labels directory (from assign_subformulae.py, --max-formulae 60)
- params_formSpec.yaml (or params_binnedSpec.yaml) configuration with dataset and candidates paths

## Outputs

- trained_model.pt checkpoint embedding all four views (graph, fingerprint, spectrum, consensus spectrum)
- training_loss.png / Lightning per-epoch training loss log
- Molecule and spectrum embeddings in the shared joint space for candidate ranking

## How to apply

First preprocess MassSpecGym via the formSpec path: run subformula_assign/assign_subformulae.py with --max-formulae 60 to compute subformula peak labels, then data_preprocess.py --spec_type formSpec to generate fingerprint and consensus-spectrum files (binnedSpec skips subformula assignment). Point params_formSpec.yaml at the preprocessed dataset and candidates files (data.tsv, candidates_mass.json). Instantiate four encoders: a GCN (DGL or PyTorch Geometric) over molecular graphs, an MLP over RDKit Morgan fingerprints, and transformer encoders over individual and consensus spectra. Train end-to-end with the CMC-style pairwise contrastive loss that aligns all four views in one joint embedding space — this joint multi-view objective, unlike concatenation or auxiliary-task approaches, is what improves molecular candidate ranking. Save weights to trained_model.pt and log per-epoch loss via Lightning; validate against reported MassSpecGym retrieval of 35.99% rank@1 by mass and 13.96% by formula using the consensus view.

## Related tools

- **MVP** (Multiview contrastive framework providing the training objective that aligns the four encoders in one joint embedding space) — github.com/HassounLab/MVP
- **DGL** (Key package for building the GCN encoder over molecular graphs)
- **PyTorch Geometric** (Alternative GNN library for the molecular-graph encoder)
- **RDKit** (Computes Morgan fingerprints fed to the fingerprint MLP encoder)
- **Lightning** (Training loop and per-epoch loss logging for the contrastive model)
- **massspecgym** (Key package providing the MassSpecGym dataset, splits, and retrieval evaluation metrics) — https://github.com/pluskal-lab/MassSpecGym

## Examples

```
python train.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- training_loss.png shows the four-view contrastive loss decreasing over epochs
- trained_model.pt loads and embeds all four views (molecular graph, fingerprint, spectrum, consensus spectrum)
- Consensus-view evaluation on MassSpecGym reproduces reported retrieval levels: 35.99% rank@1 by mass and 13.96% by formula
- Individual-spectrum ranking falls in the reported range of 26.37% rank@1 by mass and 11.10% by formula
- Environment resolves from environment.yml with the key packages (dgl, pytorch, rdkit, pytorch-geometric, massspecgym, lightning)

## Limitations

- Losses that prioritize ranking molecular candidates are not currently included in MVP, so ranking performance could be improved by such losses
- Representation quality could benefit from more accurate peak labeling and additional views
- Preprocessed MassSpecGym download URLs are empty in the README, so preprocessing must be run locally (the wget commands will not work as printed)

## Evidence

- [abstract] We present MultiView Projection (MVP), a novel framework for learning a joint embedding space between molecules and spectra: "We present MultiView Projection (MVP), a novel framework for learning a joint embedding space between molecules and spectra"
- [abstract] Unlike prior approaches that consider multiple views via concatenation or as targets of auxiliary tasks, MVP learns from all views jointly, resulting in improved molecular candidate ranking.: "Unlike prior approaches that consider multiple views via concatenation or as targets of auxiliary tasks, MVP learns from all views jointly, resulting in improved molecular candidate ranking."
- [readme] python data_preprocess.py --spec_type formSpec --dataset_pth ../data/sample/data.tsv --candidates_pth ../data/sample/candidates_mass.json --subformula_dir_pth ../data/sample/subformulae_default/ --output_dir ../data/sample/: "python data_preprocess.py --spec_type formSpec --dataset_pth ../data/sample/data.tsv --candidates_pth ../data/sample/candidates_mass.json --subformula_dir_pth ../data/sample/subformulae_default/"
- [readme] python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60: "python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60"
- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [discussion] including losses that prioritize ranking molecular candidates could improve ranking performance: "including losses that prioritize ranking molecular candidates could improve ranking performance"
