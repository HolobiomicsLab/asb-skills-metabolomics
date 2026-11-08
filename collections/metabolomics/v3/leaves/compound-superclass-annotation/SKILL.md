---
name: compound-superclass-annotation
description: Use when per-compound spectral prediction quality (e.g., spectral_sqrt_cosine
  from fiora-eval) must be grouped by chemical class, or when graph embeddings must
  be colored by superclass for UMAP visualization.
when_to_use_negative:
- Compounds lack valid SMILES or have inconsistencies between SMILES, MOL format,
  and InChiKeys, so class annotation would be unreliable
- The analysis needs finer chemical ontology (e.g., full ClassyFire class/subclass
  hierarchy or ChEBI roles) rather than the superclass level used here
- Spectral prediction quality is only needed as a single global score with no chemical-class
  stratification
tools:
- name: FIORA
  role: Produces the predicted spectra, per-compound spectral_sqrt_cosine evaluation
    scores, and per-molecule graph node embeddings that the superclass annotation
    stratifies
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:8dbfe7b5df3942c5725db397fcc9947061ecfa55b063aeabde4351d1e174bd65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# compound-superclass-annotation

## Summary

Annotate each compound in a spectral prediction evaluation set with its ClassyFire superclass derived from SMILES, so that per-compound spectral_sqrt_cosine scores and FIORA graph node embeddings can be stratified by chemical class.

## When to use

Use when per-compound spectral prediction quality (e.g., spectral_sqrt_cosine from fiora-eval) must be grouped by chemical class, or when graph embeddings must be colored by superclass for UMAP visualization.

## When NOT to use

- Compounds lack valid SMILES or have inconsistencies between SMILES, MOL format, and InChiKeys, so class annotation would be unreliable
- The analysis needs finer chemical ontology (e.g., full ClassyFire class/subclass hierarchy or ChEBI roles) rather than the superclass level used here
- Spectral prediction quality is only needed as a single global score with no chemical-class stratification

## Inputs

- Compound input CSV with columns Name, SMILES, Precursor_type, CE, Instrument_type
- Per-compound spectral_sqrt_cosine scores from fiora-eval (validation_eval.csv, test_eval.csv)
- FIORA per-molecule graph node embeddings (for mean-pooling and UMAP)

## Outputs

- Superclass-annotated compound table (SMILES-to-ClassyFire superclass mapping)
- Per-superclass median spectral_sqrt_cosine table
- 2-D UMAP projection of mean-pooled node embeddings colored by superclass

## How to apply

After running fiora-predict and fiora-eval (which writes validation_eval.csv and test_eval.csv with per-compound spectral_sqrt_cosine scores), assign a ClassyFire superclass to each test compound directly from its SMILES structure. Group the per-compound cosine similarity scores by superclass and compute the median per superclass, checking agreement with the paper's reported values: organohalogen compounds 0.63, hydrocarbon derivatives 0.66, and nucleosides 0.86. Optionally, mean-pool FIORA's per-molecule graph node embeddings and project them to 2-D with UMAP, coloring points by superclass to reproduce Fig. 4 panels a-c. The rationale is that prediction quality may vary systematically across compound classes, so class-level annotation turns a single global score into a stratified robustness analysis.

## Related tools

- **FIORA** (Produces the predicted spectra, per-compound spectral_sqrt_cosine evaluation scores, and per-molecule graph node embeddings that the superclass annotation stratifies) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf
```

## Evaluation signals

- Every test compound with a valid SMILES receives exactly one ClassyFire superclass assignment
- Per-superclass median spectral_sqrt_cosine reproduces reported values: organohalogen compounds 0.63, hydrocarbon derivatives 0.66, nucleosides 0.86
- Median cosine similarity is well above 0.7 for all superclasses except Organohalogen compounds and Hydrocarbon derivatives
- The rendered UMAP shows superclass-colored embedding clusters consistent with Fig. 4 panels a-c

## Limitations

- FIORA prediction quality itself is lower for organohalogen compounds (0.63) and hydrocarbon derivatives (0.66), so superclass-level medians in these groups should be interpreted cautiously
- ClassyFire superclass assignment is only as good as the input SMILES; compounds filtered out for SMILES/MOL/InChiKey inconsistencies cannot be annotated
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, so rare superclasses may have few test compounds and unstable medians

## Evidence

- [other] Assign a ClassyFire superclass to each test compound from its SMILES structure.: [evidence span withheld — anchor status: not_found]
- [other] Median cosine similarity above 0.7 across superclasses: [evidence span withheld — anchor status: not_found]
- [readme] Input CSV format for fiora-predict: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [discussion] Training data scale limitation: "FIORA was trained on a relatively small dataset of ~10,000 unique compounds, greatly limited by the lack of experimental metadata"
