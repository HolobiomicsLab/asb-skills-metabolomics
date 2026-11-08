---
name: formula-subset-enumeration
description: Use when you need to rank precursor molecular formula candidates for
  unlabeled LC-MS/MS metabolite spectra without training data or MS/MS spectral libraries,
  using only fragment m/z values and a PubChem-derived formula list.
when_to_use_negative:
- The target compounds contain elements beyond C, H, O, N, S, and P — the paper states
  FSA has so far been validated only on datasets of compounds composed of these elements,
  and extension to more complex molecules is future work
- You need to distinguish isomers or exploit structural information — FSA does not
  utilize structural information, so its accuracy 'may not be competitive with some
  of the state-of-the-art identification tools'
- Compounds forming various adduct types — species/adduct-specific chemical rules
  are not yet incorporated, so results for such compounds may be unreliable
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: 'The formula subset analysis algorithm itself: enumerates fragment formulas,
    derives MDRs, applies the MaxComb rule, and ranks precursor formulas by the Eq.
    (4) score'
  repo: kslynn128171/FSA
- name: PubChem
  role: Source of the reference chemical formula list and monoisotopic molecular weights
    against which fragment and precursor formulas are mass-matched
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:a253c9fcdd38901479ff5fea3b1381f94fae3580af10ba3b76fb8dd00271a7b6
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# formula-subset-enumeration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements the FSA (Formula Subset Analysis) algorithm from Lynn (Metabolomics 2025), which enumerates candidate precursor chemical formulas from MS/MS fragments by mass-matching against PubChem formulas, deriving maximum-difference-redundancy (MDR) relationships, and ranking precursors with the Eq. (4) score.

## When to use

Use when you need to rank precursor molecular formula candidates for unlabeled LC-MS/MS metabolite spectra without training data or MS/MS spectral libraries, using only fragment m/z values and a PubChem-derived formula list.

## When NOT to use

- The target compounds contain elements beyond C, H, O, N, S, and P — the paper states FSA has so far been validated only on datasets of compounds composed of these elements, and extension to more complex molecules is future work
- You need to distinguish isomers or exploit structural information — FSA does not utilize structural information, so its accuracy 'may not be competitive with some of the state-of-the-art identification tools'
- Compounds forming various adduct types — species/adduct-specific chemical rules are not yet incorporated, so results for such compounds may be unreliable

## Inputs

- MS/MS spectra (precursor plus fragment m/z values) from LC-MS/MS metabolomics experiments
- Mass matching tolerance (ppm/Da threshold for formula search)
- Reference chemical formula list derived from PubChem (the paper used a snapshot of 109,383,767 compounds)
- MaxComb parameter setting (set to 1 for best rank#1 accuracy per Supplementary Fig. S1)

## Outputs

- ranked_candidates.csv with schema (spectrum_id, candidate_formula, score, mass_difference) plus MDR counts
- Per-spectrum ranked precursor formula candidate list with Eq. (4) scores
- MaxComb sweep accuracy table (rank#1 accuracy per MaxComb setting)

## How to apply

Convert each fragment m/z to a monoisotopic mass, then search candidate chemical formulas for the precursor and every fragment within a given mass matching tolerance against the PubChem-derived reference formula list. Enumerate the mass-difference relationships (MDRs) among fragment formulas for each spectrum and determine characteristic fragments whose MDRs are unique among the candidate fragment-formula sets. Apply the MaxComb=1 rule — dropping any fragment formula carrying more than one MDR combination — which the paper's Supplementary Fig. S1 sweep shows maximizes rank#1 accuracy. Score each precursor candidate with the Eq. (4) score (e.g. salbutamol C13H21NO3 scores 1.8333 = 5/6 + 1), breaking ties by smaller mass difference, and write the ranked list per spectrum including scores, MDR counts, and mass differences. Because FSA requires no training data and only publicly available formula lists (PubChem/ChemSpider), it can serve as a fast prescreening step (average 0.024 s per spectrum) complementary to tools like SIRIUS or Buddy.

## Related tools

- **FSA** (The formula subset analysis algorithm itself: enumerates fragment formulas, derives MDRs, applies the MaxComb rule, and ranks precursor formulas by the Eq. (4) score) — kslynn128171/FSA
- **PubChem** (Source of the reference chemical formula list and monoisotopic molecular weights against which fragment and precursor formulas are mass-matched)

## Evaluation signals

- Reproduce the salbutamol (HMDB0001937) worked example: precursor formula C13H21NO3 must rank #1 and achieve the reported Eq. (4) score of 1.8333, matched to 4 decimal places
- Benchmark check: top-1 accuracy of 85.28% and top-5 accuracy of 97.35% across the 30,690 MS/MS spectra / eight test datasets, within reasonable tolerance
- MaxComb sweep: the rank#1 accuracy curve must peak at MaxComb=1 as shown in Supplementary Fig. S1
- Output schema check: every spectrum in ranked_candidates.csv has a nonempty ranked list and complete columns (spectrum_id, candidate_formula, score, mass_difference)
- Tie-breaking check: candidates with equal Eq. (4) scores are ordered by smaller mass difference

## Limitations

- Few chemical rules are implemented to rank possible formulas and filter out unfeasible ones; such rules can be crucial in certain compound species and in compounds with adducts
- The FSA package ships an outdated PubChem snapshot (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so candidate coverage may lag current PubChem
- Accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS because structural information is not yet utilized

## Evidence
