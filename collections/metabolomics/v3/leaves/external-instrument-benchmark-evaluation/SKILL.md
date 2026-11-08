---
name: external-instrument-benchmark-evaluation
description: Use when validating a spectral-similarity or metabolite-identification
  model on spectra from an independent instrument or study (external test set), e.g.
  biomarker spectra from a male infertility study measured on a Bruker Compact LC-ESI-QTOF
  after training on IT-TOF data.
when_to_use_negative:
- The goal is exact-structure identification with an in-house standard library — DeepMASS's
  criterion is validity via a structure-related metabolite, not exact matching.
- The spectra are from the same instrument and study used for leave-one-out cross-validation;
  use the internal CV evaluation instead of an external instrument benchmark.
- No trained DeepMASS model or structural database is available and you cannot train
  one from in-house spectra (the README notes the original experimental dataset was
  removed, so the package cannot be run directly without your own spectra).
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DeepMASS
  role: Deep-neural-network structural-similarity scorer that ranks known database
    metabolites against each unknown MS/MS spectrum during the external instrument
    test
  repo: github.com/hcji/DeepMASS
- name: Bruker Compact LC-ESI-QTOF
  role: External-test instrument providing biomarker MS/MS spectra used to challenge
    the model trained on IT-TOF data
- name: Shimadzu LC-ESI-IT-TOF
  role: Training-instrument counterpart whose spectra were used for leave-one-out
    cross-validation; contrast target for the external QTOF benchmark
- name: KEGG
  role: Source of the 662-compound MS/MS spectral database used for training and as
    the candidate metabolite pool during external identification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:aa8675a96eaa8cc55aa17f88f5325ae950237280320f5d2bca7a7129aa13ab17
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# external-instrument-benchmark-evaluation

## Summary

Runs a trained DeepMASS structural-similarity scorer on MS/MS spectra acquired on an instrument not used for model training (e.g. Bruker Compact LC-ESI-QTOF biomarker spectra) and assesses whether identification of unknown compounds is valid when a structure-related metabolite exists in the database.

## When to use

Use when validating a spectral-similarity or metabolite-identification model on spectra from an independent instrument or study (external test set), e.g. biomarker spectra from a male infertility study measured on a Bruker Compact LC-ESI-QTOF after training on IT-TOF data.

## When NOT to use

- The goal is exact-structure identification with an in-house standard library — DeepMASS's criterion is validity via a structure-related metabolite, not exact matching.
- The spectra are from the same instrument and study used for leave-one-out cross-validation; use the internal CV evaluation instead of an external instrument benchmark.
- No trained DeepMASS model or structural database is available and you cannot train one from in-house spectra (the README notes the original experimental dataset was removed, so the package cannot be run directly without your own spectra).

## Inputs

- External-test MS/MS spectra of biomarkers from a male infertility study (e.g. Bruker Compact LC-ESI-QTOF raw or centroided spectra files)
- Trained DeepMASS model (e.g. the 40V model package) and its structural database of known metabolite MS/MS spectra

## Outputs

- Ranked candidate-metabolite hit list with structural-similarity scores for each external spectrum
- Per-instrument identification-success summary table (valid/invalid per query, aggregate rate)

## How to apply

Load the external-test MS/MS spectra (biomarkers from the male infertility study measured on the Bruker Compact LC-ESI-QTOF) and run each through the trained DeepMAMS deep-neural-network scorer to rank database metabolites by structural-similarity score. Record the top-ranked hits and their scores for every spectrum. Apply the paper's success criterion: an identification of an unknown compound is valid if its structure-related metabolite is available in the database — not exact structure matching. Summarize the per-instrument identification performance and export the ranked-hit and identification-success table. Rationale: structurally related metabolites often have related MS/MS spectra, so the external test checks whether the learned spectral-to-structural-similarity mapping generalizes across instruments rather than memorizing training spectra.

## Related tools

- **DeepMASS** (Deep-neural-network structural-similarity scorer that ranks known database metabolites against each unknown MS/MS spectrum during the external instrument test) — github.com/hcji/DeepMASS
- **Bruker Compact LC-ESI-QTOF** (External-test instrument providing biomarker MS/MS spectra used to challenge the model trained on IT-TOF data)
- **Shimadzu LC-ESI-IT-TOF** (Training-instrument counterpart whose spectra were used for leave-one-out cross-validation; contrast target for the external QTOF benchmark)
- **KEGG** (Source of the 662-compound MS/MS spectral database used for training and as the candidate metabolite pool during external identification)

## Examples

```
python example.py  # place external-test spectra in data/spectra and the 40V model in /model/40V per README
```

## Evaluation signals

- For every external-test spectrum the scorer returns a complete ranked hit list; no query is left unranked.
- Identification success is recorded using the paper's criterion — a query counts as validly identified only when a structure-related metabolite is present in the database, so the success table must encode this relation rather than exact-match identity.
- Reported external performance must be attributable to the Bruker Compact LC-ESI-QTOF instrument specifically, distinguishing it from the Shimadzu LC-ESI-IT-TOF leave-one-out CV results.
- The exported summary should contain both the ranked hit scores and the binary valid/invalid identification outcome so the per-instrument rate can be recomputed from the table.
- Scores should behave consistently across the external set — the DeepMASS abstract claims the approach 'provides an effective approach to extend the identification range of metabolites for existing MS/MS databases', so a near-zero success rate on structure-related targets would signal a broken model or mismatched spectra.

## Limitations

- The external test set (male infertility biomarkers) was removed from the repository at the owner's request (README note 2019/04/29), so the exact experiment cannot be re-run from the public code without obtaining the spectra separately.
- Because the experimental spectra have been removed, the package cannot be run directly; users must train their own model with an in-house database or wait for public database spectra to be included (README note 2019/05/13).
- Valid identification is defined only when a structure-related metabolite exists in the database; queries whose structural relatives are absent from the database cannot be validly identified by this criterion.
- Discussion-section detail is missing from the source document, so instrument-specific failure modes and score-threshold guidance are not fully documented.

## Evidence

- [readme] Spectra removal note: [evidence span withheld — anchor status: external_doc]
- [readme] Rerun limitation: [evidence span withheld — anchor status: external_doc]
