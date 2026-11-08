---
name: msms-precursor-formula-ranking
description: Use when a precursor ion's molecular formula must be ranked from an LC-MS/MS
  spectrum using fragment peak masses and a PubChem-derived formula list, without
  MS/MS databases or trained models.
when_to_use_negative:
- Input is MS1-only data with no fragment peaks, since MDR enumeration requires fragment
  mass differences
- The goal is structural identification, since FSA 'has not utilized structural information
  yet' and its accuracy is not comparable to state-of-the-art tools like Buddy and
  SIRIUS
- Compounds contain elements beyond C, H, O, N, S, P or form adduct types beyond those
  validated, per the paper's stated scope
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: FSA
  role: Implements the full MDR-enumeration, subset-filtering, and Eq. (4) scoring
    pipeline that produces the ranked precursor formula list
  repo: https://github.com/kslynn128171/FSA
- name: PubChem
  role: Source of the chemical-formula and monoisotopic-molecular-weight reference
    list against which fragment and precursor candidates are searched
- name: SIRIUS
  role: State-of-the-art baseline used for benchmarking rank#1 accuracy of FSA-computed
    formulas
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:ae9f23d24386871f1d0873c6fc770f03c9aa19af3c26d597514dcf3a434a9022
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - PubChem
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# msms-precursor-formula-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank candidate precursor molecular formulas for LC-MS/MS metabolites using the Formula Subset Analysis (FSA) algorithm, which scores formulas from mass-difference relationships (MDRs) among fragment formulas — no training data or MS/MS spectral databases required.

## When to use

Use when a precursor ion's molecular formula must be ranked from an LC-MS/MS spectrum using fragment peak masses and a PubChem-derived formula list, without MS/MS databases or trained models.

## When NOT to use

- Input is MS1-only data with no fragment peaks, since MDR enumeration requires fragment mass differences
- The goal is structural identification, since FSA 'has not utilized structural information yet' and its accuracy is not comparable to state-of-the-art tools like Buddy and SIRIUS
- Compounds contain elements beyond C, H, O, N, S, P or form adduct types beyond those validated, per the paper's stated scope

## Inputs

- LC-MS/MS spectrum: precursor m/z plus fragment peak m/z list (m/z converted to monoisotopic masses)
- PubChem-derived reference list of chemical formulas with corresponding monoisotopic molecular weights
- Mass matching tolerance setting for fragment/precursor formula search

## Outputs

- ranked_candidates.csv with schema (spectrum_id, candidate_formula, score, mass_difference) per spectrum
- Per-candidate Eq. (4) scores, MDR counts, and mass differences

## How to apply

Convert each fragment m/z to monoisotopic mass, then search candidate chemical formulas for the precursor and every fragment within a defined mass-matching tolerance against a PubChem-derived reference formula list. Enumerate mass-difference relationships (MDRs) among the fragment formulas, and apply the MaxComb=1 rule — dropping any fragment formula carrying more than one MDR combination — so subsets match the paper's CF determination, which was validated on salbutamol (HMDB0001937) where C13H21NO3 achieved the top score of 1.8333 (5/6 + 1 by Eq. 4). Score each precursor candidate with the Eq. (4) score, breaking ties by smaller mass difference, and write the ranked list with scores, MDR counts, and mass differences to ranked_candidates.csv.

## Related tools

- **FSA** (Implements the full MDR-enumeration, subset-filtering, and Eq. (4) scoring pipeline that produces the ranked precursor formula list) — https://github.com/kslynn128171/FSA
- **PubChem** (Source of the chemical-formula and monoisotopic-molecular-weight reference list against which fragment and precursor candidates are searched)
- **SIRIUS** (State-of-the-art baseline used for benchmarking rank#1 accuracy of FSA-computed formulas)

## Examples

```
git clone https://github.com/kslynn128171/FSA && python fsa --spectrum salbutamol.mzML --out ranked_candidates.csv
```

## Evaluation signals

- Rank#1 accuracy should reach ~85.28% and rank#5 ~97.35% across the 30,690-spectrum benchmark (6792 metabolites) from the eight test datasets
- Salbutamol worked example: precursor C13H21NO3 must rank #1 and reproduce the reported Eq. (4) score of 1.8333 (exact to 4 decimal places)
- MaxComb sweep of rank#1 accuracy must peak at MaxComb=1, matching Supplementary Fig S1
- Every spectrum in ranked_candidates.csv must have a nonempty ranked list and the schema (spectrum_id, candidate_formula, score, mass_difference) must be complete
- Average processing time per spectrum should be near the reported 0.024 s

## Limitations

- Few chemical rules are implemented to rank formulas or filter out unfeasible ones; species/adduct-specific rules are not yet incorporated
- The employed PubChem snapshot (109,383,767 compounds) is outdated versus the 2022/11/27 version (112,419,402 compounds)
- Rank#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS; chemical-rule incorporation is still needed to resolve ambiguity (Sect. 3.7)

## Evidence
