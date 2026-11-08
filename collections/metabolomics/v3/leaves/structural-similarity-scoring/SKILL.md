---
name: structural-similarity-scoring
description: Use when identifying an unknown metabolite whose MS/MS spectrum lacks
  an exact match in a database, but structurally related known metabolites (e.g. KEGG
  entries) exist whose spectra can anchor similarity scoring.
when_to_use_negative:
- The unknown metabolite's spectrum has an exact match in the MS/MS database — direct
  spectral matching is faster and already sufficient
- No structure-related metabolite exists in the database, since the paper states identification
  is valid only when a structure-related metabolite is available
- Spectra were acquired at collision energies other than the pretrained model's (40V),
  and no in-house database is available to retrain (the MetDNA experimental-spectra
  dataset was removed from the repo, so the package cannot be run directly out of
  the box)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepMASS
  role: Deep neural-network model that regresses structural-similarity scores between
    MS/MS spectrum pairs and ranks candidate structures for unknown metabolites
  repo: github.com/hcji/DeepMASS
- name: KEGG
  role: Source of the 662 known-compound MS/MS reference spectra used to train and
    evaluate the similarity-scoring model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:b17c220bcc438a6a792a5143b932cbe98a8b634f229d6b92603d1a421e11866f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepMASS
  - KEGG
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# structural-similarity-scoring

## Summary

DeepMASS uses a deep neural network trained on MS/MS spectrum pairs to output a structural-similarity score that ranks database candidates for an unknown metabolite, extending identification beyond exact spectral matches in databases like KEGG.

## When to use

Use when identifying an unknown metabolite whose MS/MS spectrum lacks an exact match in a database, but structurally related known metabolites (e.g. KEGG entries) exist whose spectra can anchor similarity scoring.

## When NOT to use

- The unknown metabolite's spectrum has an exact match in the MS/MS database — direct spectral matching is faster and already sufficient
- No structure-related metabolite exists in the database, since the paper states identification is valid only when a structure-related metabolite is available
- Spectra were acquired at collision energies other than the pretrained model's (40V), and no in-house database is available to retrain (the MetDNA experimental-spectra dataset was removed from the repo, so the package cannot be run directly out of the box)

## Inputs

- MS/MS spectra of the unknown metabolite (peak list/intensity vector, e.g. placed in data/spectra)
- MS/MS spectra of known database metabolites used as the reference/candidate pool (e.g. 662 KEGG compound spectra)
- Pretrained DeepMASS model weights (e.g. model-40V.zip unpacked into /model/40V)

## Outputs

- Ranked list of candidate structures per unknown metabolite, ordered by predicted structural-similarity score
- Structural-similarity score table for each (unknown, candidate) spectrum pair
- Trained DeepMASS model weights when training a custom model (test.py) on an in-house database

## How to apply

Compile MS/MS spectra of known reference compounds (the paper used 662 KEGG compounds) into spectrum-pair examples labeled with structural similarity, preprocess each spectrum into the peak-intensity feature representation consumed by the network, and train a deep neural network (TensorFlow/Keras) to regress a structural-similarity score between the two spectra. Place the pretrained 40V model weights under /model/40V and the unknown spectra into data/spectra, then run the identification workflow (example.py) which scores the unknown against every known candidate and ranks candidates by similarity. Validation follows the paper's leave-one-out cross-validation design: hold out one compound, score it against the remaining database, and accept the model if a structure-related database entry is retrieved for the unknown. A lower collision-energy model (40V) is provided, so spectra should match the acquisition conditions used to train that model.

## Related tools

- **DeepMASS** (Deep neural-network model that regresses structural-similarity scores between MS/MS spectrum pairs and ranks candidate structures for unknown metabolites) — github.com/hcji/DeepMASS
- **KEGG** (Source of the 662 known-compound MS/MS reference spectra used to train and evaluate the similarity-scoring model)

## Examples

```
python example.py  # scores spectra in data/spectra against the pretrained /model/40V DeepMASS model and ranks candidate structures
```

## Evaluation signals

- Leave-one-out cross-validation on the 662 KEGG compounds retrieves a structure-related database entry for each held-out unknown
- Every evaluated (unknown, candidate) spectrum pair receives a similarity score in the output table — no missing scores
- Ranking places structure-related metabolites above unrelated ones for the unknown's candidate list
- The pretrained model directory /model/40V exists and matches the collision energy of the input spectra

## Limitations

- The experimental spectra dataset (MetDNA dataset) was removed from the repository at the owner's request, so the package cannot be run directly; users must train their own model on an in-house database or wait for public-database spectra to be included
- The downloaded pretrained model must be manually placed into /model/40V — installation requires unzipping model-40V.zip from ResearchGate
- Dependencies are pinned (libboost/boost/boost-cpp 1.65.1, rdkit via conda, keras, IsoSpecPy 1.0.7); version drift can break the RDKit-based structural-similarity pipeline
- Identification is only valid if a structure-related metabolite is present in the database — unrelated unknowns cannot be identified
- Discussion-section details are missing from the source document

## Evidence

- [readme] DeepMASS includes a rank method for picking out possible candidate structures: [evidence span withheld — anchor status: external_doc]
- [readme] Installation requires unzipping the pretrained model into /model/40V: [evidence span withheld — anchor status: not_found]
