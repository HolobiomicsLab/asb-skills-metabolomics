---
name: mass-formula-candidate-retrieval
description: Use when you have MS/MS spectra plus a candidate molecule list (by mass
  or formula) and need to rank candidates for annotation, e.g. reproducing MassSpecGym
  rank@1 across mol-s, fp-s, mol-cs and fp-cs views.
when_to_use_negative:
- The task is de novo molecule generation (spectrum → structure) with no candidate
  list — retrieval requires a set of candidate molecules to rank
- You need a fair comparison against CFM-ID 4.0, whose evaluation additionally requires
  excluding spectra whose molecules appear in CFM-ID's training set and sampling three
  spectra (one per collision energy level) per test molecule
- Input spectra lack precursor mass or chemical formula information, so neither the
  by-mass nor the by-formula candidate condition can be defined
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: MVP
  role: Multiview contrastive model that scores and ranks molecular candidates given
    spectra; run via test.py with params_formSpec.yaml or params_binnedSpec.yaml
  repo: github.com/HassounLab/MVP
- name: massspecgym
  role: Benchmark package providing the spectra/candidate datasets, predefined splits,
    and the retrieval challenge metrics used to compute rank@1
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: Lightning
  role: Framework in which MassSpecGym's RetrievalMassSpecGymModel and DataModule
    for train/val/test splitting are implemented
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: PyTorch
  role: Underlying deep-learning framework for MassSpecDataset transforms (SpecTokenizer,
    MolFingerprinter) and MVP model execution
  repo: https://github.com/pytorch/pytorch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:5f4ed90249518be8f9903beff94665d77f83b4df41fad316aaf6496f904bd95d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lightning
  - MVP
  - PyTorch
  - massspecgym
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mass-formula-candidate-retrieval

## Summary

Ranks molecular candidate structures against query MS/MS spectra using the MVP multiview contrastive model, either constrained by precursor mass or by chemical formula, on the MassSpecGym retrieval benchmark.

## When to use

Use when you have MS/MS spectra plus a candidate molecule list (by mass or formula) and need to rank candidates for annotation, e.g. reproducing MassSpecGym rank@1 across mol-s, fp-s, mol-cs and fp-cs views.

## When NOT to use

- The task is de novo molecule generation (spectrum → structure) with no candidate list — retrieval requires a set of candidate molecules to rank
- You need a fair comparison against CFM-ID 4.0, whose evaluation additionally requires excluding spectra whose molecules appear in CFM-ID's training set and sampling three spectra (one per collision energy level) per test molecule
- Input spectra lack precursor mass or chemical formula information, so neither the by-mass nor the by-formula candidate condition can be defined

## Inputs

- MS/MS spectra TSV (e.g. data.tsv) with precursor mass or formula
- Candidate molecule lists as JSON (candidates_mass.json / candidates_formula.json)
- MassSpecGym spectra and candidate datasets (Hugging Face / pluskal-lab repo)
- params_formSpec.yaml or params_binnedSpec.yaml configuration with dataset paths
- Pretrained MVP checkpoint trained on MassSpecGym

## Outputs

- Ranked candidate lists per query spectrum for each of the four ranking views (mol-s, fp-s, mol-cs, fp-cs)
- rank@1 (and other retrieval metrics) per ranking view for candidates by mass and by formula
- Preprocessing artifacts: subformula label assignments, fingerprint files, and consensus spectra files

## How to apply

Retrieve candidate structures within a 0.01 Da tolerance from reference databases (FoodDB, HMDB, MassSpecGym, NIST23, GNPS Multiplex) or use the MassSpecGym-provided candidate lists for the by-mass and by-formula conditions. Preprocess spectra with data_preprocess.py choosing --spec_type formSpec (formula-annotated spectra) or binnedSpec; for formSpec first assign subformula labels with subformula_assign/assign_subformulae.py using --max-formulae 60, which also generates fingerprint and consensus spectra files. Point params_formSpec.yaml or params_binnedSpec.yaml at the preprocessed dataset paths and run the pretrained MVP model via test.py, which scores candidates under four ranking views: molecular-graph vs spectrum (mol-s), fingerprint vs spectrum (fp-s), molecular-graph vs consensus spectrum (mol-cs), and fingerprint vs consensus spectrum (fp-cs). Compute rank@1 per view for both candidate conditions; the consensus-spectrum views should outperform the individual-spectrum views (reported 35.99%/13.96% rank@1 by mass/formula for mol-cs vs 26.37%/11.10% for mol-s). For downstream annotation reporting, optionally focus on results with a molecule–spectra score > 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores.

## Related tools

- **MVP** (Multiview contrastive model that scores and ranks molecular candidates given spectra; run via test.py with params_formSpec.yaml or params_binnedSpec.yaml) — github.com/HassounLab/MVP
- **massspecgym** (Benchmark package providing the spectra/candidate datasets, predefined splits, and the retrieval challenge metrics used to compute rank@1) — https://github.com/pluskal-lab/MassSpecGym
- **Lightning** (Framework in which MassSpecGym's RetrievalMassSpecGymModel and DataModule for train/val/test splitting are implemented) — https://github.com/Lightning-AI/pytorch-lightning
- **PyTorch** (Underlying deep-learning framework for MassSpecDataset transforms (SpecTokenizer, MolFingerprinter) and MVP model execution) — https://github.com/pytorch/pytorch

## Examples

```
python data_preprocess.py --spec_type formSpec --dataset_pth ../data/sample/data.tsv --candidates_pth ../data/sample/candidates_mass.json --output_dir ../data/sample/ ; python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Compiled benchmark table reproduces mol-cs rank@1 of 35.99% (candidates by mass) and 13.96% (candidates by formula), and mol-s rank@1 of 26.37% and 11.10%, within acceptable numeric tolerance
- The consensus-spectrum views (mol-cs, fp-cs) outperform the individual-spectrum views (mol-s, fp-s) on the MassSpecGym benchmark, consistent with consensus spectra beating rank aggregation of constituent spectra
- All candidate structures were retrieved within a 0.01 Da tolerance from the reference databases, or candidate lists match the MassSpecGym-provided by-mass/by-formula conditions
- Preprocessing ran with --max-formulae 60 for formSpec, producing subformula, fingerprint, and consensus spectra files before test.py was invoked
- Result files match the structure shown in notebooks/demo.ipynb

## Limitations

- The README's preprocessed MassSpecGym download URLs are empty (the wget commands have no targets), so the full preprocessing pipeline must be run locally
- Losses that prioritize ranking molecular candidates are not currently included in MVP; adding them could improve ranking performance
- Representation quality could benefit from more accurate peak labeling and additional views
- Reported MVP annotations are putative and require experimental validation

## Evidence

- [readme] You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates.: "You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates."
- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [results] Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
- [readme] python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60: "python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60"
- [results] we focus on results with a molecule–spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores: "we focus on results with a molecule–spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores"
