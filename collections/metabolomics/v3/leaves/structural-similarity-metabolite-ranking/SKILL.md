---
name: structural-similarity-metabolite-ranking
description: Use when you have acquired tandem MS/MS spectra of unknown metabolites
  (e.g. from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) and need to identify
  them by ranking known database metabolites (e.g. KEGG) by structural similarity
  rather than exact spectral match.
when_to_use_negative:
- When you already have a high-quality experimental MS/MS spectrum of the unknown
  compound in the database, an exact spectral match should be preferred over structural-similarity
  ranking.
- When your query spectra are not from the instruments/formats the trained DeepMASS
  model was built for (e.g. spectra outside the 40V collision-energy context or an
  in-house database very different from KEGG), retraining with test.py is required
  before ranking.
- When the analysis goal is de-novo structure elucidation rather than ranking known
  database metabolites — this skill only ranks candidates that exist in the reference
  database.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: DeepMASS
  role: Provides the deep-learning model that scores structural similarity between
    the unknown metabolite and known database metabolites from their MS/MS spectra,
    plus the rank method for picking candidate structures.
  repo: github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument used to acquire the male-infertility biomarker MS/MS spectra that
    are fed to the DeepMASS scorer in the external test.
- name: Bruker Compact LC-ESI-QTOF
  role: Second instrument used to acquire male-infertility biomarker MS/MS spectra,
    allowing per-instrument evaluation of identification validity on the external
    test.
- name: KEGG
  role: Reference database of MS/MS spectra for 662 known compounds that the unknown
    metabolites are scored and ranked against.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:2d230897ffa581bc3b4c1aff73dab2a149771411325e2f4f5a207e948c274ca4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# structural-similarity-metabolite-ranking

## Summary

Rank candidate structures for an unknown metabolite by scoring structural similarity against known database metabolites from their MS/MS spectra, using the DeepMASS deep-neural-network scorer, and validate identifications via a structure-related-metabolite criterion.

## When to use

Use when you have acquired tandem MS/MS spectra of unknown metabolites (e.g. from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) and need to identify them by ranking known database metabolites (e.g. KEGG) by structural similarity rather than exact spectral match.

## When NOT to use

- When you already have a high-quality experimental MS/MS spectrum of the unknown compound in the database, an exact spectral match should be preferred over structural-similarity ranking.
- When your query spectra are not from the instruments/formats the trained DeepMASS model was built for (e.g. spectra outside the 40V collision-energy context or an in-house database very different from KEGG), retraining with test.py is required before ranking.
- When the analysis goal is de-novo structure elucidation rather than ranking known database metabolites — this skill only ranks candidates that exist in the reference database.

## Inputs

- Acquired MS/MS spectra of unknown metabolites (e.g. male-infertility biomarker spectra from Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF)
- Reference database of known metabolite MS/MS spectra (KEGG, 662 compounds)
- Trained DeepMASS deep-neural-network model (model-40V.zip placed in /model/40V directory)

## Outputs

- Ranked list of database metabolites with structural-similarity scores for each unknown spectrum
- Identification-success table per instrument (valid when a structure-related metabolite is available in the database)
- Ranked-hit and identification-success table written to a file

## How to apply

Load the experimental MS/MS spectra of the unknown metabolites (e.g. the male-infertility biomarker spectra measured on Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) and run each one through the trained DeepMASS deep-neural-network model, which scores the structural similarity between the unknown and every metabolite in the reference database (662 KEGG compounds). Rank the database metabolites by the similarity score and record the top-ranked hits with their scores; this can be done via the repository's example.py identification workflow or test.py model retraining on an in-house database. Judge identification success by the paper's criterion that an identification is valid when a structure-related metabolite of the query is available in the database, exploiting the fact that structurally related metabolites often have related MS/MS spectra. Performance is established by leave-one-out cross-validation on the KEGG set and then confirmed on an external test on the male-infertility biomarker spectra; repeat the same scoring-and-rank procedure per instrument to summarize identification performance and write the ranked-hit table to a file.

## Related tools

- **DeepMASS** (Provides the deep-learning model that scores structural similarity between the unknown metabolite and known database metabolites from their MS/MS spectra, plus the rank method for picking candidate structures.) — github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument used to acquire the male-infertility biomarker MS/MS spectra that are fed to the DeepMASS scorer in the external test.)
- **Bruker Compact LC-ESI-QTOF** (Second instrument used to acquire male-infertility biomarker MS/MS spectra, allowing per-instrument evaluation of identification validity on the external test.)
- **KEGG** (Reference database of MS/MS spectra for 662 known compounds that the unknown metabolites are scored and ranked against.)

## Examples

```
python example.py  # put spectra in data/spectra and model-40V in /model/40V, then run DeepMASS identification
```

## Evaluation signals

- Every unknown spectrum should produce a ranked list of KEGG metabolites with non-empty similarity scores; empty or NaN scores indicate a scoring failure.
- Identification is counted as valid only when the top-ranked hit is a structure-related metabolite of the query; verify per-spectrum hits against this criterion.
- Leave-one-out cross-validation on the 662 KEGG compounds should reproduce the published identification performance before trusting the external-test results.
- External-test identification performance should be summarized separately per instrument (Shimadzu LC-ESI-IT-TOF vs Bruker Compact LC-ESI-QTOF) and compared with the paper's reported per-instrument results.
- The ranked-hit and identification-success table should be written to a file with per-spectrum top hits and scores that match the article's reported output format.

## Limitations

- The experimental spectra dataset from the MetDNA owner has been removed from the repository, so the package cannot be run directly; users must train their own model on an in-house database or wait for public-database spectra to be included.
- The provided model archive (model-40V.zip) must be unzipped into /model/40V directory before use; missing or misplacing it will break the identification run.
- Identification is valid only when a structure-related metabolite is available in the database — truly novel compounds with no structurally related entry cannot be identified by this workflow.
- The workflow depends on RDKit, Keras and IsoSpecPy with specific version pins (libboost/boost/boost-cpp 1.65.1, IsoSpecPy 1.0.7), which may be hard to reproduce in modern environments.

## Evidence

- [readme] DeepMASS is a known-to-unknown metabolite identification workflow: [evidence span withheld — anchor status: external_doc]
- [readme] Model download instruction: [evidence span withheld — anchor status: not_found]
- [readme] Dataset removed, package cannot be run directly: [evidence span withheld — anchor status: external_doc]
- [readme] Take example.py as an example for metabolite identification: [evidence span withheld — anchor status: not_found]
