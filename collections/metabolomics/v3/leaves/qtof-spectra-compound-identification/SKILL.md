---
name: qtof-spectra-compound-identification
description: Use when an untargeted LC-MS/MS run (e.g. Shimadzu LC-ESI-IT-TOF or Bruker
  Compact LC-ESI-QTOF) yields MS/MS spectra of unknown metabolites that must be matched
  to database compounds via structural-similarity ranking, not exact spectral library
  hits.
when_to_use_negative:
- When the unknown compound's exact spectrum already exists in a public spectral library,
  a direct library search gives a definitive annotation and DeepMASS's structural-similarity
  route is unnecessary.
- When the query metabolite has no structure-related metabolite in the reference database,
  DeepMASS identification is not valid by the paper's own criterion and other methods
  should be sought.
- When the analysis goal is quantification or feature detection rather than structural
  annotation of unknowns.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DeepMASS
  role: Deep-neural-network pipeline that scores structural similarity of unknown
    vs. known metabolites from MS/MS spectra and ranks candidate structures
  repo: github.com/hcji/DeepMASS
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument used to acquire the external-test MS/MS spectra of male-infertility
    biomarker unknowns
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that generated the training/LOO-validation spectra and complementary
    external-test spectra
- name: KEGG
  role: Reference spectral database of 662 compounds providing the candidate known
    structures
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:678865d68be1c19c9a71c22d7291f0c529bcf419f64bbd08b7d0fdc6b98756a0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# qtof-spectra-compound-identification

## Summary

DeepMASS structural-similarity scoring identifies unknown metabolites from QTOF MS/MS spectra (e.g. male-infertility biomarkers on a Bruker Compact LC-ESI-QTOF) by ranking database metabolites with a deep neural network, remaining valid when a structure-related metabolite exists in the database (KEGG).

## When to use

Use when an untargeted LC-MS/MS run (e.g. Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) yields MS/MS spectra of unknown metabolites that must be matched to database compounds via structural-similarity ranking, not exact spectral library hits.

## When NOT to use

- When the unknown compound's exact spectrum already exists in a public spectral library, a direct library search gives a definitive annotation and DeepMASS's structural-similarity route is unnecessary.
- When the query metabolite has no structure-related metabolite in the reference database, DeepMASS identification is not valid by the paper's own criterion and other methods should be sought.
- When the analysis goal is quantification or feature detection rather than structural annotation of unknowns.

## Inputs

- MS/MS spectra of unknown metabolites from a QTOF (or IT-TOF) instrument, as peak-list spectrum files placed in data/spectra
- Reference MS/MS spectral database (KEGG compounds) used to train the model and serve as candidate structures
- Pretrained DeepMASS model weights (model-40V)

## Outputs

- Ranked list of candidate database metabolites with structural-similarity scores for each unknown spectrum
- Identification-success table per instrument (valid when a structure-related metabolite is available in the database)

## How to apply

Collect the unknown compounds' MS/MS spectra (peak lists) acquired on the LC-ESI-QTOF (e.g. male-infertility biomarker spectra) and place them in data/spectra. Run the trained DeepMASS deep-neural-network model (model-40V, trained on MS/MS spectra of 662 KEGG compounds) to score each spectrum against every known database metabolite, producing a structural-similarity score per pair. Rank the database metabolites by that score and record the top-ranked hits for every spectrum. Judge identification success using the paper's criterion: identification is valid only if a structure-related metabolite of the query exists in the database; report the ranked-hit and identification-success table per instrument. Rationale: structurally related metabolites often have related MS/MS spectra, so similarity scoring extends the identification range of existing MS/MS databases beyond exact library matches.

## Related tools

- **DeepMASS** (Deep-neural-network pipeline that scores structural similarity of unknown vs. known metabolites from MS/MS spectra and ranks candidate structures) — github.com/hcji/DeepMASS
- **Bruker Compact LC-ESI-QTOF** (Instrument used to acquire the external-test MS/MS spectra of male-infertility biomarker unknowns)
- **Shimadzu LC-ESI-IT-TOF** (Instrument that generated the training/LOO-validation spectra and complementary external-test spectra)
- **KEGG** (Reference spectral database of 662 compounds providing the candidate known structures)

## Examples

```
git clone https://github.com/hcji/DeepMASS; unzip model-40V.zip -d DeepMASS/model/40V; cp unknown_spectra/* DeepMASS/data/spectra/; cd DeepMASS && python example.py
```

## Evaluation signals

- Every unknown spectrum yields a ranked candidate list whose top hits can be compared against the paper's criterion — identification counts as valid only when a structure-related metabolite of the query is present in the database.
- Leave-one-out cross-validation on the 662 KEGG compounds reproduces the reported DeepMASS ranking performance before applying the model to external-test spectra.
- The output table lists, for each spectrum, top-ranked hits with their scores, and can be split per instrument (Bruker Compact LC-ESI-QTOF vs. Shimadzu LC-ESI-IT-TOF) for comparability.
- Model files must sit under /model/40V (model-40V zip unzipped into the /model/40V directory) before scoring.
- Candidate scores should be reproducible: re-running the trained model on the same spectra must give the same top-ranked metabolites.

## Limitations

- Identification is only valid when a structure-related metabolite exists in the database; structurally isolated unknowns cannot be identified by similarity scoring.
- The original experimental spectra (MetDNA dataset) were removed from the repo at the owner's request, so the package cannot be run directly and users must train their own model on in-house spectra or wait for public-database spectra.
- Model training depends on specific library versions (libboost/boost/boost-cpp 1.65.1, RDKit, Keras, IsoSpecPy 1.0.7) and Anaconda Python 3.6; environment drift can break reproduction.
- Model weights are tied to the 40 V collision-energy training set (model-40V); spectra at other collision energies may not score reliably.

## Evidence

- [readme] The model archive must be unzipped into the /model/40V directory for the workflow to run.: [evidence span withheld — anchor status: not_found]
- [readme] Spectra files for identification or training are placed in data/spectra.: [evidence span withheld — anchor status: external_doc]
- [readme] example.py is the reference invocation for unknown-metabolite identification.: [evidence span withheld — anchor status: external_doc]
