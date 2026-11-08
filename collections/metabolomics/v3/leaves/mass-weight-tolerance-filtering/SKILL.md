---
name: mass-weight-tolerance-filtering
description: Use when a large enumerated derivative database (hundreds of millions
  of candidates) must be pruned to plausible candidates before similarity scoring,
  and a reference M+ mass from a GC/EI-MS analyte spectrum is available.
when_to_use_negative:
- The analyte's spectrum already matches an established reference spectrum (NIST/Wiley/SWGDRUG)
  — the conventional screening path should be used instead of enumerating and mass-filtering
  derivatives
- No reliable M+ peak can be identified in the analyte spectrum, since the entire
  filter is anchored to that reference mass
- You need isomer-level discrimination — mass tolerance filtering only removes grossly
  different molecular masses and cannot separate isobaric candidates; use integrated
  spectrum/fingerprint similarity for that
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: PS2MS
  role: Implements the enumeration and rule-based filtering pipeline, including M+-referenced
    mass-weight filtering of enumerated compounds
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime for scanning the enumerated database and writing the tolerance-window
    candidate CSV subsets
  repo: https://www.python.org/downloads/
- name: rdkit
  role: Used in the enumeration stage to generate the derivative structures and their
    exact molecular masses prior to mass filtering
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:529c3468205357846f11a56b58ab72dec865a337f6a463e2da4ec791a8c343c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - python
  - python 3.6.9
  - rdkit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# mass-weight-tolerance-filtering

## Summary

Rule-based filtering step from PS2MS that narrows an enumerated synthetic NPS database by retaining only derivatives whose molecular mass falls within a tolerance window (e.g., ±1 Da or ±5 Da) of the analyte's M+ peak reference mass.

## When to use

Use when a large enumerated derivative database (hundreds of millions of candidates) must be pruned to plausible candidates before similarity scoring, and a reference M+ mass from a GC/EI-MS analyte spectrum is available.

## When NOT to use

- The analyte's spectrum already matches an established reference spectrum (NIST/Wiley/SWGDRUG) — the conventional screening path should be used instead of enumerating and mass-filtering derivatives
- No reliable M+ peak can be identified in the analyte spectrum, since the entire filter is anchored to that reference mass
- You need isomer-level discrimination — mass tolerance filtering only removes grossly different molecular masses and cannot separate isobaric candidates; use integrated spectrum/fingerprint similarity for that

## Inputs

- M+ peak mass of the unknown analyte taken from its SWGDRUG reference GC/EI-MS spectrum
- Enumerated synthetic NPS derivative database with precomputed exact masses per compound (from PS2MS enumeration with functional-group substitution and drug-like criteria)

## Outputs

- CSV candidate list restricted to compounds within ±1 Da of the reference M+ mass
- CSV candidate list restricted to compounds within ±5 Da of the reference M+ mass
- Summary of candidate counts per tolerance setting (mwf ±1 and mwf ±5)

## How to apply

Extract the molecular-weight reference from the M+ peak of the analyte's SWGDRUG reference spectrum, since PS2MS 'utilizes the M+ peak as a reference for the mass weight filtering of enumerated compounds within a certain tolerance range'. Load the enumerated derivative database (built by substituting hydrogens in the drug core with functional groups, precomputed with exact masses) and scan it twice: once with a tight ±1 Da tolerance and once with a broader ±5 Da tolerance. Write each retained subset to a CSV candidate list and record the candidate counts per tolerance setting in a summary. This mass filter precedes spectrum/fingerprint similarity ranking (SMSF plus Jaccard similarity) so that downstream deep-learning predictions (NEIMS spectra, DeepEI fingerprints) are only computed for mass-compatible candidates. The two tolerance settings trade candidate recall against database size — ±1 Da is highly restrictive, ±5 Da admits near-isobaric derivatives.

## Related tools

- **PS2MS** (Implements the enumeration and rule-based filtering pipeline, including M+-referenced mass-weight filtering of enumerated compounds) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime for scanning the enumerated database and writing the tolerance-window candidate CSV subsets) — https://www.python.org/downloads/
- **rdkit** (Used in the enumeration stage to generate the derivative structures and their exact molecular masses prior to mass filtering) — https://www.rdkit.org/docs/Install.html

## Evaluation signals

- Every candidate in the ±1 Da subset must have an exact mass within 1 Da of the reference M+ mass; the same invariant must hold for the ±5 Da subset
- Reported candidate counts must equal the row counts of the corresponding CSV files
- The ±1 Da subset must be a strict subset (or equal) of the ±5 Da subset
- All retained candidates' masses must lie within [M+ − tolerance, M+ + tolerance] bounds — no negatives or out-of-window rows

## Limitations

- The filter is anchored to the analyte's M+ peak; a misassigned molecular ion propagates directly into the candidate set
- Mass tolerance filtering cannot distinguish isobaric derivatives, so the candidate set remains large and must be resolved by downstream SMSF/Jaccard similarity ranking
- Exact tolerance ranges beyond the described ±1/±5 settings are not parameterized in the article, and no accession IDs for the synthetic NPS database are provided

## Evidence

- [discussion] M+ peak as reference with tolerance range: "utilizes the
M+ peak as a reference for the mass
weight filtering of enumerated compounds within a certain tolerance
range"
- [methods] filter steps in methods: "## Methods > Enumeration And Rule-Based Filtering Of Derivatives"
- [methods] enumeration scale: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [readme] readme on enumeration: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [readme] downstream similarity ranking: "The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score."
