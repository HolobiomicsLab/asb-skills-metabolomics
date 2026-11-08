---
name: multiview-contrastive-model-inference
description: Use when you have preprocessed MS/MS spectra (formSpec with subformula
  labels or binnedSpec) plus a candidate molecule list (by mass or formula) and need
  ranked candidate structures using the pretrained MVP checkpoint.
when_to_use_negative:
- When the task is de novo molecule generation or spectrum simulation (MassSpecGym
  challenges) rather than candidate retrieval from a provided candidate list
- When spectra whose molecules appear in CFM-ID's training set must be included, or
  other fairness-filter conditions from the benchmark apply
- When candidates have not been retrieved within the 0.01 Da mass tolerance and no
  candidate list exists — MVP ranks provided candidates, it does not enumerate them
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MVP
  role: Multiview contrastive model (mol-cs consensus-spectrum ranking view) used
    to rank candidate molecules from spectra
  repo: github.com/HassounLab/MVP
- name: MassSpecGym
  role: Benchmark dataset and evaluation infrastructure providing train/test splits
    and candidate sets for retrieval
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: lightning
  role: Training/inference framework used by MVP's test.py pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:a3d8796bea6c96ba0de2de659ed34cd71df67f1eee55636846367becf6018f72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MVP
  - MassSpecGym
  - lightning
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multiview-contrastive-model-inference

## Summary

Run MVP's pretrained multiview contrastive model (molecular graph, fingerprint, spectrum, consensus-spectrum views) to rank molecular candidates for MS/MS annotation, including the mol-cs consensus-spectrum ranking view that outperforms per-spectrum rank aggregation on MassSpecGym.

## When to use

Use when you have preprocessed MS/MS spectra (formSpec with subformula labels or binnedSpec) plus a candidate molecule list (by mass or formula) and need ranked candidate structures using the pretrained MVP checkpoint.

## When NOT to use

- When the task is de novo molecule generation or spectrum simulation (MassSpecGym challenges) rather than candidate retrieval from a provided candidate list
- When spectra whose molecules appear in CFM-ID's training set must be included, or other fairness-filter conditions from the benchmark apply
- When candidates have not been retrieved within the 0.01 Da mass tolerance and no candidate list exists — MVP ranks provided candidates, it does not enumerate them

## Inputs

- Preprocessed MS/MS spectra files (formSpec with subformula assignments or binnedSpec) in data.tsv-style format
- Candidate molecule list (e.g. candidates_mass.json), retrieved within a 0.01 Da tolerance from databases such as FoodDB, HMDB, MassSpecGym, NIST23, or GNPS
- Pretrained MVP checkpoint trained on MassSpecGym
- MVP config file (params_formSpec.yaml or params_binnedSpec.yaml)

## Outputs

- Ranked candidate molecule lists per query spectrum or per consensus spectrum
- rank@1 / rank@k metrics per ranking view (e.g. mol-cs vs per-spectrum rank aggregation)
- Structured result files demonstrated in notebooks/demo.ipynb; comparison table CSV for aggregate-then-rank vs rank-then-aggregate

## How to apply

Prepare spectra data and candidate lists (e.g. candidates_mass.json), then run MVP's preprocessing (data_preprocess.py with --spec_type formSpec or binnedSpec, including assign_subformulae.py with --max-formulae 60 for formSpec) to generate fingerprint and consensus-spectrum files. Edit params_formSpec.yaml or params_binnedSpec.yaml with your dataset paths and run test.py with the pretrained MassSpecGym checkpoint to produce candidate rankings. For molecules with multiple query spectra, use the mol-cs ranking view (aggregate-then-rank via consensus spectrum) rather than ranking each constituent spectrum separately and aggregating ranks (average rank or reciprocal rank), since on the three-spectrum test subset consensus annotation achieved 35.99% rank@1 vs 26.37% by mass, and 13.96% vs 11.10% by formula. If you need to reproduce the benchmark comparison, restrict to test molecules with exactly three spectra (1,531 cases) and compute rank@1 for both strategies.

## Related tools

- **MVP** (Multiview contrastive model (mol-cs consensus-spectrum ranking view) used to rank candidate molecules from spectra) — github.com/HassounLab/MVP
- **MassSpecGym** (Benchmark dataset and evaluation infrastructure providing train/test splits and candidate sets for retrieval) — https://github.com/pluskal-lab/MassSpecGym
- **lightning** (Training/inference framework used by MVP's test.py pipeline)

## Examples

```
python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- On the MassSpecGym test subset of molecules with exactly three spectra, mol-cs consensus ranking should give ~35.99% rank@1 (mass candidates) and ~13.96% rank@1 (formula candidates); per-spectrum average/reciprocal rank aggregation should land near 26.37% / 11.10%
- Subformula labels for formSpec must exist (assign_subformulae.py output) before preprocessing; preprocessing must produce fingerprint and consensus spectra files
- Candidate retrieval mass tolerance must not exceed 0.01 Da around the query precursor
- Result files should match the structure shown in notebooks/demo.ipynb
- For strict benchmark reproduction, the evaluation subset must contain exactly 1,531 three-spectrum test cases

## Limitations

- Annotation results are putative and require experimental validation
- Representation quality could benefit from more accurate peak labeling and additional views; losses that prioritize ranking molecular candidates are not currently included in MVP
- The preprocessed MassSpecGym download URLs in the README are empty, so users may need to preprocess the dataset themselves
- Rank-aggregation comparison findings apply specifically to the test subset with exactly three spectra; consensus-spectrum generation requires multiple constituent spectra per molecule

## Evidence

- [abstract] We present MultiView Projection (MVP), a novel framework for learning a joint embedding space between molecules and spectra: "We present MultiView Projection (MVP), a novel framework for learning a joint embedding space between molecules and spectra"
- [results] MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula: "MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula"
- [results] we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra: "we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra"
- [readme] You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates.: "You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates."
- [readme] python test.py --param_pth params_formSpec.yaml: "# If using formSpec
python test.py --param_pth params_formSpec.yaml"
- [results] Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
