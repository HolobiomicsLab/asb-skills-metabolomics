---
name: multiview-contrastive-learning-for-metabolite-annotation
description: Use when ranking candidate molecular structures for MS/MS query spectra
  on MassSpecGym-style data, particularly when multiple spectra per molecule are available
  to build a consensus-spectrum view.
when_to_use_negative:
- The task is de novo molecule generation or spectrum simulation (the other MassSpecGym
  challenges) — MVP only ranks a provided set of candidate structures given a query
  spectrum.
- You need a training objective that directly optimizes candidate ranking — the paper
  notes losses that prioritize ranking molecular candidates are not currently included
  in MVP.
- No candidate structure list exists for the query spectra — MVP annotates by ranking
  provided candidates (candidates_pth), not by generating structures.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: MVP
  role: 'The multiview contrastive framework being trained: supplies the four encoders,
    the CMC-style pairwise contrastive loss, and the train.py/test.py entry points'
  repo: github.com/HassounLab/MVP
- name: MassSpecGym
  role: Benchmark dataset supplying the spectra table (data.tsv) and candidate molecule
    files used for training and retrieval evaluation
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: RDKit
  role: Computes Morgan fingerprints for the fingerprint (Zfp) view
- name: DGL
  role: GCN encoder over the molecular graph (Zmol) view
- name: PyTorch Geometric
  role: Alternative graph library for the molecular graph GCN encoder
- name: PyTorch
  role: Implements the MLP fingerprint encoder and the transformer encoders over individual
    and consensus spectra
  repo: https://github.com/pytorch/pytorch
- name: Lightning
  role: Runs the end-to-end training loop and logs the per-epoch contrastive training
    loss
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: MIST
  role: Source of the formula-based peak labels underlying the formSpec subformula
    views; also an inverse-annotation baseline for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:b0fbbab0cfcff50fd38a0b5f073c369f801beb3abb4680bae7201e5aff842273
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGL
  - Lightning
  - MIST
  - MVP
  - MassSpecGym
  - PyTorch
  - PyTorch Geometric
  - RDKit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# multiview-contrastive-learning-for-metabolite-annotation

## Summary

Train and apply MVP (MultiView Projection), a contrastive framework that jointly aligns four views — molecular graph (Zmol), Morgan fingerprint (Zfp), spectrum (Zs), and consensus spectrum (Zcs) — in one joint embedding space to rank molecular candidates for MS/MS spectra on MassSpecGym. Joint multi-view learning improves candidate ranking over approaches that concatenate views or use them as auxiliary-task targets.

## When to use

Use when ranking candidate molecular structures for MS/MS query spectra on MassSpecGym-style data, particularly when multiple spectra per molecule are available to build a consensus-spectrum view.

## When NOT to use

- The task is de novo molecule generation or spectrum simulation (the other MassSpecGym challenges) — MVP only ranks a provided set of candidate structures given a query spectrum.
- You need a training objective that directly optimizes candidate ranking — the paper notes losses that prioritize ranking molecular candidates are not currently included in MVP.
- No candidate structure list exists for the query spectra — MVP annotates by ranking provided candidates (candidates_pth), not by generating structures.

## Inputs

- MassSpecGym spectra table (data.tsv) with MS/MS spectra, precursor masses and formulae
- Candidate molecule list (candidates_mass.json or formula-based candidates file)
- Subformula label directory produced by assign_subformulae.py --max-formulae 60 (required for formSpec)
- params_formSpec.yaml or params_binnedSpec.yaml configuration with dataset and candidates paths

## Outputs

- trained_model.pt — weights of the four encoders (GCN, fingerprint MLP, spectrum and consensus-spectrum transformers) in a joint embedding space
- Per-spectrum candidate ranking result files mapping each query spectrum to ranked molecular structures (demo in notebooks/demo.ipynb)
- Per-epoch four-view contrastive training loss curve logged by Lightning (training_loss.png)
- Embeddings Zmol, Zfp, Zs, Zcs for molecules and query spectra

## How to apply

