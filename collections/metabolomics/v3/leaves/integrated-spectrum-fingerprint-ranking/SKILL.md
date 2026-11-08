---
name: integrated-spectrum-fingerprint-ranking
description: Use when an analyte's GC/EI-MS spectrum matches no known reference spectra
  and a ranked list of putative NPS identities must be produced from an enumerated
  synthetic database.
when_to_use_negative:
- When the analyte spectrum already matches a known reference spectrum in NIST, Wiley,
  or SWGDRUG libraries — conventional library screening suffices.
- When the chemical structure of the candidate is already known and only its spectrum
  is needed — use NEIMS alone rather than the integrated ranking.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: PS2MS
  role: End-to-end system that enumerates derivatives, orchestrates NEIMS/DeepEI,
    and computes the integrated SMSF similarity ranking.
  repo: https://github.com/jhhung/PS2MS
- name: NEIMS
  role: Predicts the EI mass spectrum of each enumerated derivative in the synthetic
    NPS database, producing the candidate spectra side of the similarity comparison.
  repo: https://github.com/jhhung/PS2MS
- name: DeepEI
  role: Converts the unknown analyte's mass spectrum into its molecular fingerprint
    (without structure knowledge) and fingerprints the database compounds; fingerprints
    are merged into the msp file.
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Python chemistry toolkit required to build/run the enumeration and fingerprint
    steps.
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:5c84b2ed821d71e87fc2a4d58f1532739e432f2ca266a387068914e3660059ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - RASSP
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# integrated-spectrum-fingerprint-ranking

## Summary

Ranks candidate NPS identities by combining GC/EI-MS spectrum similarity with molecular-fingerprint similarity (SMSF), using NEIMS-predicted spectra of enumerated derivatives and DeepEI-predicted fingerprints of the unknown analyte.

## When to use

Use when an analyte's GC/EI-MS spectrum matches no known reference spectra and a ranked list of putative NPS identities must be produced from an enumerated synthetic database.

## When NOT to use

- When the analyte spectrum already matches a known reference spectrum in NIST, Wiley, or SWGDRUG libraries — conventional library screening suffices.
- When the chemical structure of the candidate is already known and only its spectrum is needed — use NEIMS alone rather than the integrated ranking.

## Inputs

- Query GC/EI-MS spectrum of an unknown analyte (70 eV EI, from NIST/Wiley/SWGDRUG-style acquisition)
- Enumerated derivative candidate subset (SMILES) from the synthetic NPS database
- NEIMS-predicted EI mass spectra of the enumerated derivatives (MSP format)
- DeepEI-predicted molecular fingerprints of the query analyte

## Outputs

- Ranked list of candidate NPS identities per query, with SMSF similarity scores (top 100 reported)
- Per-substance rank table and aggregate top-1 / top-20 recovery proportions on SWGDRUG/NIST test sets
- MSP file with fingerprints merged into spectrum records

## How to apply

After rule-based enumeration of derivatives (with SCScore and drug-like filtering yielding ~422 million compounds) and NEIMS prediction of their EI spectra, compute cosine similarity plus Jaccard similarity over top-intensity peaks between the query spectrum and each candidate, allowing a ±1 Da tolerance. Use DeepEI to convert the query analyte's mass spectrum into its molecular fingerprint without knowing the analyte structure. Combine the two spectrum similarity terms with fingerprint similarity as the weighted-sum SMSF score for every analyte–derivative pair, then rank all database candidates per query. Validation is against SWGDRUG and NIST test sets, where >50% top-1 and >85%/90% top-20 recovery was reported; PS2MS merges fingerprints into the MSP file so ranking runs directly on the enriched msp.

## Related tools

- **PS2MS** (End-to-end system that enumerates derivatives, orchestrates NEIMS/DeepEI, and computes the integrated SMSF similarity ranking.) — https://github.com/jhhung/PS2MS
- **NEIMS** (Predicts the EI mass spectrum of each enumerated derivative in the synthetic NPS database, producing the candidate spectra side of the similarity comparison.) — https://github.com/jhhung/PS2MS
- **DeepEI** (Converts the unknown analyte's mass spectrum into its molecular fingerprint (without structure knowledge) and fingerprints the database compounds; fingerprints are merged into the msp file.) — https://github.com/jhhung/PS2MS
- **rdkit** (Python chemistry toolkit required to build/run the enumeration and fingerprint steps.) — https://www.rdkit.org/docs/Install.html

## Examples

```
python drug-detection/run.py --msp synthetic_database.msp --query analyte.msp --fingerprint merged.msp
```

## Evaluation signals

- On the SWGDRUG test set, over 85% of test substances should rank within the top 20 predictions (top-20 recovery), mirroring the reported benchmark.
- On the NIST test set, over 90% of test substances should rank within the top 20 predictions.
- After incorporating Jaccard similarity, the top-1 correct-prediction proportion should rise from below 40% to more than 50%.
- The output should be a ranked list of the hundred most similar compounds per query, ordered by the integrated SMSF similarity score.
- Each query–candidate pair must produce both a spectrum similarity (cosine + Jaccard with ±1 Da tolerance) and a fingerprint similarity; a missing term indicates a broken DeepEI or NEIMS stage.

## Limitations

- Relies on NEIMS prediction quality; the original NEIMS was trained solely on NIST library spectra, which may limit generalization to SWGDRUG-type analytes.
- Similarity scores can be ambiguous — distinguishing cathinone-like from non-cathinone-like drugs relied on Gaussian distributions of similarity scores, and additional postprocessing is anticipated to determine precise identities.
- The enumerated database (~422 million compounds) is large; enumeration level and screening choices (SCScore thresholds, drug-like criteria) directly affect runtime and candidate coverage.
- Isotopic/peak variance requires a ±1 Da mass tolerance in the similarity metric, which can introduce near-mass false matches.

## Evidence

- [intro] The integrated similarity metric between the analyte and enumerated compounds in the NPS database based on the mass spectrum and fingerprint are calculated, summarized, and ranked.: "The integrated similarity metric between the analyte and enumerated compounds in the NPS database based on the mass spectrum and fingerprint are calculated, summarized, and ranked."
- [methods] over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [methods] The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity.: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [readme] The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score.: "The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score."
- [results] By forming Gaussian distributions of similarity scores for cathinone-like and noncathinone-like drugs from SWGDRUG data sets: "By forming Gaussian distributions of similarity scores for cathinone-like and noncathinone-like drugs from SWGDRUG data sets"
- [methods] the process of enumeration and filtering resulted in a total of approximately 422 million compounds.: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
