---
name: tandem-ms-metabolite-annotation
description: Use when you have consensus MS/MS spectra (e.g. from MZmine3 feature
  extraction) with a precursor mass and need to rank molecular candidates retrieved
  from FoodDB, HMDB, MassSpecGym, NIST23, or GNPS libraries to annotate metabolites.
when_to_use_negative:
- When spectra lack precursor mass information, since candidate retrieval depends
  on a 0.01 Da mass tolerance.
- When the analysis goal is de novo structure generation or spectrum simulation rather
  than candidate ranking — MassSpecGym defines these as separate challenges.
- When reference library coverage is sufficient for direct spectral matching, since
  ML-based ranking is motivated by limited library coverage.
edam_operation: http://edamontology.org/operation_3804
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MVP
  role: Main contrastive model that ranks molecular candidates for each consensus
    spectrum via joint embedding of molecular graphs, fingerprints, spectra, and consensus
    spectra
  repo: https://github.com/HassounLab/MVP
- name: massspecgym
  role: Python package providing the MassSpecGym dataset, transforms, datamodules,
    and challenge evaluation used for training and benchmarking
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: pytorch
  role: Deep learning framework underlying MVP training and inference
- name: pytorch-geometric
  role: Handles molecular graph representations as one of the MVP views
- name: dgl
  role: Deep graph library used for molecular graph processing in MVP
- name: rdkit
  role: Cheminformatics toolkit for molecular fingerprints and candidate structure
    handling
- name: lightning
  role: Training/evaluation framework for the MVP model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:81d084fced86cb7e3c70ed1947c2cab2f837978397ace07a42cb9f8be3b06461
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - MVP
  - dgl
  - lightning
  - massspecgym
  - pytorch
  - pytorch-geometric
  - rdkit
  edam_operation: http://edamontology.org/operation_3804
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# tandem-ms-metabolite-annotation

## Summary

Annotate MS/MS spectra by ranking molecular candidates with MVP, a multiview contrastive model that jointly aligns molecular graphs, fingerprints, spectra, and consensus spectra in a shared embedding space. The skill covers the full inference pipeline from data preprocessing through pretrained-model scoring to high-confidence annotation filtering.

## When to use

Use when you have consensus MS/MS spectra (e.g. from MZmine3 feature extraction) with a precursor mass and need to rank molecular candidates retrieved from FoodDB, HMDB, MassSpecGym, NIST23, or GNPS libraries to annotate metabolites.

## When NOT to use

- When spectra lack precursor mass information, since candidate retrieval depends on a 0.01 Da mass tolerance.
- When the analysis goal is de novo structure generation or spectrum simulation rather than candidate ranking — MassSpecGym defines these as separate challenges.
- When reference library coverage is sufficient for direct spectral matching, since ML-based ranking is motivated by limited library coverage.

## Inputs

- Consensus spectra TSV file (e.g. data/sample/data.tsv)
- Candidate molecules JSON (candidates_mass.json)
- Subformula labels file for formSpec preprocessing
- params_formSpec.yaml or params_binnedSpec.yaml configuration
- Pretrained MVP model checkpoint (trained on MassSpecGym)

## Outputs

- Ranked molecular candidate lists per spectrum with molecule-spectra scores
- Filtered high-confidence annotations (score > 0.5, top-2 margin >= 0.01)
- Result files viewable in notebooks/demo.ipynb

## How to apply

Prepare the dataset by running data_preprocess.py on the consensus spectra TSV and candidates_mass.json, computing subformula labels via subformula_assign/assign_subformulae.py if using the formSpec view, which generates fingerprint and consensus spectra files for inference. Point the params_formSpec.yaml or params_binnedSpec.yaml at the processed dataset and candidate paths, then run test.py with the pretrained model (trained on MassSpecGym) to score each consensus spectrum against the candidate set. Candidates are retrieved within a 0.01 Da tolerance from FoodDB, HMDB, MassSpecGym, NIST23, and the GNPS Multiplex library. For biological interpretation, filter ranked results to those with a molecule-spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores, focusing on the appropriate ion mode. The rationale is that MVP's joint contrastive learning across all views produces better candidate ranking than approaches that concatenate views or use them as auxiliary tasks.

## Related tools

- **MVP** (Main contrastive model that ranks molecular candidates for each consensus spectrum via joint embedding of molecular graphs, fingerprints, spectra, and consensus spectra) — https://github.com/HassounLab/MVP
- **massspecgym** (Python package providing the MassSpecGym dataset, transforms, datamodules, and challenge evaluation used for training and benchmarking) — https://github.com/pluskal-lab/MassSpecGym
- **pytorch** (Deep learning framework underlying MVP training and inference)
- **pytorch-geometric** (Handles molecular graph representations as one of the MVP views)
- **dgl** (Deep graph library used for molecular graph processing in MVP)
- **rdkit** (Cheminformatics toolkit for molecular fingerprints and candidate structure handling)
- **lightning** (Training/evaluation framework for the MVP model)

## Examples

```
python data_preprocess.py --spec_type formSpec --dataset_pth ../data/sample/data.tsv --candidates_pth ../data/sample/candidates_mass.json --subformula_dir_pth ../data/sample/subformulae_default/ --output_dir ../data/sample/
```

## Evaluation signals

- For MassSpecGym retrieval, MVP with the consensus spectrum view achieves 35.99% rank@1 by mass and 13.96% rank@1 by formula; results below these baselines suggest misconfiguration.
- On the meat-study evaluation, MVP annotates 640 spectra corresponding to 146 unique molecules after applying the score > 0.5 and margin >= 0.01 filters.
- MVP with the mol-cs ranking view outperforms other tools across all metrics for both candidates by mass and formula on the MassSpecGym benchmark.
- Consensus-spectrum-based annotation must outperform rank aggregation of individual-spectrum annotations (26.37% rank@1 by mass, 11.10% by formula).
- Result files should be inspectable in notebooks/demo.ipynb with per-spectrum candidate rankings and scores.

## Limitations

- The reported annotations for the meat study are putative and require experimental validation.
- Neither the processed spectra nor the mapping to identified metabolites was provided by the original meat study; the evaluation was replicated independently.
- MVP's ranking losses could be improved with losses that prioritize ranking molecular candidates; this is listed as a future direction.
- Preprocessed MassSpecGym download URLs in the README are empty, requiring manual dataset preparation.

## Evidence

- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "Using the consensus spectrum view, MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively."
- [results] we focus on results with a molecule–spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores: "we focus on results with a molecule–spectra score greater than 0.5 and a margin of at least 0.01 between the top- and second-ranked candidate scores"
- [results] Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
- [results] Feature extraction and representative spectra selection are performed using MZmine3: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [readme] After prepping your data, modify the params_binnedSpec.yaml or params_formSpec.yaml with your dataset paths: "After prepping your data, modify the params_binnedSpec.yaml or params_formSpec.yaml with your dataset paths"
- [readme] Run our preprocess code to obatain fingerprints and consensus spectra files: "Run our preprocess code to obatain fingerprints and consensus spectra files"
