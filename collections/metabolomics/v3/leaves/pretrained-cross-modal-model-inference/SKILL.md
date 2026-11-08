---
name: pretrained-cross-modal-model-inference
description: Use when you have preprocessed consensus MS/MS spectra plus candidate
  molecule lists (retrieved by mass tolerance) and need to annotate spectra by ranking
  candidates with a pretrained joint-embedding model.
when_to_use_negative:
- Spectra and candidate structures have not been preprocessed into fingerprint and
  consensus spectra files via data_preprocess.py — run preprocessing first
- You need experimentally validated identifications rather than putative annotations
  — MVP outputs require experimental validation
- Only raw vendor profiles or unprocessed mzML are available — feature extraction
  and representative spectra selection with MZmine3 and formula/adduct assignment
  with SIRIUS v6.1.0 must precede inference
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MVP
  role: Pretrained multiview contrastive model used to rank molecular candidates given
    consensus spectra
  repo: https://github.com/HassounLab/MVP
- name: massspecgym
  role: Source of the dataset the pretrained model was trained on and a candidate
    database
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: rdkit
  role: Key dependency for molecular/fingerprint processing during inference
- name: dgl
  role: Key dependency for molecular graph representations fed to the model
- name: lightning
  role: Framework used by the test.py inference pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:1e38e34a36bbf3fd3d7abc0adc3e3fce037d0f7f1e0378f9cecdb56c81af143a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - MVP
  - dgl
  - lightning
  - massspecgym
  - rdkit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pretrained-cross-modal-model-inference

## Summary

Run a pretrained cross-modal (molecule–spectrum) embedding model such as MVP to rank molecular candidates for each consensus MS/MS spectrum, then filter ranked outputs to high-confidence putative annotations.

## When to use

Use when you have preprocessed consensus MS/MS spectra plus candidate molecule lists (retrieved by mass tolerance) and need to annotate spectra by ranking candidates with a pretrained joint-embedding model.

## When NOT to use

- Spectra and candidate structures have not been preprocessed into fingerprint and consensus spectra files via data_preprocess.py — run preprocessing first
- You need experimentally validated identifications rather than putative annotations — MVP outputs require experimental validation
- Only raw vendor profiles or unprocessed mzML are available — feature extraction and representative spectra selection with MZmine3 and formula/adduct assignment with SIRIUS v6.1.0 must precede inference

## Inputs

- Consensus spectra TSV (e.g. data.tsv with representative MS/MS spectra per feature)
- Candidate molecules JSON keyed by precursor mass (candidates_mass.json), retrieved within 0.01 Da tolerance
- Subformula label assignments for formSpec mode (from subformula_assign/assign_subformulae.py)
- Pretrained model checkpoint (trained on MassSpecGym) and params_formSpec.yaml or params_binnedSpec.yaml

## Outputs

- Ranked candidate lists with molecule–spectra scores per consensus spectrum
- High-confidence annotation set (score > 0.5, top-2 margin >= 0.01) with counts of annotated spectra and unique molecules
- Result files inspectable via notebooks/demo.ipynb

## How to apply

Preprocess the consensus spectra TSV and candidates_mass.json with data_preprocess.py (using --spec_type formSpec or binnedSpec) to generate fingerprint and consensus spectra files. Point params_formSpec.yaml or params_binnedSpec.yaml at the processed dataset and candidate paths, then run test.py with the chosen params file so each consensus spectrum is scored against candidates retrieved within a 0.01 Da tolerance from FoodDB, HMDB, MassSpecGym, NIST23, and the GNPS Multiplex library. Inspect ranked result files via notebooks/demo.ipynb and filter to high-confidence annotations: molecule–spectra scores greater than 0.5 with at least 0.01 margin between the top- and second-ranked candidates. The rationale is that the jointly trained multiview embedding space (molecular graphs, fingerprints, spectra, consensus spectra) lets the pretrained model score spectrum–molecule pairs without retraining, while the score/margin cutoffs reduce false positives among putative annotations.

## Related tools

- **MVP** (Pretrained multiview contrastive model used to rank molecular candidates given consensus spectra) — https://github.com/HassounLab/MVP
- **massspecgym** (Source of the dataset the pretrained model was trained on and a candidate database) — https://github.com/pluskal-lab/MassSpecGym
- **rdkit** (Key dependency for molecular/fingerprint processing during inference)
- **dgl** (Key dependency for molecular graph representations fed to the model)
- **lightning** (Framework used by the test.py inference pipeline)

## Examples

```
python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Preprocessing produces the expected fingerprint and consensus spectra files from the input TSV and candidates_mass.json
- Every consensus spectrum receives scores against the full candidate set retrieved within the 0.01 Da tolerance
- Filtered annotations satisfy both cutoffs: molecule–spectra score > 0.5 and top-2 margin >= 0.01
- On the cultured-vs-conventional meat study (positive ion mode), the pipeline reproduces 640 annotated spectra corresponding to 146 unique molecules
- Result files load and are interpretable in notebooks/demo.ipynb

## Limitations

- Annotations are putative and require experimental validation
- Neither the processed spectra nor the mapping to identified metabolites was provided by the original meat study, so the evaluation required replicating the study
- The pretrained MassSpecGym download URLs in the README are empty, so preprocessed files may need local regeneration
- Inference is demonstrated for positive ion mode data in the meat-study evaluation; other ion modes are not covered by that benchmark

## Evidence

- [results] we focus on results with a molecule–spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores: "we focus on results with a molecule–spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores"
- [results] Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
- [readme] You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates.: "You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates."
- [readme] python data_preprocess.py --spec_type binnedSpec --dataset_pth ../data/sample/data.tsv --candidates_pth  ../data/sample/candidates_mass.json --output_dir ../data/sample/: "python data_preprocess.py --spec_type binnedSpec --dataset_pth ../data/sample/data.tsv --candidates_pth  ../data/sample/candidates_mass.json --output_dir ../data/sample/"
- [results] The reported annotations are putative and require experimental validation.: "The reported annotations are putative and require experimental validation."
