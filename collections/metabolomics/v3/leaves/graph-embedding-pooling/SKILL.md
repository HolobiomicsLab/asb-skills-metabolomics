---
name: graph-embedding-pooling
description: Use when you need to visualize whether a graph neural network's learned
  molecular representations cluster by compound superclass, or to diagnose per-class
  spectral prediction quality from per-compound cosine similarity scores.
when_to_use_negative:
- When the goal is comparing fragmentation algorithms (e.g., FIORA vs. ICEBERG/CFM-ID
  benchmarking) rather than inspecting learned representation structure
- When compounds lack valid SMILES, since superclass assignment and graph construction
  both depend on structure (inconsistencies between SMILES, MOL format and InChiKeys
  must be filtered first)
- When using a model that pools node features into a single embedding already, since
  the bond-level locality rationale for this pooling approach no longer applies
edam_operation: http://edamontology.org/operation_3646
tools:
- name: FIORA
  role: Predicts MS/MS spectra with fiora-predict and provides the per-molecule graph
    node embeddings that are mean-pooled and UMAP-projected; fiora-eval computes spectral_sqrt_cosine
    scores per compound
  repo: https://github.com/BAMeScience/fiora
- name: ClassyFire
  role: Assigns a superclass to each test compound from its SMILES structure for grouping
    cosine scores and coloring UMAP points
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:1e2ab13303d8f8669838f0316d2e62712625800d0c3c04a74bef37e7a12c2ec2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - ClassyFire
  - FIORA
  edam_operation: http://edamontology.org/operation_3646
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# graph-embedding-pooling

## Summary

Mean-pool FIORA's per-molecule graph node embeddings into fixed-size molecular vectors, project them to 2-D with UMAP, and color by ClassyFire superclass to assess how learned spectral-prediction representations organize by chemical class alongside per-superclass spectral_sqrt_cosine prediction quality.

## When to use

Use when you need to visualize whether a graph neural network's learned molecular representations cluster by compound superclass, or to diagnose per-class spectral prediction quality from per-compound cosine similarity scores.

## When NOT to use

- When the goal is comparing fragmentation algorithms (e.g., FIORA vs. ICEBERG/CFM-ID benchmarking) rather than inspecting learned representation structure
- When compounds lack valid SMILES, since superclass assignment and graph construction both depend on structure (inconsistencies between SMILES, MOL format and InChiKeys must be filtered first)
- When using a model that pools node features into a single embedding already, since the bond-level locality rationale for this pooling approach no longer applies

## Inputs

- Test compounds CSV with header columns: Name, SMILES, Precursor_type, CE, Instrument_type
- Trained FIORA model checkpoint (e.g., checkpoints/fiora.pt) or the default packaged model under fiora/resources/models
- SMILES structures for ClassyFire superclass assignment
- FIORA per-molecule graph node embeddings from the trained model

## Outputs

- Predicted spectra in mgf (or msp) format
- Per-compound spectral_sqrt_cosine similarity scores in validation_eval.csv and test_eval.csv
- Per-superclass median spectral_sqrt_cosine table
- 2-D UMAP projection of mean-pooled node embeddings colored by superclass (Fig. 4 panels a-c)

## How to apply

First generate predicted spectra with fiora-predict on a test-set CSV (columns Name, SMILES, Precursor_type, CE, Instrument_type) using the packaged default model under fiora/resources/models, then run fiora-eval with the trained checkpoint (e.g., checkpoints/fiora.pt) and --output-dir to compute per-compound spectral_sqrt_cosine scores written to validation_eval.csv and test_eval.csv. Assign a ClassyFire superclass to each compound from its SMILES and group the per-compound spectral_sqrt_cosine scores by superclass, computing the median per group. Rationale: FIORA learns bond-level representations from the 'molecular neighborhood of bonds', so pooling node embeddings per molecule yields a class-sensitive molecular vector. Mean-pool the per-molecule graph node embeddings, project to 2-D with UMAP, and color points by superclass to reproduce the Fig. 4 panels. Check the medians against reported values: organohalogen compounds 0.63 and hydrocarbon derivatives 0.66 are the only classes below the 'well above 0.7' level seen elsewhere (e.g., nucleosides at 0.86).

## Related tools

- **FIORA** (Predicts MS/MS spectra with fiora-predict and provides the per-molecule graph node embeddings that are mean-pooled and UMAP-projected; fiora-eval computes spectral_sqrt_cosine scores per compound) — https://github.com/BAMeScience/fiora
- **ClassyFire** (Assigns a superclass to each test compound from its SMILES structure for grouping cosine scores and coloring UMAP points)

## Examples

```
fiora-predict -i test_compounds.csv -o test_pred.mgf && fiora-eval -c checkpoints/fiora.pt -i test_compounds.csv --output-dir eval_out
```

## Evaluation signals

- Per-superclass median spectral_sqrt_cosine reproduces reported values: organohalogen compounds 0.63, hydrocarbon derivatives 0.66, nucleosides 0.86
- All superclasses except Organohalogen compounds and Hydrocarbon derivatives show median cosine similarity well above 0.7
- test_eval.csv and validation_eval.csv contain a spectral_sqrt_cosine column with one score per compound
- Rendered UMAP shows superclass-colored clusters of the mean-pooled embeddings, reproducing Fig. 4 panels a-c
- fiora-predict output mgf matches the expected results (examples/expected_output.mgf) for the example input, verifying the model used for embeddings

## Limitations

- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, so embedding structure may not generalize to classes underrepresented in training
- Organohalogen compounds and hydrocarbon derivatives yield degraded prediction quality (median spectral_sqrt_cosine of 0.63 and 0.66), so class-level interpretation should flag these as weak classes
- Exact model specification is only in the Supplementary Information, complicating independent re-implementation of the embedding extraction

## Evidence

- [abstract] FIORA uses the molecular neighborhood of bonds to learn breaking patterns and derive fragment ion probabilities: "utilizing the molecular neighborhood of bonds to learn breaking patterns and derive fragment ion probabilities"
- [results] Median cosine similarity well above 0.7 for all superclasses except organohalogens and hydrocarbon derivatives: [evidence span withheld — anchor status: not_found]
- [readme] Default model packaged under fiora/resources/models: "Default models are packaged under `fiora/resources/models`. The CLI uses these automatically when `--model default` is selected."
- [readme] Input CSV format specification: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
