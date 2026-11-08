---
name: maximum-difference-redundancy-fragment-filtering
description: Use when enumerating candidate molecular formulas from LC-MS/MS fragment
  peaks with FSA and fragment formulas resolve to multiple maximum-difference-redundancy
  (MDR) combinations, and you must set MaxComb to control which fragments are dropped
  before scoring precursor candidates with Eq. 4.
when_to_use_negative:
- Do not rely on this as the sole ranking step when structural information matters
  — the paper notes FSA 'has not utilized structural information yet and therefore
  its accuracy may not be competitive with some of the state-of-the-art identification
  tools' such as Buddy and SIRIUS.
- Do not apply to compounds containing elements beyond C, H, O, N, S, P or to compounds
  forming various adduct types, since FSA has not been validated in those regimes.
- Do not use when chemical species/adduct-specific feasibility filtering is required
  — the paper reports that few chemical rules have been implemented to rank formulas
  and filter out unfeasible ones.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Implements the fragment subset analysis pipeline in which MaxComb filters
    fragments with multiple MDR combinations before precursor formula scoring; benchmarked
    with rank#1 accuracy of 85.28% and 0.024 s average processing time per spectrum.
  repo: kslynn128171/FSA
- name: SIRIUS
  role: State-of-the-art comparator whose rank#1 accuracy FSA (with the MaxComb=1
    rule) is measured against; also a slower alternative (5 h 41 min for 5518 spectra).
- name: Buddy
  role: State-of-the-art ranking benchmark referenced when judging whether FSA's MaxComb-filtered
    formula ranking is competitive.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:d2d3cd183965b3d7aafba24e34da0da6143408fe302e135b85ad5172ea289017
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
  - Buddy
  - FSA
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# maximum-difference-redundancy-fragment-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the MaxComb parameter of FSA (Formula Subset Analysis) to limit how many maximum-difference-redundancy (MDR) combinations a fragment formula may carry; setting MaxComb=1 drops any fragment with multiple MDR combinations and, per the paper's Supplementary Fig S1 sweep, yields the highest rank#1 formula accuracy.

## When to use

Use when enumerating candidate molecular formulas from LC-MS/MS fragment peaks with FSA and fragment formulas resolve to multiple maximum-difference-redundancy (MDR) combinations, and you must set MaxComb to control which fragments are dropped before scoring precursor candidates with Eq. 4.

## When NOT to use

- Do not rely on this as the sole ranking step when structural information matters — the paper notes FSA 'has not utilized structural information yet and therefore its accuracy may not be competitive with some of the state-of-the-art identification tools' such as Buddy and SIRIUS.
- Do not apply to compounds containing elements beyond C, H, O, N, S, P or to compounds forming various adduct types, since FSA has not been validated in those regimes.
- Do not use when chemical species/adduct-specific feasibility filtering is required — the paper reports that few chemical rules have been implemented to rank formulas and filter out unfeasible ones.

## Inputs

- MS/MS spectrum with a precursor peak and daughter fragment m/z values (e.g. the salbutamol HMDB0001937 worked example)
- Candidate chemical formulas with monoisotopic molecular weights from PubChem/ChemSpider (C, H, O, N, S, P element space)
- MDR combination assignments per fragment formula
- MaxComb setting (sweep range; MaxComb=1 is the accuracy-optimal setting)

## Outputs

- Filtered fragment-formula subset (fragments with >1 MDR combination removed at MaxComb=1)
- Ranked list of precursor formula candidates scored via Eq. 4 (e.g. C13H21NO3 scoring 1.8333 for salbutamol)
- MaxComb sweep table of rank#1 accuracy per setting (cf. Supplementary Fig S1)

## How to apply

During formula subset analysis, enumerate candidate formulas for the precursor and daughter fragments (element range C, H, O, N, S, P), then for each fragment formula determine its MDR combinations. Apply the MaxComb rule as a filter step: with MaxComb=1, any fragment formula that carries more than one MDR combination is dropped from the subset before CF determination, removing ambiguous daughter fragments. Score surviving precursor formula combinations with Eq. 4 (the paper's combination score, e.g. 5/6 + 1 = 1.8333 for salbutamol) and rank candidates. Rationale: the MaxComb sweep (Supplementary Fig S1) shows rank#1 accuracy peaks when MaxComb is reduced to 1, because fragments admitting several MDR combinations introduce ambiguity into the CF-selection logic; the salbutamol worked example (HMDB0001937) confirms the rule — the combination with precursor C13H21NO3 (paired with C13H17NO2, C12H17NO, and C9H10NO2) obtained the highest score of 1.8333 and C13H21NO3 was indeed the correct formula.

## Related tools

- **FSA** (Implements the fragment subset analysis pipeline in which MaxComb filters fragments with multiple MDR combinations before precursor formula scoring; benchmarked with rank#1 accuracy of 85.28% and 0.024 s average processing time per spectrum.) — kslynn128171/FSA
- **SIRIUS** (State-of-the-art comparator whose rank#1 accuracy FSA (with the MaxComb=1 rule) is measured against; also a slower alternative (5 h 41 min for 5518 spectra).)
- **Buddy** (State-of-the-art ranking benchmark referenced when judging whether FSA's MaxComb-filtered formula ranking is competitive.)

## Evaluation signals

- MaxComb sweep: the rank#1 accuracy curve must peak at MaxComb=1, replicating Supplementary Fig S1.
- Salbutamol validation: precursor formula C13H21NO3 (or its protonated form) must rank #1 after CF determination.
- Score reproduction: C13H21NO3 must obtain the reported score of 1.8333 (5/6 + 1 by Eq. 4), matching to 4 decimal places.
- Sanity check on the filter: at MaxComb=1, no retained fragment formula carries more than one MDR combination.
- Aggregate regression: on the 30,690-spectrum / 6792-metabolite benchmark, correct formula in top-1 for ~85.28% and top-5 for ~97.35% of cases, at ~0.024 s average processing time per spectrum.

## Limitations

- Few chemical rules are implemented to rank possible formulas and filter out unfeasible ones; such rules are crucial for certain compound species and compounds with adducts.
- FSA has only been validated on compounds composed of C, H, O, N, S, and P; extension to more complex molecules and adduct-forming compounds is future work.
- The PubChem snapshot used is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so fragment peaks without a PubChem match may lack derivable formulas.
- Attempts to encourage longer fragmentation paths (consecutive MDRs) by adding terms to Eq. 4 and combining scoring via a genetic algorithm were tested but the accuracy 'was not yet comparable to that of state-of-the-art tools such as Buddy and SIRIUS'.

## Evidence

- [results] the combination with precursor formula C13H21NO3 (with C13H17NO2, C12H17NO, and C9H10NO2) exhibited the most MDRs and obtained the highest score of 1.8333 (5/6 + 1 by Eq. 4): [evidence span withheld — anchor status: not_found]
