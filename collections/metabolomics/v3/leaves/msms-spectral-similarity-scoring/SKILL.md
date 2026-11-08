---
name: msms-spectral-similarity-scoring
description: Use when you have MS/MS spectra of an unknown metabolite (e.g. from Shimadzu
  LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) and need to identify it against a database
  whose entries may only be structurally related, not identical, compounds.
when_to_use_negative:
- The experimental spectra dataset has been removed from the repository, so the package
  cannot be run directly out of the box; do not attempt direct reproduction without
  the original MetDNA dataset or your own in-house database spectra.
- Use when the goal is scoring structural similarity from MS/MS spectra — if you only
  need exact spectrum match against a database without deep-learning similarity transfer,
  this skill is the wrong choice.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMASS
  role: Deep-learning model and ranking workflow that scores structural similarity
    of unknown metabolites against known ones from MS/MS spectra
  repo: github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that acquired external-test biomarker MS/MS spectra from the male
    infertility study used to validate identification
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument that acquired external-test biomarker MS/MS spectra from the male
    infertility study used to validate identification
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:0cb4af352826db4242a0f05f13d3745c5551c4ef3669164a3309c0e20dab2c8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msms-spectral-similarity-scoring

## Summary

DeepMASS scores structural similarity between an unknown metabolite and every metabolite in a database from their MS/MS spectra, using a deep neural network plus a candidate-ranking step to identify unknowns even when only structure-related metabolites are in the database.

## When to use

Use when you have MS/MS spectra of an unknown metabolite (e.g. from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) and need to identify it against a database whose entries may only be structurally related, not identical, compounds.

## When NOT to use

- The experimental spectra dataset has been removed from the repository, so the package cannot be run directly out of the box; do not attempt direct reproduction without the original MetDNA dataset or your own in-house database spectra.
- Use when the goal is scoring structural similarity from MS/MS spectra — if you only need exact spectrum match against a database without deep-learning similarity transfer, this skill is the wrong choice.

## Inputs

- MS/MS spectra files of unknown metabolites (placed into data/spectra)
- MS/MS spectra files of known metabolites forming the reference database (e.g. 662 KEGG compounds), also in data/spectra
- Pretrained DeepMASS model archive (model-40V.zip) unzipped into /model/40V directory

## Outputs

- Ranked list of candidate structures for each unknown metabolite, with DeepMASS structural-similarity scores against known database metabolites
- Identification calls, valid when a structure-related metabolite is available in and recovered from the database

## How to apply

Clone the hcji/DeepMASS repository, install Anaconda for Python 3.6 plus RDKit (libboost=1.65.1, boost=1.65.1, boost-cpp=1.65.1, rdkit), Keras, and IsoSpecPy==1.0.7. Download the pretrained model (model-40V.zip) and unzip it into /model/40V; place the query MS/MS spectra files into data/spectra. To train a model on an in-house database instead, put the spectra files into data/spectra and run test.py. For identification, follow example.py: the deep neural network scores structural similarity of the unknown metabolite against each known database metabolite (e.g. the 662-compound KEGG library) based on their MS/MS spectra, then a rank method picks out possible candidate structures; identification is considered valid when the top hit is a structure-related metabolite. Validate by leave-one-out cross-validation on the 662 KEGG compounds or by external tests on instrument-different spectra (Shimadzu LC-ESI-IT-TOF / Bruker Compact LC-ESI-QTOF biomarkers).

## Related tools

- **DeepMASS** (Deep-learning model and ranking workflow that scores structural similarity of unknown metabolites against known ones from MS/MS spectra) — github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument that acquired external-test biomarker MS/MS spectra from the male infertility study used to validate identification)
- **Bruker Compact LC-ESI-QTOF** (Instrument that acquired external-test biomarker MS/MS spectra from the male infertility study used to validate identification)

## Examples

```
python example.py  # rank candidate structures for spectra placed in data/spectra using the /model/40V model
```

## Evaluation signals

- Leave-one-out cross-validation on MS/MS spectra of 662 KEGG compounds succeeds when the top-ranked hit is a structure-related metabolite.
- External test on male infertility biomarkers from Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF recovers structure-related metabolites as top hits.
- The pretrained model directory /model/40V exists and contains the unzipped model-40V model files.
- Every query spectrum in data/spectra yields a ranked candidate list; unknowns whose structure-related metabolite is absent from the database should fail identification.
- RDKit, Keras, and IsoSpecPy==1.0.7 imports succeed under Python 3.6 in the configured conda environment.

## Limitations

- The experimental spectra dataset (MetDNA) has been removed at the dataset owner's request, so the package cannot be run directly; you must train your own model on an in-house database or wait for public-database spectra.
- If the MetDNA dataset is downloaded, it can only be used to reproduce the results of the DeepMASS paper; contact the owner for other uses.
- Identification of an unknown compound is valid only if its structure-related metabolite is available in the database — unknowns without any structure-related metabolite in the database are out of scope.

## Evidence

- [readme] DeepMAMS workflow: DL structural similarity prediction plus rank method for candidates: [evidence span withheld — anchor status: external_doc]
- [readme] Model setup and spectra placement: [evidence span withheld — anchor status: external_doc]
- [readme] Spectra dataset removed; cannot run directly: [evidence span withheld — anchor status: external_doc]