Download the MassSpecGym spectra table (data.tsv) and candidate files (candidates_mass.json) from pluskal-lab/MassSpecGym, then preprocess via the formSpec path: run subformula_assign/assign_subformulae.py with --max-formulae 60 to compute per-peak subformula labels, followed by data_preprocess.py --spec_type formSpec to emit fingerprint and consensus-spectrum files (or --spec_type binnedSpec to skip subformula assignment). Point params_formSpec.yaml (or params_binnedSpec.yaml) at the preprocessed dataset and candidates paths. Instantiate the four encoders — a GCN (DGL/PyTorch Geometric) over molecular graphs, an MLP over RDKit Morgan fingerprints, and PyTorch transformer encoders over individual and consensus spectra — then launch end-to-end training with 'python train.py --param_pth params_formSpec.yaml'; Lightning logs the per-epoch loss and weights are saved to trained_model.pt. The objective is a CMC-style pairwise mutual-information contrastive loss between view embeddings (Zmol, Zfp, Zs, Zcs), justified because learning all views jointly captures complementary and shared information while mitigating view-specific artifacts. At inference, rank candidates with the consensus-spectrum (mol-cs) view, which on MassSpecGym reaches 35.99% rank@1 by mass and 13.96% by formula, outperforming rank aggregation over individual spectra (26.37% / 11.10% rank@1).

## Related tools

- **MVP** (The multiview contrastive framework being trained: supplies the four encoders, the CMC-style pairwise contrastive loss, and the train.py/test.py entry points) — github.com/HassounLab/MVP
- **MassSpecGym** (Benchmark dataset supplying the spectra table (data.tsv) and candidate molecule files used for training and retrieval evaluation) — https://github.com/pluskal-lab/MassSpecGym
- **RDKit** (Computes Morgan fingerprints for the fingerprint (Zfp) view)
- **DGL** (GCN encoder over the molecular graph (Zmol) view)
- **PyTorch Geometric** (Alternative graph library for the molecular graph GCN encoder)
- **PyTorch** (Implements the MLP fingerprint encoder and the transformer encoders over individual and consensus spectra) — https://github.com/pytorch/pytorch
- **Lightning** (Runs the end-to-end training loop and logs the per-epoch contrastive training loss) — https://github.com/Lightning-AI/pytorch-lightning
- **MIST** (Source of the formula-based peak labels underlying the formSpec subformula views; also an inverse-annotation baseline for comparison)

## Examples

```
python train.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Lightning-logged per-epoch training loss, plotted to training_loss.png, decreases over epochs
- trained_model.pt loads and embeds all four views (Zmol, Zfp, Zs, Zcs) into one joint embedding space
- Consensus-spectrum (mol-cs) retrieval on MassSpecGym reproduces approximately 35.99% rank@1 by mass and 13.96% rank@1 by formula
- Individual-spectrum ranking lands near 26.37% / 11.10% rank@1 (mass / formula), i.e., clearly below the consensus view
- By-mass rank@1 exceeds the MIST baseline by ~145.83% and the JESTR baseline by ~130.71% relative

## Limitations

- Preprocessed MassSpecGym files cannot be downloaded directly — the README's wget URLs are empty, so local preprocessing via assign_subformulae.py + data_preprocess.py is required
- Losses that prioritize ranking molecular candidates are not currently included in MVP; representation quality could benefit from more accurate peak labeling and additional views
- Ranked annotations are putative and require experimental validation

## Evidence

- [abstract] We present MultiView Projection (MVP), a novel framework for learning a joint embedding space between molecules and spectra: "We present MultiView Projection (MVP), a novel framework for learning a joint embedding space between molecules and spectra"
- [abstract] Unlike prior approaches that consider multiple views via concatenation or as targets of auxiliary tasks, MVP learns from all views jointly, resulting in improved molecular candidate ranking.: "Unlike prior approaches that consider multiple views via concatenation or as targets of auxiliary tasks, MVP learns from all views jointly, resulting in improved molecular candidate ranking."
- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [intro] for candidates by mass, MVP outperforms MIST by 145.83% and JESTR by 130.71% for rank@1.: "for candidates by mass, MVP outperforms MIST by 145.83% and JESTR by 130.71% for rank@1."
- [results] MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula: "MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula"
- [discussion] including losses that prioritize ranking molecular candidates could improve ranking performance: "including losses that prioritize ranking molecular candidates could improve ranking performance"
- [readme] This repository provides the implementation of MultiView Projection (MVP). MVP can be used to rank a set of molecular candidates given a spectrum.: "This repository provides the implementation of MultiView Projection (MVP). MVP can be used to rank a set of molecular candidates given a spectrum."
- [readme] Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym). Follow their instruction to download the spectra and candidate dataset.: "Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym). Follow their instruction to download the spectra and candidate dataset."
