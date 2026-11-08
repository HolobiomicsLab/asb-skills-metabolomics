---
name: umap-embedding-visualization
description: Use when a trained FIORA checkpoint and a test compound set are available
  and you need to verify that graph embeddings encode chemical class structure (Fig.
  4) by UMAP clustering plus per-superclass cosine similarity medians.
when_to_use_negative:
- The goal is benchmarking against ICEBERG or CFM-ID on CASMI 2016/2022 challenge
  data rather than inspecting FIORA's embedding space or per-superclass quality
- Compounds are outside the curated scope (e.g., missing precursor modes such as [M+H]+/[M-H]-,
  mass above 1000 Da, or collision energy above 100 eV), so superclass-level cosine
  similarity is not comparable to reported values
- You only need spectrum-level comparison of predicted vs. experimental peaks and
  no structural-class interpretation
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Generates predicted MS/MS spectra via fiora-predict, computes spectral_sqrt_cosine
    evaluation scores via fiora-eval, and supplies per-molecule graph node embeddings
    for mean-pooling and UMAP projection
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:ea1b3b32778f36b5cad6ec433574ee482923b3ab49c90adb080a845127bf82a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# umap-embedding-visualization

## Summary

Visualize and validate FIORA's learned graph representations by mean-pooling per-molecule node embeddings, projecting them to 2-D with UMAP colored by ClassyFire superclass, and cross-checking clustering against per-superclass spectral_sqrt_cosine prediction quality.

## When to use

Use when a trained FIORA checkpoint and a test compound set are available and you need to verify that graph embeddings encode chemical class structure (Fig. 4) by UMAP clustering plus per-superclass cosine similarity medians.

## When NOT to use

- The goal is benchmarking against ICEBERG or CFM-ID on CASMI 2016/2022 challenge data rather than inspecting FIORA's embedding space or per-superclass quality
- Compounds are outside the curated scope (e.g., missing precursor modes such as [M+H]+/[M-H]-, mass above 1000 Da, or collision energy above 100 eV), so superclass-level cosine similarity is not comparable to reported values
- You only need spectrum-level comparison of predicted vs. experimental peaks and no structural-class interpretation

## Inputs

- Test compounds CSV with header columns: Name, SMILES, Precursor_type, CE, Instrument_type
- Trained FIORA checkpoint (e.g., checkpoints/fiora.pt) or default model under fiora/resources/models
- ClassyFire superclass annotations derived from compound SMILES
- FIORA per-molecule graph node embeddings from the packaged model

## Outputs

- Predicted spectra in mgf (or msp) format from fiora-predict
- validation_eval.csv and test_eval.csv with per-compound spectral_sqrt_cosine similarity scores
- 2-D UMAP embedding of mean-pooled node embeddings colored by superclass (Fig. 4 panels a-c)
- Per-superclass median spectral_sqrt_cosine table

## How to apply

Run fiora-predict on the test compounds CSV (columns: Name, SMILES, Precursor_type, CE, Instrument_type) with the default packaged model under fiora/resources/models to generate predicted spectra in mgf format. Then run fiora-eval with the trained checkpoint (e.g., checkpoints/fiora.pt) and --output-dir to compute per-compound spectral_sqrt_cosine scores into validation_eval.csv and test_eval.csv. Assign a ClassyFire superclass to each test compound from its SMILES. Mean-pool FIORA's per-molecule graph node embeddings and project them to 2-D with UMAP, coloring points by superclass to reproduce Fig. 4 panels a-c. Group spectral_sqrt_cosine scores by superclass, compute medians, and check agreement with reported values (organohalogens 0.63, hydrocarbon derivatives 0.66, nucleosides 0.86; all other superclasses well above 0.7).

## Related tools

- **FIORA** (Generates predicted MS/MS spectra via fiora-predict, computes spectral_sqrt_cosine evaluation scores via fiora-eval, and supplies per-molecule graph node embeddings for mean-pooling and UMAP projection) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf
```

## Evaluation signals

- The rendered UMAP shows superclass-colored clusters of the mean-pooled node embeddings, matching Fig. 4 panels a-c
- Per-superclass median spectral_sqrt_cosine reproduces reported values: Organohalogen compounds 0.63, Hydrocarbon derivatives 0.66, Nucleosides 0.86
- All superclasses other than Organohalogen compounds and Hydrocarbon derivatives reach a median cosine similarity well above 0.7
- fiora-predict output mgf matches the packaged expected results (examples/expected_output.mgf), which pytest verifies automatically
- test_eval.csv and validation_eval.csv exist with a spectral_sqrt_cosine column for every test compound

## Limitations

- Median cosine similarity degrades for Organohalogen compounds (0.63) and Hydrocarbon derivatives (0.66), so embeddings for these classes may cluster less cleanly
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, which may limit embedding generalization to structurally diverse test sets
- Exact model specification is only in the Supplementary Information, so reproducing embeddings requires the packaged checkpoint

## Evidence

- [readme] By default, an open-source model is selected automatically, and predictions typically complete within a few seconds.: "By default, an open-source model is selected automatically, and predictions typically complete within a few seconds."
- [readme] Default models are packaged under fiora/resources/models.: "Default models are packaged under `fiora/resources/models`. The CLI uses these automatically when `--model default` is selected."
- [other] Mean-pool FIORA's per-molecule graph node embeddings and project them to 2-D with UMAP, coloring points by superclass to reproduce Fig. 4 panels a-c.: [evidence span withheld — anchor status: not_found]
- [other] FIORA achieves a median cosine similarity well above 0.7 for all compound superclasses except Organohalogen compounds and Hydrocarbon derivatives.: [evidence span withheld — anchor status: not_found]
- [other] Group the per-compound spectral_sqrt_cosine scores by superclass and compute the median per superclass, checking agreement with the reported values.: [evidence span withheld — anchor status: not_found]
