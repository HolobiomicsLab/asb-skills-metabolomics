---
name: massspec-candidate-ranking
description: Use when you have MS/MS spectra plus a candidate molecule list (by mass
  or formula) and need a ranked list of structures per spectrum, or when reproducing
  MVP's MassSpecGym rank@1 benchmark.
when_to_use_negative:
- The task is de novo molecule generation or spectrum simulation — this skill only
  scores and ranks an already-retrieved candidate set.
- You need to build candidate sets yourself; candidate retrieval within a 0.01 Da
  tolerance from databases (FoodDB, HMDB, NIST23, GNPS Multiplex) is a separate upstream
  step.
- Spectra come from an instrument/protocol far outside MassSpecGym's training distribution
  without retraining — the pretrained model is trained on MassSpecGym only.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MVP
  role: Multiview contrastive model used to rank molecular candidates given a spectrum
    via its mol-s, fp-s, mol-cs, and fp-cs ranking views
  repo: github.com/HassounLab/MVP
- name: massspecgym
  role: Provides the benchmark spectra and candidate datasets the pretrained MVP model
    is trained and evaluated on
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: PyTorch
  role: Deep learning backend for running the pretrained MVP model inference
- name: Lightning
  role: Training/evaluation framework underlying MVP's test.py pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:30e74ec3a80a873d9577ecf0eedf5ad323a38014cf3febb6ffaa2b299b090482
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lightning
  - MVP
  - PyTorch
  - massspecgym
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# massspec-candidate-ranking

## Summary

Rank a set of molecular candidates against an MS/MS query spectrum (or consensus spectrum) using MVP's pretrained multiview embedding model, reporting rank@k retrieval performance on the MassSpecGym benchmark.

## When to use

Use when you have MS/MS spectra plus a candidate molecule list (by mass or formula) and need a ranked list of structures per spectrum, or when reproducing MVP's MassSpecGym rank@1 benchmark.

## When NOT to use

- The task is de novo molecule generation or spectrum simulation — this skill only scores and ranks an already-retrieved candidate set.
- You need to build candidate sets yourself; candidate retrieval within a 0.01 Da tolerance from databases (FoodDB, HMDB, NIST23, GNPS Multiplex) is a separate upstream step.
- Spectra come from an instrument/protocol far outside MassSpecGym's training distribution without retraining — the pretrained model is trained on MassSpecGym only.

## Inputs

- MS/MS spectra file (e.g. data.tsv) with collision-energy spectra per molecule
- Candidate molecular structures JSON (e.g. candidates_mass.json) retrieved by mass or formula
- Pretrained MVP model checkpoint (trained on MassSpecGym)
- params_formSpec.yaml or params_binnedSpec.yaml configuration with dataset paths

## Outputs

- Ranked list of molecular candidates per query spectrum under each ranking view (mol-s, fp-s, mol-cs, fp-cs)
- Rank@1 (and related retrieval metrics) per ranking view for candidates-by-mass and candidates-by-formula conditions
- Benchmark table aggregating rank@1 across views and candidate conditions

## How to apply

Preprocess spectra with MVP's data_preprocess.py: for the formSpec spec_type, first assign subformula labels via subformula_assign/assign_subformulae.py with --max-formulae 60, then generate fingerprint and consensus spectra files; for binnedSpec, run data_preprocess.py directly. Point the params_formSpec.yaml or params_binnedSpec.yaml configuration at your preprocessed dataset and candidate JSON (e.g. candidates_mass.json), then run test.py to load the pretrained model trained on MassSpecGym and score candidates under four ranking views: mol-s and fp-s (individual spectrum) and mol-cs and fp-cs (consensus spectrum). The consensus-spectrum view is preferred — on MassSpecGym it reaches 35.99% rank@1 by mass versus 26.37% for individual spectra. Validate by comparing computed rank@1 against the reported values: mol-cs 35.99% (mass) and 13.96% (formula); mol-s 26.37% (mass) and 11.10% (formula).

## Related tools

- **MVP** (Multiview contrastive model used to rank molecular candidates given a spectrum via its mol-s, fp-s, mol-cs, and fp-cs ranking views) — github.com/HassounLab/MVP
- **massspecgym** (Provides the benchmark spectra and candidate datasets the pretrained MVP model is trained and evaluated on) — https://github.com/pluskal-lab/MassSpecGym
- **PyTorch** (Deep learning backend for running the pretrained MVP model inference)
- **Lightning** (Training/evaluation framework underlying MVP's test.py pipeline)

## Examples

```
python subformula_assign/assign_subformulae.py --spec-files data.tsv --output-dir subformulae_default --max-formulae 60 --labels-file data.tsv; python data_preprocess.py --spec_type formSpec --dataset_pth data.tsv --candidates_pth candidates_mass.json --subformula_dir_pth subformulae_default/ --output_dir ./; python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Computed rank@1 matches reported values within tolerance: mol-cs 35.99% (mass) / 13.96% (formula); mol-s 26.37% (mass) / 11.10% (formula)
- Preprocessing artifacts (subformula labels, fingerprint files, consensus spectra files) exist in the output directory before test.py runs
- params yaml correctly resolves the preprocessed dataset and candidates paths so test.py completes without file-not-found errors
- demo notebook (notebooks/demo.ipynb) loads and inspects the produced result files without schema errors

## Limitations

- Reported annotations/putative candidate rankings require experimental validation.
- Losses that prioritize ranking molecular candidates are not currently included in MVP.
- Representation quality could benefit from more accurate peak labeling and additional views.
- Rank-aggregation comparison limited to test molecules with exactly three spectra (1,531 cases).
- Preprocessed MassSpecGym download URLs are empty in the README; preprocessing yourself may be required.

## Evidence

- [abstract] rank@1 by mass/formula per view: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [abstract] individual-spectrum rank@1: "achieving 26.37% and 11.10% rank@1 for candidates by mass and formula, respectively"
- [readme] preprocessing command: "python data_preprocess.py --spec_type formSpec --dataset_pth ../data/sample/data.tsv --candidates_pth  ../data/sample/candidates_mass.json"
- [readme] subformula assignment parameter: "python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60"
- [readme] evaluation entry point: "python test.py --param_pth params_formSpec.yaml"
- [readme] pretrained model scope: "You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates."
- [abstract] consensus spectrum advantage: "annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation"
- [results] candidate retrieval step: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
