---
name: deep-neural-network-similarity-regression
description: Use when you need to identify unknown metabolites whose MS/MS spectra
  are not in a database, by scoring structural similarity against known reference
  compounds instead of direct spectral matching.
when_to_use_negative:
- The unknown's exact MS/MS spectrum is already present in a curated database (e.g.
  spectral match suffices) — DeepMASS extends rather than replaces direct identification.
- No structure-related metabolite is available in the database for the unknown, which
  the paper states invalidates this identification route.
- The experimental spectra dataset (MetDNA) is missing from the repo — since 2019/04/29
  it was removed and the package cannot be run directly; you must train your own model
  on an in-house database instead.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMASS
  role: Python workflow providing the deep-learning model that predicts structural
    similarity between unknown and known metabolites from MS/MS spectra, plus a rank
    method for picking candidate structures.
  repo: https://github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument used to acquire MS/MS spectra of male infertility biomarkers for
    the external test set.
- name: Bruker Compact LC-ESI-QTOF
  role: Second instrument platform for acquiring the external biomarker MS/MS spectra.
- name: KEGG
  role: Source of the 662 reference-compound MS/MS spectra used for leave-one-out
    cross-validation and model training.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:e8dc230cf0d2c792e0afa893b4b6bc7a32a1d959eeb9d5ffb95dfd41363c3334
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# deep-neural-network-similarity-regression

## Summary

Train a TensorFlow/Keras deep neural network on paired MS/MS spectra (unknown vs known metabolite) to regress a structural-similarity score, then rank database candidates by that score for unknown metabolite identification (DeepMASS).

## When to use

Use when you need to identify unknown metabolites whose MS/MS spectra are not in a database, by scoring structural similarity against known reference compounds instead of direct spectral matching.

## When NOT to use

- The unknown's exact MS/MS spectrum is already present in a curated database (e.g. spectral match suffices) — DeepMASS extends rather than replaces direct identification.
- No structure-related metabolite is available in the database for the unknown, which the paper states invalidates this identification route.
- The experimental spectra dataset (MetDNA) is missing from the repo — since 2019/04/29 it was removed and the package cannot be run directly; you must train your own model on an in-house database instead.

## Inputs

- MS/MS spectra (peak-intensity lists) of unknown metabolites and known reference compounds, e.g. from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF acquisitions
- Structural-similarity labels computed from chemical structures of the paired known metabolites
- Pre-trained model weights (model-40V.zip unpacked into /model/40V) for scoring without retraining
- In-house or public spectral database (e.g. KEGG MS/MS spectra of 662 compounds) for candidate ranking

## Outputs

- Structural-similarity score for each unknown–candidate spectrum pair
- Ranked list of candidate structures for each unknown metabolite
- Trained model weights and similarity_scores.csv table for spectrum pairs

## How to apply

Collect MS/MS spectra of known reference compounds (e.g. the 662 KEGG compounds used to evaluate DeepMASS) and build paired (unknown, candidate) spectrum examples labeled with structural-similarity values computed from their chemical structures. Preprocess each spectrum into the peak-intensity feature representation the network consumes, then train a deep neural network in TensorFlow/Keras to regress the similarity score on these spectrum pairs. Apply the trained network to score structural similarity for each unknown against every candidate in the database and rank candidates by descending score; identification is considered valid when a structurally related metabolite exists in the database. Evaluate with leave-one-out cross-validation on the reference set and an external biological test set (male infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF). Save model weights (e.g. model-40V.zip unzipped into /model/40V) and write the similarity-score table to similarity_scores.csv.

## Related tools

- **DeepMASS** (Python workflow providing the deep-learning model that predicts structural similarity between unknown and known metabolites from MS/MS spectra, plus a rank method for picking candidate structures.) — https://github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument used to acquire MS/MS spectra of male infertility biomarkers for the external test set.)
- **Bruker Compact LC-ESI-QTOF** (Second instrument platform for acquiring the external biomarker MS/MS spectra.)
- **KEGG** (Source of the 662 reference-compound MS/MS spectra used for leave-one-out cross-validation and model training.)

## Examples

```
python example.py  # place unknown spectra in data/spectra and pre-trained weights in /model/40V
```

## Evaluation signals

- Leave-one-out cross-validation on the 662 KEGG compounds retrieves a structure-related database entry for each held-out unknown.
- The similarity_scores.csv table covers every evaluated spectrum pair and ranks a structurally related candidate at the top for leave-one-out unknowns.
- External validation on male infertility biomarkers on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF yields plausible structure-related identifications.
- Model weights load correctly from /model/40V (model-40V.zip) and the network produces scores for arbitrary unknown–candidate pairs without NaNs.

## Limitations

- The MetDNA experimental spectra dataset was removed from the repository at the owner's request, so the package cannot be run directly; users must train their own model on an in-house database or wait for public spectra to be included.
- Identification is only valid when a structure-related metabolite is available in the database.
- Depends on specific package versions: Anaconda Python 3.6, RDKit with libboost/boost/boost-cpp 1.65.1, Keras, and IsoSpecPy == 1.0.7.

## Evidence

- [abstract] DeepMASS scores structural similarity of unknown metabolites against known ones using MS/MS spectra and deep neural networks: "which can score the structural similarity of unknown metabolite against the known one with MS/MS spectra and deep neural networks"
- [abstract] Leave-one-out evaluation scope: "We evaluated DeepMASS with leave-one-out cross-validation on MS/MS spectra of 662 compounds in KEGG"
- [abstract] Validity condition for identification: "Results show that the identification of unknown compound is valid if its structure-related metabolite is available in the database"
- [readme] Workflow description: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset removal limitation: [evidence span withheld — anchor status: external_doc]
