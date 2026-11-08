---
name: ms2deepscore-embedding-generation
description: Use when a set of HR-MS/MS spectra (e.g. MSnLib MGF files plus in-house
  BOKU database spectra, or inference extracts) must be converted into MS2DeepScore
  structure-informed embeddings before AnnoMe classifier training or prediction.
when_to_use_negative:
- Embeddings for the spectra set have already been computed and cached on disk — reuse
  the cache instead of regenerating.
- The goal is to fix classification failure caused by a highly imbalanced training
  dataset; the paper states the package 'is not capable of resolving this issue'.
- The available machine lacks the compute budget/resources (e.g. a passively cooled
  laptop with 16 GB RAM), since embedding generation is the limiting, hours-long step.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: AnnoMe
  role: Python package whose workflow computes and caches the MS2DeepScore embeddings
    for filtered MGF spectra as the input feature representation for its LDA/NN/SVM
    majority-vote classifiers
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Provides the structure-informed spectral embedding model applied to each HR-MS/MS
    spectrum
- name: uv
  role: Package/runner used to install and execute the AnnoMe embedding pipeline (`uv
    run` commands)
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:8f3f2f5f021d0701d347a336ffe0b46b19a17b486b94313d9fc335aa4427c1bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# ms2deepscore-embedding-generation

## Summary

Generates MS2DeepScore structure-informed embeddings for HR-MS/MS spectra (MGF files) so that AnnoMe can train LDA/NN/SVM 'relevant' vs 'other' classifiers and apply a majority vote. This is the most time-consuming step of the AnnoMe workflow and must be run once per dataset.

## When to use

Use when a set of HR-MS/MS spectra (e.g. MSnLib MGF files plus in-house BOKU database spectra, or inference extracts) must be converted into MS2DeepScore structure-informed embeddings before AnnoMe classifier training or prediction.

## When NOT to use

- Embeddings for the spectra set have already been computed and cached on disk — reuse the cache instead of regenerating.
- The goal is to fix classification failure caused by a highly imbalanced training dataset; the paper states the package 'is not capable of resolving this issue'.
- The available machine lacks the compute budget/resources (e.g. a passively cooled laptop with 16 GB RAM), since embedding generation is the limiting, hours-long step.

## Inputs

- Annotated/labeled HR-MS/MS spectra in MGF format (e.g. filtered MSnLib v6 spectra libraries and BOKU in-house database spectra) as 'relevant'/'other' training data
- Inference MS/MS spectra from plant extracts (e.g. Paulownia tomentosa fruits, Glycyrrhiza uralensis roots)

## Outputs

- MS2DeepScore structure-informed embeddings for every input MS/MS spectrum, cached on disk and reused by AnnoMe classifier training (LDA, NN, SVM with majority vote) and inference

## How to apply

After downloading the public MS/MS resources (~11 GB via `uv run annome_downloadresources`) and filtering MGF spectra files into 'relevant'/'other' labels with the AnnoMe filtering GUI, run the embedding generation step from within AnnoMe for all filtered spectra. Expect this step to dominate runtime: the reference run took about 4 hours on a 12-core laptop (Intel Core Ultra 5 125U, 16 GB RAM, SSD); less powerful machines without active cooling may not complete it. Embeddings are cached on disk after the first run, so schedule this once (e.g. overnight or over a weekend) and rely on the cache in subsequent runs, when the step drops from hours to a few minutes. The cached embeddings then feed training of LDA, NN, and SVM classifiers with cross-validation and different random seeds, and must be computed for both training spectra and inference spectra (e.g. Paulownia tomentosa and Glycyrrhiza uralensis extracts) before majority-vote classification.

## Related tools

- **AnnoMe** (Python package whose workflow computes and caches the MS2DeepScore embeddings for filtered MGF spectra as the input feature representation for its LDA/NN/SVM majority-vote classifiers) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Provides the structure-informed spectral embedding model applied to each HR-MS/MS spectrum)
- **uv** (Package/runner used to install and execute the AnnoMe embedding pipeline (`uv run` commands)) — https://github.com/astral-sh/uv

## Evaluation signals

- Every spectrum in the filtered MGF training and inference sets has a corresponding cached embedding (no missing spectra after the run).
- Subsequent AnnoMe runs load embeddings from the disk cache, reducing this step from hours to a few minutes.
- Downstream classifier training (LDA, NN, SVM with cross-validation and multiple random seeds) succeeds on the embeddings and yields balanced accuracy in the reported range (training ~0.866, test ~0.82 for the flavonoid demo).
- Embedding generation completes only on hardware meeting the reference profile (12 cores, 16 GB RAM, SSD); a hardware failure or abort indicates the resource requirement was not met.
- The derived 'relevant'-class training-set size (~1000 spectra) remains sufficient: 'relevant'-class recall stays near ~62% from 100% down to 50% training fractions.

## Limitations

- Embedding generation is the most time-consuming step (about 4 hours on a standard 12-core laptop) and must be scheduled in advance; the same dataset could not be processed on a less powerful MacBook Air (M4, 16 GB RAM, passive cooling).
- Embeddings alone do not resolve classification failures for highly imbalanced training datasets; users are warned to exercise caution with compound-class assignments.
- The public MS/MS resources that feed the pipeline total approximately 11 GB, which matters on metered connections.

## Evidence

- [methods] The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [readme] Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes.: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes."
- [abstract] alongside structure-informed embeddings (MS2DeepScore), the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote: "alongside structure-informed embeddings (MS2DeepScore), the package enables rapid and accurate prediction of "relevant" compounds"
