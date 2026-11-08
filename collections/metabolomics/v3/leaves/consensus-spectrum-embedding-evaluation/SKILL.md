---
name: consensus-spectrum-embedding-evaluation
description: Use when benchmarking metabolite annotation models on MassSpecGym and
  you need to compare aggregate-then-rank (consensus spectrum) versus rank-then-aggregate
  (individual spectra) candidate retrieval, reporting rank@1 per ranking view and
  candidate condition.
when_to_use_negative:
- You are doing de novo molecule generation or spectrum simulation challenges of MassSpecGym
  rather than the molecule retrieval challenge
- Your goal is annotation of an individual study's spectra against FoodDB/HMDB/NIST23/GNPS
  candidate sets with 0.01 Da tolerance, not the standardized MassSpecGym retrieval
  benchmark
- You want to evaluate baselines like MIST, ESP, JESTR, or CFM-ID 4.0 that do not
  expose consensus-spectrum ranking views; those require their own evaluation protocols
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MVP
  role: Multiview contrastive model providing the four ranking views (mol-s, fp-s,
    mol-cs, fp-cs), preprocessing scripts, and pretrained MassSpecGym checkpoint used
    for candidate ranking
  repo: https://github.com/HassounLab/MVP
- name: MassSpecGym
  role: Benchmark dataset and retrieval challenge definition (spectrum to ranked candidate
    list, with bonus chemical formulae condition) supplying spectra, candidates, splits,
    and metrics
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: PyTorch
  role: Deep learning framework underlying MVP model training and evaluation
- name: Lightning
  role: Training/evaluation framework used by MVP and MassSpecGym's MassSpecDataModule
    / LightningModule interfaces
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:4be4df36587256a8233de55673059e1156116d2eb29e0681c7c1e9f01367af1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - Lightning
  - MVP
  - MassSpecGym
  - PyTorch
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# consensus-spectrum-embedding-evaluation

## Summary

Evaluates cross-modal candidate ranking using a consensus-spectrum embedding view (mol-cs / fp-cs) versus individual-spectrum ranking (mol-s / fp-s) on the MassSpecGym retrieval benchmark, quantified by rank@1 for candidates retrieved by mass and by formula.

## When to use

Use when benchmarking metabolite annotation models on MassSpecGym and you need to compare aggregate-then-rank (consensus spectrum) versus rank-then-aggregate (individual spectra) candidate retrieval, reporting rank@1 per ranking view and candidate condition.

## When NOT to use

- You are doing de novo molecule generation or spectrum simulation challenges of MassSpecGym rather than the molecule retrieval challenge
- Your goal is annotation of an individual study's spectra against FoodDB/HMDB/NIST23/GNPS candidate sets with 0.01 Da tolerance, not the standardized MassSpecGym retrieval benchmark
- You want to evaluate baselines like MIST, ESP, JESTR, or CFM-ID 4.0 that do not expose consensus-spectrum ranking views; those require their own evaluation protocols

## Inputs

- MassSpecGym spectra dataset (downloaded per pluskal-lab/MassSpecGym instructions)
- MassSpecGym candidate datasets (candidates by mass and candidates by formula)
- Preprocessed fingerprint and consensus spectra files from MVP data_preprocess.py
- Pretrained MVP checkpoint trained on MassSpecGym
- params_formSpec.yaml / params_binnedSpec.yaml configuration files with dataset paths

## Outputs

- Ranked candidate molecule lists per query spectrum for each ranking view (mol-s, fp-s, mol-cs, fp-cs)
- rank@1 values per ranking view for candidates by mass and by formula
- Aggregated benchmark table comparing consensus-spectrum vs individual-spectrum retrieval

## How to apply

Download the MassSpecGym spectra and candidate datasets following pluskal-lab/MassSpecGym, then preprocess with MVP's data_preprocess.py for each spec_type (formSpec and binnedSpec), computing subformula labels via subformula_assign/assign_subformulae.py with --max-formulae 60 and generating fingerprint and consensus spectra files. Point params_formSpec.yaml or params_binnedSpec.yaml at the preprocessed MassSpecGym paths and run test.py with the pretrained model to rank molecular candidates for each of the four ranking views: mol-s, fp-s, mol-cs, fp-cs. Compute rank@1 per view under both candidate conditions (by mass and by formula) and aggregate into a single table. The rationale is that annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation, so the consensus view (mol-cs) should be the headline metric. Correct application reproduces mol-cs rank@1 of 35.99% (by mass) and 13.96% (by formula), and mol-s rank@1 of 26.37% (by mass) and 11.10% (by formula), within acceptable numeric tolerance.

## Related tools

- **MVP** (Multiview contrastive model providing the four ranking views (mol-s, fp-s, mol-cs, fp-cs), preprocessing scripts, and pretrained MassSpecGym checkpoint used for candidate ranking) — https://github.com/HassounLab/MVP
- **MassSpecGym** (Benchmark dataset and retrieval challenge definition (spectrum to ranked candidate list, with bonus chemical formulae condition) supplying spectra, candidates, splits, and metrics) — https://github.com/pluskal-lab/MassSpecGym
- **PyTorch** (Deep learning framework underlying MVP model training and evaluation)
- **Lightning** (Training/evaluation framework used by MVP and MassSpecGym's MassSpecDataModule / LightningModule interfaces)

## Examples

```
python subformula_assign/assign_subformulae.py --spec-files data/msgym/data.tsv --output-dir data/msgym/subformulae_default --max-formulae 60 --labels-file data/msgym/data.tsv; python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Reproduced rank@1 values match: mol-cs 35.99% (candidates by mass) and 13.96% (by formula); mol-s 26.37% (by mass) and 11.10% (by formula), within acceptable numeric tolerance
- Consensus-spectrum views (mol-cs, fp-cs) outperform the corresponding individual-spectrum views (mol-s, fp-s) on both candidate conditions
- All four ranking views (mol-s, fp-s, mol-cs, fp-cs) produce rankings for both the by-mass and by-formula candidate sets, yielding eight rank@1 cells in the table
- Preprocessing outputs (subformula labels, fingerprint files, consensus spectra files) exist for both formSpec and binnedSpec spec_types before running test.py
- Data splits and evaluation metrics follow the predefined MassSpecGym retrieval challenge rather than ad hoc splits

## Limitations

- Preprocessed MassSpecGym download URLs in the MVP README are empty (wget commands have no targets), so preprocessing must be run locally
- Losses that prioritize ranking molecular candidates are not currently included in MVP, so ranking performance may understate embedding quality
- Representation quality could benefit from more accurate peak labeling and additional views; peak labels come from MIST's formula annotation algorithm, which assumes every peak is charged with the adduct
- MVP with mol-cs is the intended headline view; results with individual spectra reflect the weaker rank-then-aggregate strategy

## Evidence

- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [abstract] individual spectra rank@1: "achieving 26.37% and 11.10% rank@1 for candidates by mass and formula, respectively"
- [abstract] consensus vs aggregation: "annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation"
- [readme] MVP preprocessing command: "python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60"
- [readme] pretrained model invocation: "python test.py --param_pth params_formSpec.yaml"
- [other] retrieval challenge definition: [evidence span withheld — anchor status: not_found]
