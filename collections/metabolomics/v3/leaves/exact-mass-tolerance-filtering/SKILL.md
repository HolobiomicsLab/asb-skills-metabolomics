---
name: exact-mass-tolerance-filtering
description: Use when filtering metabolomics annotations against a large structure
  database (e.g. PubChem) and a narrow mass window plus CCS-based ΔCCS threshold (2.8%)
  is needed to remove false positives.
when_to_use_negative:
- Training sets only cover common adducts ([M−H]−, [M+H]+, [M+Na]+, [M+K]+); dimers
  like [2M+H]+, uncommon adducts, or in-source fragments are not supported
- Data already consist of a curated annotation table with no need for structure-level
  filtering
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: CCSP 2.0
  role: Jupyter-notebook implementation of the SVR-based CCS predictor that consumes
    the exact-mass-filtered candidate list and predicts CCS values for each remaining
    candidate
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Computes the 1613 molecular descriptors used to represent each candidate's
    structure for CCS prediction
- name: PubChem
  role: Structure database queried for candidate compounds whose exact masses fall
    inside the specified ppm window
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:627a770eda977b015f61e1e5fb20596cfcbfbe1e5c2f212ea963783316e5988f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - Biopython
  - CCSP 2.0
  - Mordred
  - PubChem
  - PubChemPy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Exact-Mass Tolerance Filtering

## Summary

Narrows candidate metabolite annotations by selecting compounds whose exact masses fall within a user-specified mass-error window (e.g. 10 ppm of the target m/z for a given adduct type) before any collision-cross-section prediction is applied.

## When to use

Use when filtering metabolomics annotations against a large structure database (e.g. PubChem) and a narrow mass window plus CCS-based ΔCCS threshold (2.8%) is needed to remove false positives.

## When NOT to use

- Training sets only cover common adducts ([M−H]−, [M+H]+, [M+Na]+, [M+K]+); dimers like [2M+H]+, uncommon adducts, or in-source fragments are not supported
- Data already consist of a curated annotation table with no need for structure-level filtering

## Inputs

- Target m/z value from an IM-MS peak
- Adduct type (e.g. [M−H]−, [M+H]+, [M+Na]+)
- Mass tolerance in ppm (e.g. 10 ppm)
- Structure database of candidate metabolites (e.g. PubChem)

## Outputs

- Filtered list of candidate compounds within the ppm mass window
- Per-candidate exact masses and SMILES for downstream descriptor calculation

## How to apply

Convert the observed m/z into a neutral-molecule search window centered on the target and scaled by the desired tolerance (CCSP 2.0 uses a 10 ppm window when testing class-matched filtering). Query a structure database for candidates whose exact masses fall inside that window, retrieving compound records and canonical/isomeric SMILES for each hit. Compute molecular descriptors for the surviving candidates with Mordred so that downstream linear support-vector-regression models can predict CCS values from the candidate structures. Combine the mass window with the ΔCCS threshold: CCSP 2.0's filtering studies used 2.8% ΔCCS together with 10 ppm mass error, retaining 100% of correct annotations while removing 36.1% of incorrect structures in class-matched data sets. Verify that every retained candidate's exact mass lies inside the specified ppm window of the observed m/z before issuing CCS predictions.

## Related tools

- **CCSP 2.0** (Jupyter-notebook implementation of the SVR-based CCS predictor that consumes the exact-mass-filtered candidate list and predicts CCS values for each remaining candidate) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Computes the 1613 molecular descriptors used to represent each candidate's structure for CCS prediction)
- **PubChem** (Structure database queried for candidate compounds whose exact masses fall inside the specified ppm window)

## Evaluation signals

- Every retained candidate's exact mass falls within the specified ppm window of the observed m/z
- All filtered candidates carry a canonical/isomeric SMILES suitable for Mordred descriptor computation
- Mass window plus ΔCCS threshold (2.8%) retains the correct annotation while removing a measurable fraction (36.1% in the article's class-matched test) of incorrect structures
- Median relative CCS prediction error on the retained set stays below 2%

## Limitations

- The mass window must be paired with a superclass-matched CCS model; class mismatch weakens filtering confidence
- Performance is evaluated only on [M−H]−, [M+H]+, and [M+Na]+ adducts from the McLean CCS Compendium; other adduct types may behave differently
- The 36.1% incorrect-structure removal was observed for class-matched data sets and may not generalize to broader or heterogeneous sample sets

## Evidence
