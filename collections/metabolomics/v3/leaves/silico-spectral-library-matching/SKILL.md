---
name: silico-spectral-library-matching
description: Use when an unannotated empirical tandem MS spectrum must be matched
  against candidate molecules whose predicted spectra are available, and ranking accuracy
  (Top-1/Top-5/Top-10) or annotation confidence is the goal.
when_to_use_negative:
- The query spectrum already has a confident empirical library match — curated empirical
  libraries like NIST MS/MS outperform in silico matching when coverage exists
- No precursor mass information is available to filter candidates, since candidate-list
  assembly relies on 1 ppm / 10 ppm precursor mass accuracy tolerances
- The molecule class falls outside the ANN model's trained domain (e.g., needs 3D
  structure; CFM-ID or 3DMolMS may be alternatives)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SingleFrag
  role: Generates the in silico predicted MS/MS spectra (per-MZ-position ANN models)
    against which empirical spectra are cosine-matched for candidate ranking
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds candidate SMILES into the 300-dimensional vector used as input to SingleFrag's
    ANN models
- name: 3DMolMS
  role: Alternative deep learning spectral predictor for benchmarking ranking accuracy
    against SingleFrag
  repo: https://github.com/JosieHong/3DMolMS
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:81681d086758d1d7ad6abb3eea94fcf97d35a907651ae1ec4ba17177d4ae9c3f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  - mol2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# silico-spectral-library-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Annotate unannotated empirical MS/MS spectra by ranking candidate molecules with cosine similarity between the experimental spectrum and SingleFrag's in silico predicted spectra from a ~1.9M-molecule library.

## When to use

Use when an unannotated empirical tandem MS spectrum must be matched against candidate molecules whose predicted spectra are available, and ranking accuracy (Top-1/Top-5/Top-10) or annotation confidence is the goal.

## When NOT to use

- The query spectrum already has a confident empirical library match — curated empirical libraries like NIST MS/MS outperform in silico matching when coverage exists
- No precursor mass information is available to filter candidates, since candidate-list assembly relies on 1 ppm / 10 ppm precursor mass accuracy tolerances
- The molecule class falls outside the ANN model's trained domain (e.g., needs 3D structure; CFM-ID or 3DMolMS may be alternatives)

## Inputs

- Unannotated empirical MS/MS spectra (e.g., from the 2,401-spectrum test set or ARUS database recurrent unidentified spectra)
- Predicted in silico spectral library from SingleFrag per-MZ-position ANN models (ModelVal*.pickle, ~1.9M molecules)
- Candidate molecule lists with SMILES and precursor masses (inputMols_git.xlsx format)
- Per-query top-1 correctness labels for validation

## Outputs

- Ranked candidate list per query spectrum with cosine similarity scores for top candidates
- Top-1/Top-5/Top-10 ranking accuracy table across collision-energy conditions (low/medium/high/merged) and ppm tolerances
- Δcos gap-stratified accuracy table (gap_stratified_accuracy.csv) and plot (gap_stratified_accuracy.png)

## How to apply

Generate predicted spectra for candidate molecules with SingleFrag's spectraPrediction.ipynb pipeline, which applies per-MZ-position ANN models (ModelVal*.pickle) to a 300-dimensional mol2vec embedding of each SMILES and reconstructs spectra from per-position peak predictions. Assemble candidate lists per query spectrum, filtering by precursor mass accuracy at 1 ppm and 10 ppm tolerances. Compute cosine similarity between each query spectrum and all candidate predicted spectra, then rank candidates by score. Validate ranking by reporting the fraction of queries where the true compound appears at Top-1, Top-5, or Top-10 positions; merged spectra across all collision energies give 37.6%/71.9%/82.7% Top-1/Top-5/Top-10. As a confidence gate, compute Δcos = cos(top-1) − cos(top-2): when the gap exceeds 0.2, the top-ranked candidate is correct in the majority of cases.

## Related tools

- **SingleFrag** (Generates the in silico predicted MS/MS spectra (per-MZ-position ANN models) against which empirical spectra are cosine-matched for candidate ranking) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds candidate SMILES into the 300-dimensional vector used as input to SingleFrag's ANN models)
- **3DMolMS** (Alternative deep learning spectral predictor for benchmarking ranking accuracy against SingleFrag) — https://github.com/JosieHong/3DMolMS

## Examples

```
jupyter notebook spectraPrediction.ipynb  # loads data/ANN/ModelVal*.pickle + Mol2vecModel, writes results/dfPredictionPeak_***.pickle and results/predSpectra.json
```

## Evaluation signals

- Top-1/Top-5/Top-10 accuracies on the test set with exact molecular formula known and merged collision energies reach 37.6%/71.9%/82.7% within expected numeric agreement
- Accuracy is evaluated separately at 1 ppm and 10 ppm precursor mass filter tolerances and across four collision-energy conditions (low/medium/high/merged)
- Bins with Δcos exceeding 0.2 show top-candidate correctness above 0.5 (top candidate correct in the majority of cases)
- Reconstructed predicted spectra are saved as results/predSpectra.json and per-MZ-position predictions as results/dfPredictionPeak_***.pickle
- All SingleFrag models should outperform CFM-ID and 3DMolMS on equivalent ranking benchmarks

## Limitations

- GNN model shows slightly lower performance than ANN; only ANN models are used in spectraPrediction.ipynb because they got better results
- Public spectral libraries such as GNPS, MSDial, or MassBank often lack sufficient curation and typically contain spectra acquired at only one or a few collision energies per compound, so merged collision-energy spectra give the best ranking
- The toy example spectrum format only contains integers while predictions have 2-decimal MZ resolution
- Author neural network architectures were deliberately kept simple and are not optimized

## Evidence

- [full_text] the merged (37.6/71.9/82.7%): [evidence span withheld — anchor status: not_found]
- [results] when the difference in cosine similarity scores between the top and second ranked candidates exceeds 0.2, the top candidate is the correct molecule in the majority of cases: [evidence span withheld — anchor status: not_found]
