---
name: msms-spectrum-pair-feature-encoding
description: Use when MS/MS spectra of an unknown metabolite must be converted into
  paired feature vectors against database candidate structures so a trained similarity-scoring
  model can rank candidates.
when_to_use_negative:
- The unknown metabolite has no structure-related metabolite in the database — the
  paper states identification is only valid when a structure-related metabolite is
  available
- Spectra were acquired at collision energies incompatible with the available model
  (only the 40V model is distributed)
- The goal is de novo structure elucidation rather than ranking known database candidates
  by structural similarity
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: DeepMASS
  role: Provides the deep-learning spectrum-pair model that consumes peak-intensity
    features and outputs structural-similarity scores; repo supplies example.py for
    identification and test.py for training on in-house spectra
  repo: github.com/hcji/DeepMASS
- name: RDKit
  role: Conda dependency used to handle candidate metabolite structures and structural-similarity
    labels for the spectrum pairs
- name: Keras
  role: Framework in which the deep neural network regressing the structural-similarity
    score from encoded spectrum pairs is trained and applied
- name: IsoSpecPy
  role: Pip dependency (IsoSpecPy == 1.0.7) supporting isotope-pattern handling in
    the DeepMASS pipeline
- name: KEGG
  role: Source of the MS/MS spectra of the 662 reference compounds used to build and
    evaluate the spectrum-pair training set
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:b2ea3a5657d05b21fcaa3127707fba4ef7cc5438e17de3e5842653a485adf454
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepMASS
  - IsoSpecPy
  - KEGG
  - Keras
  - RDKit
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# msms-spectrum-pair-feature-encoding

## Summary

Encodes MS/MS spectra of unknown and known metabolites into peak-intensity feature representations consumed as spectrum pairs by the DeepMASS deep neural network, enabling regression of a structural-similarity score for candidate ranking.

## When to use

Use when MS/MS spectra of an unknown metabolite must be converted into paired feature vectors against database candidate structures so a trained similarity-scoring model can rank candidates.

## When NOT to use

- The unknown metabolite has no structure-related metabolite in the database — the paper states identification is only valid when a structure-related metabolite is available
- Spectra were acquired at collision energies incompatible with the available model (only the 40V model is distributed)
- The goal is de novo structure elucidation rather than ranking known database candidates by structural similarity

## Inputs

- MS/MS spectra files placed in data/spectra (e.g., spectra of the 662 KEGG reference compounds or in-house database spectra)
- Known candidate metabolite structures from an in-house or reference database (e.g., KEGG entries)
- Pretrained DeepMASS model weights (model-40V.zip unzipped into /model/40V)

## Outputs

- Peak-intensity feature representations for each (unknown, candidate) spectrum pair
- Structural-similarity score table for spectrum pairs (similarity_scores.csv) used to rank database candidates

## How to apply

Load MS/MS spectra of the 662 KEGG reference compounds into paired (unknown, candidate) spectrum-pair examples labeled with structural similarity, computed from the known structures (RDKit is the declared dependency for structure handling). Preprocess each spectrum into the peak-intensity feature representation consumed by the DeepMASS network, keeping collision-energy context consistent (the released pretrained model is a 40V model placed in /model/40V). Train a Keras-based deep neural network on these spectrum pairs to regress the structural-similarity score, then apply the trained network to held-out pairs and rank database candidates for each unknown. The rationale is that structurally related metabolites often have related MS/MS spectra, so spectrum-pair encoding transfers structural relatedness from known to unknown compounds.

## Related tools

- **DeepMASS** (Provides the deep-learning spectrum-pair model that consumes peak-intensity features and outputs structural-similarity scores; repo supplies example.py for identification and test.py for training on in-house spectra) — github.com/hcji/DeepMASS
- **RDKit** (Conda dependency used to handle candidate metabolite structures and structural-similarity labels for the spectrum pairs)
- **Keras** (Framework in which the deep neural network regressing the structural-similarity score from encoded spectrum pairs is trained and applied)
- **IsoSpecPy** (Pip dependency (IsoSpecPy == 1.0.7) supporting isotope-pattern handling in the DeepMASS pipeline)
- **KEGG** (Source of the MS/MS spectra of the 662 reference compounds used to build and evaluate the spectrum-pair training set)

## Examples

```
git clone https://github.com/hcji/DeepMASS; cp *.ms data/spectra/; python example.py
```

## Evaluation signals

- Leave-one-out cross-validation on the MS/MS spectra of the 662 KEGG compounds retrieves a structure-related database entry for each unknown
- The similarity-score table covers all evaluated spectrum pairs with no missing entries
- Candidate ranking places a structurally related metabolite at the top for known unknowns, consistent with the paper's reported leave-one-out behavior
- The trained model is validated on external biomarker spectra from the male infertility study measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF instruments
- Model weights load from /model/40V and inference runs through example.py without dependency errors (RDKit, Keras, IsoSpecPy==1.0.7)

## Limitations

- The MetDNA experimental spectra dataset was removed from the repository at the owner's request, so the package cannot be run directly out of the box; users must train their own model with an in-house database or wait for public spectra to be included
- Identification is only valid when a structure-related metabolite is available in the database
- Only the 40V collision-energy pretrained model is distributed via ResearchGate, constraining applicable spectra
- Discussion section text is absent from the source document, so detailed parameter tuning guidance is unavailable

## Evidence

- [readme] Model placement instruction: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset removal note: [evidence span withheld — anchor status: external_doc]
