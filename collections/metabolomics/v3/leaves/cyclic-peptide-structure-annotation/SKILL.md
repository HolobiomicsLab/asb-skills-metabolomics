---
name: cyclic-peptide-structure-annotation
description: Use when you have 1D/2D NMR (optionally LC-MS/MS) spectra of purified
  or fractionated natural-product compounds of unknown structural class — e.g. cyclic
  peptides or swinholide-like macrolides — and need programmatic class-level annotation
  beyond manual characterization.
when_to_use_negative:
- Input is already a curated natural-product class table or known structures — use
  the /classify?smiles=<> API on known SMILES instead of re-running spectral classification
- The question is general molecular networking / dereplication of crude MS2 spectra
  rather than NMR-based class annotation of isolated compounds — molecular networking
  covers MS2-level prioritization instead
- Compounds lack NMR data entirely and only MS2 spectra are available — SMART 2.0
  is NMR-based and cannot classify from MS/MS alone
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: SMART 2.0
  role: NMR-based convolutional neural network classifier that produces predicted
    natural-product class assignments for cyclic-peptide spectra
- name: NP Classifier
  role: Dockerized classification server for programmatically classifying natural
    products from SMILES strings via the /classify API
  repo: https://github.com/mwang87/NP-Classifier
- name: molecular networking
  role: MS2-based molecular networking used alongside SMART 2.0 to prioritize and
    guide fractionation
- name: docker-compose
  role: Brings up the local NP Classifier server deployment (make server-compose)
    after model download and conversion
  repo: https://github.com/mwang87/NP-Classifier
- name: tensorflow 2.3.0
  role: Required to convert the keras models into HDF5 TF2 models before serving
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:24793c58fd94ed8bac9535b7e73b53a1c4a397077286fdb5b058146ab4392da2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP Classifier
  - SMART 2.0
  - docker-compose
  - molecular networking
  - tensorflow 2.3.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cyclic-peptide-structure-annotation

## Summary

Rapidly assign natural-product class annotations to structurally novel cyclic peptides by running their spectra through SMART 2.0, an NMR-based convolutional neural network (Small Molecule Accurate Recognition Technology), and cross-checking assignments against supporting 1D/2D NMR and LC-MS2 evidence.

## When to use

Use when you have 1D/2D NMR (optionally LC-MS/MS) spectra of purified or fractionated natural-product compounds of unknown structural class — e.g. cyclic peptides or swinholide-like macrolides — and need programmatic class-level annotation beyond manual characterization.

## When NOT to use

- Input is already a curated natural-product class table or known structures — use the /classify?smiles=<> API on known SMILES instead of re-running spectral classification
- The question is general molecular networking / dereplication of crude MS2 spectra rather than NMR-based class annotation of isolated compounds — molecular networking covers MS2-level prioritization instead
- Compounds lack NMR data entirely and only MS2 spectra are available — SMART 2.0 is NMR-based and cannot classify from MS/MS alone

## Inputs

- 1D/2D NMR spectra of purified or fractionated natural-product compounds
- LC-MS/MS spectra supporting structural confirmation
- Optionally, SMILES strings of candidate structures for NP Classifier API classification
- Cyclic-peptide case-study spectra from the article's second case study

## Outputs

- Per-compound SMART 2.0 natural-product class probabilities
- Top SMART 2.0 class assignment for each compound
- Annotation/characterization table written to cyclic_peptide_classifications.csv with columns: compound name, SMART 2.0 class assignment, supporting spectral evidence

## How to apply

Collect the compound spectra (1D/2D NMR and LC-MS2) from the relevant case study and run each through the SMART 2.0 NMR-based convolutional neural network classifier to obtain predicted natural-product class probabilities. Record the top SMART 2.0 class assignment per compound, then confirm each annotation by cross-checking against the supporting spectral evidence described in the case study. Deploy the accompanying models locally with docker and docker-compose (docker network create nginx-net; make server-compose), after downloading models via Classifier/models_folder/models and converting with tensorflow 2.3.0. Verify deployment by checking the /model/metadata endpoint and that input layer names are 'input_2048' and 'input_4096' with output layer 'output'. For SMILES-based programmatic classification of candidate structures, query the NP Classifier API. Assemble results into a per-compound table (compound name, SMART 2.0 class assignment, supporting spectral evidence) written to cyclic_peptide_classifications.csv.

## Related tools

- **SMART 2.0** (NMR-based convolutional neural network classifier that produces predicted natural-product class assignments for cyclic-peptide spectra)
- **NP Classifier** (Dockerized classification server for programmatically classifying natural products from SMILES strings via the /classify API) — https://github.com/mwang87/NP-Classifier
- **molecular networking** (MS2-based molecular networking used alongside SMART 2.0 to prioritize and guide fractionation)
- **docker-compose** (Brings up the local NP Classifier server deployment (make server-compose) after model download and conversion) — https://github.com/mwang87/NP-Classifier
- **tensorflow 2.3.0** (Required to convert the keras models into HDF5 TF2 models before serving)

## Examples

```
curl "http://localhost:8501/classify?smiles=CC(=O)Oc1ccccc1C(=O)O&cached=true"
```

## Evaluation signals

- Each compound's top SMART 2.0 class assignment is corroborated by the supporting 1D/2D NMR and LC-MS2 spectral evidence (as in the symplocolide A confirmation)
- Deployed model metadata endpoint /model/metadata returns input layer names 'input_2048' and 'input_4096' and output layer name 'output'
- cyclic_peptide_classifications.csv contains one row per compound with non-empty compound name, class assignment, and spectral-evidence columns
- Per-compound class probability vectors sum to a valid distribution and the top assignment matches the benchmark comparison against the 'atomic sort' method
- Server responds successfully to /classify?smiles=<> requests, and cached=true returns results consistent with the uncached path

## Limitations

- Benchmarked against the 'atomic sort' method, but no changelog was found in the repository, complicating version tracking of model behavior
- If the model input layer names change, the code must be updated accordingly — deployment breaks silently otherwise
- Annotation is class-level (natural-product class probabilities), not full structure elucidation; planar-structure confirmation still relies on 1D/2D NMR and LC-MS2

## Evidence

- [abstract] SMART 2.0 first application: "first application of the novel NMR-based machine learning tool ‘Small Molecule Accurate Recognition Technology’ (SMART 2.0)"
- [abstract] symplocolide A confirmation: "The planar structure of symplocolide A was confirmed to be a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS 2 analysis"
- [abstract] fractionation prioritization: "prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS 2 -based molecular networking"

Output layer's name should be "output""
