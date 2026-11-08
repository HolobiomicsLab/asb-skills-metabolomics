---
name: per-compound-majority-vote-aggregation
description: Use when a compound-level decision must be derived from multiple per-spectrum
  classifier outputs (e.g. EI-MS and ESI-MS/MS spectra of the same compound) and a
  single bitter/nonbitter label is required for precision/recall evaluation.
when_to_use_negative:
- When only a single spectrum is available per compound — there is no vote to aggregate
  and per-spectrum predictions showed poor results on their own.
- When the goal is to annotate a chemical structure from a spectrum rather than to
  obtain a compound-level bitterness label; the article notes the spectrum→structure→bitterness
  route is less effective and covers fewer compounds.
- When predictions come from a structure-based classifier such as BitterPredict, which
  already outputs one label per compound.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: BitterMasS
  role: Random Forest classifier that produces the per-spectrum bitter/nonbitter predictions
    that are aggregated by majority vote
- name: MassBank
  role: Source of the experimental EI-MS and ESI-MS/MS positive ion mode spectra that
    feed the per-spectrum predictions
- name: BitterPredict
  role: Structure-based alternative compared against the spectrum-based route; not
    used in the vote aggregation itself
  repo: Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:3abde114a209c4051543d83fe7d13f125fcf083302e61c43652f866e072b1528
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-compound-majority-vote-aggregation

## Summary

Aggregates per-spectrum bitterness predictions from the BitterMasS Random Forest classifier into a single per-compound label by majority voting, as used for BitterMasS's external test set 1 evaluation on 759 MassBank/DrugBank spectra covering 106 literature-extracted compounds.

## When to use

Use when a compound-level decision must be derived from multiple per-spectrum classifier outputs (e.g. EI-MS and ESI-MS/MS spectra of the same compound) and a single bitter/nonbitter label is required for precision/recall evaluation.

## When NOT to use

- When only a single spectrum is available per compound — there is no vote to aggregate and per-spectrum predictions showed poor results on their own.
- When the goal is to annotate a chemical structure from a spectrum rather than to obtain a compound-level bitterness label; the article notes the spectrum→structure→bitterness route is less effective and covers fewer compounds.
- When predictions come from a structure-based classifier such as BitterPredict, which already outputs one label per compound.

## Inputs

- Per-spectrum BitterMasS predictions (bitter/nonbitter) for EI-MS and ESI-MS/MS positive ion mode spectra
- Mapping of each spectrum to its parent compound identifier
- Reference sensory (taste-panel) bitter/nonbitter labels for each compound for metric computation

## Outputs

- One aggregated bitter/nonbitter label per compound
- Per-compound precision and recall against sensory labels (reported: precision = 0.67, recall = 0.93)

## How to apply

First obtain a per-spectrum bitterness prediction from BitterMasS for every retrieved spectrum of a compound (the Random Forest was trained on 5414 experimental mass spectra and operates on EI-MS or ESI-MS/MS spectra in positive ion mode). For each compound, gather all per-spectrum predictions — in the external test set 1 this covered 759 spectra for 106 compounds retrieved from MassBank, supplemented with DrugBank spectra for medication and vitamin categories. Assign the compound's final label by majority vote across its per-spectrum predictions. Then compute per-compound precision and recall against the sensory (taste-panel) labels; the reported external test set 1 result was precision = 0.67 and recall = 0.93. The rationale is that prediction on individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set, so combining evidence across spectra of the same compound is necessary for reliable classification.

## Related tools

- **BitterMasS** (Random Forest classifier that produces the per-spectrum bitter/nonbitter predictions that are aggregated by majority vote)
- **MassBank** (Source of the experimental EI-MS and ESI-MS/MS positive ion mode spectra that feed the per-spectrum predictions)
- **BitterPredict** (Structure-based alternative compared against the spectrum-based route; not used in the vote aggregation itself) — Niv-Lab/BitterPredict1

## Evaluation signals

- Every compound in the test set receives exactly one final label, and every spectrum in the set contributes to its parent compound's vote.
- Per-compound precision and recall computed against sensory labels fall near the reported external test set 1 values (precision = 0.67, recall = 0.93).
- Aggregated per-compound performance must exceed per-spectrum performance; per-spectrum-only results were reported as poor relative to the combined set.
- Spectrum-to-compound mapping must be consistent: compounds with spectra from both EI-MS and ESI-MS/MS in positive ion mode must have all their spectra included in the vote.

## Limitations

- BitterMasS only accepts EI-MS or ESI-MS/MS spectra obtained in positive ion mode; negative-mode spectra cannot contribute votes.
- Per-spectrum predictions alone performed poorly compared with the combined (majority-vote) set, so single-spectrum compounds may be unreliable.
- Nonbitter compound spectral representation in the training set was insufficient, which may bias vote outcomes for nonbitter compounds.
- Only ~30% of the initial bitter and nonbitter training compounds had at least one mass spectrum, limiting training coverage and hence vote quality.

## Evidence

- [results] On external test set 1, per-spectrum predictions were combined by majority vote to assign a single bitterness label per compound.: [evidence span withheld — anchor status: not_found]
- [discussion] Combined-set prediction is needed because individual spectrum sets performed poorly.: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] BitterMasS operates on positive ion mode spectra, constraining which spectra can vote.: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
