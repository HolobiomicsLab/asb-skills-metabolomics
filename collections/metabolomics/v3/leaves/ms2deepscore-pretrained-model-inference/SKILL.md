---
name: ms2deepscore-pretrained-model-inference
description: Use when MS/MS spectra in .mgf format need pairwise spectral similarity
  scores and no suitable MS/MS library search target exists, so pretrained machine-learned
  scoring (ms2deepscore via MS2Query) is required.
when_to_use_negative:
- Metabolites/spectra far outside ms2deepscore's training domain — the authors caution
  it 'can only be expected to work well for spectra and metabolites well covered by
  or related to its training data', with no coverage check reported
- Datasets well beyond ~5000 features — dashboard scalability above this is untested
  and performance for larger datasets is only anticipated
- Situations where a conventional library-search similarity (e.g. cosine to reference
  spectra) suffices and pretrained ML scores would add unnecessary MS2Query model-loading
  overhead (~62 min for the urine dataset)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: ms2deepscore
  role: Provides the pretrained machine-learned pairwise spectral similarity used
    as the primary score
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: Supplies the pretrained models (and MS2Query local library search) consumed
    by the importing pipeline
- name: specXplore
  role: Importing pipeline that orchestrates the pairwise similarity computation and
    packages results into a session data object
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports and preprocesses the .mgf spectra before similarity scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:373a319c79d29fedee101cc4e9563772c7a4cc7ddae90ec70a87e115f1fb76b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# ms2deepscore-pretrained-model-inference

## Summary

Runs MS2Query-provided pretrained ms2deepscore models to compute the full pairwise ms2deepscore similarity matrix for LC–MS/MS features in .mgf format, which serves as the quantitative backbone for downstream specXplore visualizations such as the t-SNE overview embedding.

## When to use

Use when MS/MS spectra in .mgf format need pairwise spectral similarity scores and no suitable MS/MS library search target exists, so pretrained machine-learned scoring (ms2deepscore via MS2Query) is required.

## When NOT to use

- Metabolites/spectra far outside ms2deepscore's training domain — the authors caution it 'can only be expected to work well for spectra and metabolites well covered by or related to its training data', with no coverage check reported
- Datasets well beyond ~5000 features — dashboard scalability above this is untested and performance for larger datasets is only anticipated
- Situations where a conventional library-search similarity (e.g. cosine to reference spectra) suffices and pretrained ML scores would add unnecessary MS2Query model-loading overhead (~62 min for the urine dataset)

## Inputs

- MS/MS spectral feature data in .mgf format (each entry with a unique 'feature_id' key, precursor m/z, and m/z–intensity tuples)
- Pretrained MS2Query ms2deepscore model files (Zenodo records 10527997 for positive mode, 10528030 for negative mode)

## Outputs

- Full pairwise ms2deepscore similarity matrix over all features
- specXplore session data object saved to disk for direct loading into the specXplore dashboard
- Companion modified cosine and spec2vec similarity matrices from the same pipeline call

## How to apply

Import the .mgf MS/MS spectra into Python with matchms and run the specXplore importing pipeline in a Jupyter notebook (python 3.8 conda environment), which automatically computes pairwise similarities using ms2deepscore as the primary score alongside modified cosine and spec2vec. Load the pretrained MS2Query model and library files (separate Zenodo downloads for positive and negative ion modes, placed in a 'models/pos' sub-directory for the illustrative example notebooks). The pipeline call computes the full pairwise matrix in one go — reported ~9 min for a urine metabolome dataset of <4000 features and <15 min total preprocessing for a wheat dataset of <1000 features. For positive/negative mode data, select the matching model set; the resulting ms2deepscore matrix is exported and used for t-SNE embedding with perplexity tuning between 5 and 50.

## Related tools

- **ms2deepscore** (Provides the pretrained machine-learned pairwise spectral similarity used as the primary score) — https://github.com/kevinmildau/specXplore
- **MS2Query** (Supplies the pretrained models (and MS2Query local library search) consumed by the importing pipeline)
- **specXplore** (Importing pipeline that orchestrates the pairwise similarity computation and packages results into a session data object) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports and preprocesses the .mgf spectra before similarity scoring)

## Examples

```
conda create --name specxplore_environment python=3.8; conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- All pairwise similarity values are finite and bounded (ms2deepscore scores are model predictions in [0,1]); no NaNs from the affected macOS arm64 issue 199 scenario
- Matrix is square with N x N entries for N features, consistent with the .mgf feature count after matchms preprocessing
- Pipeline timing consistent with reported benchmarks: ~9 min for all pairwise similarity computations on <4000 features; <75 min total for the urine dataset including ~62 min MS2Query
- High-ms2deepscore-similarity feature pairs occupy nearby positions in the t-SNE embedding computed from the matrix
- Importing pipeline produces a session data object loadable by the specXplore dashboard without errors

## Limitations

- macOS arm64 (Apple M1/M2): ms2deepscore issue 199 can silently produce similarity predictions inconsistent with other systems — 'does not result in any errors or warning messages, but makes ms2deepscore results unreliable'
- Applicability domain: ms2deepscore works reliably only for spectra/metabolites covered by or related to its training data; no coverage check is performed by the pipeline
- specXplore (and this scoring path) fails on Windows in its current version; requires macOS or Linux (or the Anaconda power shell plus MSVC redistributable workaround)
- No tool version numbers are pinned in the article, complicating exact reproducibility

## Evidence

- [methods] Pretrained models via MS2Query: "For the machine learning scores, pretrained models provided with MS2Query are used."
- [methods] Pipeline computes three similarity scores: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [results] Urine dataset runtime: "the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [readme] README model file sources: [evidence span withheld — anchor status: not_found]
- [readme] README arm64 reliability warning: [evidence span withheld — anchor status: external_doc]
- [discussion] Applicability domain caveat: "it can only be expected to work well for spectra and metabolites well covered by or related to its training data"
