---
name: metabolite-candidate-ranking-kegg-database
description: Use when an experimental MS/MS spectrum of an unknown metabolite must
  be matched to a known reference library (e.g. KEGG 662-compound MS/MS spectra) and
  ranked candidates by predicted structural similarity, especially when no exact spectral
  match exists.
when_to_use_negative:
- The unknown metabolite's structure-related reference metabolite is absent from the
  KEGG library — DeepMASS cannot extend the identification beyond what a database
  entry supports, and the paper states identification is valid only when such a metabolite
  is available in the database.
- You need exact-match spectral identification against a fully populated database
  of the compound itself rather than similarity-based ranking through related structures
  — DeepMASS's value is extending the range of metabolites identifiable with an incomplete
  database, not replacing exact-match search.
- The spectra are not acquired under comparable collision-energy conditions to the
  model (the shipped model is 40V-specific), or the spectra cannot be run through
  DeepMASS's deep-neural-network scorer — ranking with a different scoring function
  would not reproduce the paper's results.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: DeepMASS
  role: Deep-neural-network-based tool that scores the structural similarity between
    an unknown metabolite's MS/MS spectrum and every metabolite in the KEGG library,
    and ranks candidates for identification.
  repo: https://github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that acquired the external-test biomarker MS/MS spectra used to
    validate candidate ranking on spectra not seen in training.
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument that acquired the external-test biomarker MS/MS spectra used to
    validate candidate ranking on spectra not seen in training.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:7a38040dbf86e3e775eb6f7feb138ef25d812ecfd3b52e5bdcfca820e7d0d039
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metabolite-candidate-ranking-kegg-database

## Summary

DeepMASS ranks candidate metabolite structures for an unknown MS/MS spectrum by scoring structural similarity against every metabolite in a KEGG reference library (662 compounds) with a deep neural network trained on MS/MS spectra, then picking top-ranked candidates; identification is considered valid only when a structure-related metabolite is present in and recovered from the database.

## When to use

Use when an experimental MS/MS spectrum of an unknown metabolite must be matched to a known reference library (e.g. KEGG 662-compound MS/MS spectra) and ranked candidates by predicted structural similarity, especially when no exact spectral match exists.

## When NOT to use

- The unknown metabolite's structure-related reference metabolite is absent from the KEGG library — DeepMASS cannot extend the identification beyond what a database entry supports, and the paper states identification is valid only when such a metabolite is available in the database.
- You need exact-match spectral identification against a fully populated database of the compound itself rather than similarity-based ranking through related structures — DeepMASS's value is extending the range of metabolites identifiable with an incomplete database, not replacing exact-match search.
- The spectra are not acquired under comparable collision-energy conditions to the model (the shipped model is 40V-specific), or the spectra cannot be run through DeepMASS's deep-neural-network scorer — ranking with a different scoring function would not reproduce the paper's results.

## Inputs

- MS/MS spectra of unknown metabolites (experimental, e.g. Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF spectra placed in data/spectra)
- KEGG reference library of MS/MS spectra for 662 compounds (database search space)
- Pre-trained DeepMASS model for 40V spectra (model-40V.zip unpacked into /model/40V)

## Outputs

- Ranked list of candidate metabolites for each unknown spectrum, ordered by DeepMASS structural-similarity score
- Per-biomarker identification-success labels (whether the structure-related metabolite is recovered as top hit), e.g. external_shimadzu_identification.csv

## How to apply

Load the unknown MS/MS spectra (e.g. the male-infertility biomarker spectra acquired on Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) and the KEGG reference library of MS/MS spectra for 662 compounds. Run DeepMASS's deep-neural-network scorer to predict the structural similarity of the unknown against every known metabolite in the library. Rank all library metabolites by the DeepMASS score and take the top-ranked candidates as the possible structures of the unknown. Treat an identification as valid only when a structure-related (true) metabolite exists in the database and is recovered as a top hit; the paper evaluated this with leave-one-out cross-validation on the 662 KEGG compounds and externally on biomarker spectra from independent instruments, so an equivalent validation should be run before trusting a new ranking.

## Related tools

- **DeepMASS** (Deep-neural-network-based tool that scores the structural similarity between an unknown metabolite's MS/MS spectrum and every metabolite in the KEGG library, and ranks candidates for identification.) — https://github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument that acquired the external-test biomarker MS/MS spectra used to validate candidate ranking on spectra not seen in training.)
- **Bruker Compact LC-ESI-QTOF** (Instrument that acquired the external-test biomarker MS/MS spectra used to validate candidate ranking on spectra not seen in training.)

## Examples

```
python example.py  # puts unknown spectra in data/spectra, loads model from /model/40V, and ranks KEGG candidates by DeepMASS structural-similarity score
```

## Evaluation signals

- Per-spectrum ranking must place the structure-related (true) metabolite as the top hit for the identification to count as valid, as scored in metric_external_shimadzu_identification.
- Leave-one-out cross-validation over the 662-compound KEGG library should reproduce the paper's identification performance before trusting a new ranking.
- The external biomarker spectra (Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF) must be identified correctly when the structure-related metabolite is present in the database.
- Output table (e.g. external_shimadzu_identification.csv) must contain, for each biomarker spectrum, ranked database metabolites with identification-success labels.
- Model directory /model/40V must contain the unzipped model-40V.zip, and data/spectra must contain the spectra files, for the workflow to run as in example.py.

## Limitations

- The experimental spectra dataset (MetDNA dataset) was removed from the repository at the owner's request, so the package cannot be run directly; users must train their own model with an in-house database or wait for public-database spectra to be included.
- Identification is only valid when a structure-related metabolite is available in the database — truly novel scaffolds without a related reference cannot be identified.
- The shipped model is specific to 40V collision-energy spectra; other energies require retraining (test.py) with an in-house database.

## Evidence

- [abstract] DeepMASS scores structural similarity of unknown metabolites against known ones using MS/MS spectra and deep neural networks: [evidence span withheld — anchor status: not_found]
- [readme] DeepMASS is a known-to-unknown metabolite identification workflow, which includes a deep-learning based model to predict structural similarity between the unknown metabolites and the known ones based on their MS/MS spectra: [evidence span withheld — anchor status: external_doc]
- [readme] Since the experimental spectra has been removed, this package cannot be run directly. You can train your own model with your in-house database.: [evidence span withheld — anchor status: external_doc]
