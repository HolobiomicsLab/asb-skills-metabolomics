---
name: spectral-data-preprocessing
description: Use when centroided MS/MS spectra (peak m/z–intensity lists, e.g. 40V
  collision-energy spectra) must be converted into network-ready peak-intensity feature
  vectors for known-to-unknown metabolite identification and candidate ranking.
when_to_use_negative:
- The unknown has no structure-related metabolite in the database — the paper states
  identification is only valid when a structure-related metabolite is available
- The task is direct library spectral matching against curated MS/MS databases rather
  than extending identification range via known-to-unknown similarity scoring
- Only the pretrained model is needed but spectra were acquired at a collision energy
  other than 40V without retraining on matched in-house spectra
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DeepMASS
  role: Provides the preprocessing pipeline and Keras deep-learning model that consume
    the preprocessed peak-intensity spectra and output structural-similarity scores
  repo: github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Source instrument for the raw MS/MS spectra that are preprocessed into feature
    representation
- name: Bruker Compact LC-ESI-QTOF
  role: Source instrument for the raw MS/MS spectra that are preprocessed into feature
    representation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:ba46896fd5f53eee58744fc1cb44d765cac99be524618e100f51de9b8067df3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  - github.com/hcji/DeepMASS
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spectral-data-preprocessing

## Summary

Converts raw MS/MS peak lists (e.g., from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF runs) into the peak-intensity feature representation consumed by DeepMASS's Keras deep neural network, so that unknown metabolites can be scored for structural similarity against known database candidates such as the 662 KEGG compounds.

## When to use

Use when centroided MS/MS spectra (peak m/z–intensity lists, e.g. 40V collision-energy spectra) must be converted into network-ready peak-intensity feature vectors for known-to-unknown metabolite identification and candidate ranking.

## When NOT to use

- The unknown has no structure-related metabolite in the database — the paper states identification is only valid when a structure-related metabolite is available
- The task is direct library spectral matching against curated MS/MS databases rather than extending identification range via known-to-unknown similarity scoring
- Only the pretrained model is needed but spectra were acquired at a collision energy other than 40V without retraining on matched in-house spectra

## Inputs

- Centroided MS/MS spectra (peak m/z and intensity lists) of reference database compounds (662 KEGG compounds in the paper)
- Centroided MS/MS spectra of unknown metabolites, e.g. biomarkers from a male infertility study measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF
- Structural information / structural-similarity labels for reference compounds (RDKit-parseable structures)

## Outputs

- Peak-intensity feature representation per MS/MS spectrum, consumable by the DeepMASS deep neural network
- Trained model weights (e.g. saved under /model/40V for 40V spectra)
- similarity_scores.csv — structural-similarity scores for all evaluated spectrum pairs, used to rank database candidates per unknown

## How to apply

Place the MS/MS spectra files for the reference set (in the paper, the 662 KEGG compounds) into the repo's data/spectra directory and preprocess each spectrum into the peak-intensity feature representation the DeepMASS network expects. Build paired (unknown, candidate) spectrum examples with structural-similarity labels (RDKit is a project dependency, used for structure handling), respecting the collision energy of the model — the shipped pretrained model is for 40V and must be unzipped into /model/40V. Train the Keras network (installable via 'conda install keras'; IsoSpecPy 1.0.7 is required for isotope handling) on these spectrum pairs to regress the structural-similarity score, or reuse the pretrained 40V weights. Apply the trained network to score similarity of each unknown's spectrum against every database candidate spectrum, then rank candidates by score and write the pair scores to similarity_scores.csv. The rationale is that structurally related metabolites often have related MS/MS spectra, so a learned similarity score extends identification beyond exact spectral matches in existing databases.

## Related tools

- **DeepMASS** (Provides the preprocessing pipeline and Keras deep-learning model that consume the preprocessed peak-intensity spectra and output structural-similarity scores) — github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Source instrument for the raw MS/MS spectra that are preprocessed into feature representation)
- **Bruker Compact LC-ESI-QTOF** (Source instrument for the raw MS/MS spectra that are preprocessed into feature representation)

## Examples

```
python example.py
```

## Evaluation signals

- similarity_scores.csv contains a similarity score for every evaluated spectrum pair (no missing pairs)
- Leave-one-out cross-validation on the 662 KEGG compounds retrieves a structure-related database entry as the ranked candidate for each unknown
- Every preprocessed spectrum yields a finite, fixed-length peak-intensity feature vector accepted by the Keras network without error
- Trained weights are saved to the model directory (e.g. /model/40V) and the external male-infertility biomarker spectra score without preprocessing failures
- Scores behave consistently with the paper: structurally related metabolites receive higher similarity scores than unrelated ones

## Limitations

- As of the 2019/05/13 README note, the experimental MetDNA spectra were removed at the owner's request, so the package cannot be run directly out of the box; users must train their own model with an in-house database
- The pretrained model is tied to 40V collision energy; spectra at other collision energies require retraining on matched data
- Identification of an unknown is only valid if a structure-related metabolite exists in the database

## Evidence

- [readme] README specifies where spectra files go and how to train a model on an in-house database: [evidence span withheld — anchor status: not_found]
- [readme] README warns the shipped experimental dataset was removed, so out-of-the-box preprocessing is not directly runnable: [evidence span withheld — anchor status: external_doc]
