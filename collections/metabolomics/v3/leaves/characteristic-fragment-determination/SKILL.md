---
name: characteristic-fragment-determination
description: Use when ranking precursor chemical formula candidates from LC-MS/MS
  fragment peaks and you need to flag fragments whose mass-difference relationships
  are unique to each candidate formula set, per the FSA algorithm.
when_to_use_negative:
- The compound contains elements beyond C, H, O, N, S, and P — FSA has so far been
  validated only on datasets with compounds composed of those elements
- 'You need full metabolite identification with structural information — FSA has not
  utilized structural information and its rank #1 accuracy is not yet comparable to
  tools such as Buddy and SIRIUS'
- The compound forms adduct types not covered — few chemical rules and species/adduct-specific
  rules have been implemented to filter out unfeasible formulas
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Standalone module implementing the fragment-formula search, MDR enumeration,
    characteristic-fragment determination, and Eq. (4) ranking
  repo: https://github.com/kslynn128171/FSA
- name: PubChem
  role: Source of the reference chemical formulas and monoisotopic molecular weights
    used to search fragment formulas; also used to derive formulas for unmatched fragment
    peaks via known adducts and neutral losses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:61ca711d08024b7a5e247c4c9220cbaa3e908d4d125ced2e7773c3ea44148268
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
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# characteristic-fragment-determination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines which fragments in an MS/MS spectrum are 'characteristic' — those whose mass-difference relationships (MDRs) are unique to a given candidate precursor formula set — as the core signal the FSA algorithm (Lynn 2025, Metabolomics) uses to rank precursor chemical formulas.

## When to use

Use when ranking precursor chemical formula candidates from LC-MS/MS fragment peaks and you need to flag fragments whose mass-difference relationships are unique to each candidate formula set, per the FSA algorithm.

## When NOT to use

- The compound contains elements beyond C, H, O, N, S, and P — FSA has so far been validated only on datasets with compounds composed of those elements
- You need full metabolite identification with structural information — FSA has not utilized structural information and its rank #1 accuracy is not yet comparable to tools such as Buddy and SIRIUS
- The compound forms adduct types not covered — few chemical rules and species/adduct-specific rules have been implemented to filter out unfeasible formulas

## Inputs

- MS/MS spectrum (fragment m/z and intensity peak list) per metabolite
- Precursor monoisotopic mass
- PubChem-derived reference list of chemical formulas and their monoisotopic molecular weights
- Mass matching tolerance parameter

## Outputs

- Set of characteristic fragments per candidate precursor formula (fragments with MDRs unique among candidate fragment-formula sets)
- Mass-difference relationships (MDRs) per spectrum
- Ranked precursor candidate list with Eq. (4) scores, MDR counts, and mass differences (ranked_candidates.csv with schema spectrum_id, candidate_formula, score, mass_difference)

## How to apply

First, load the MS/MS spectrum and convert each fragment m/z to monoisotopic masses as described in the FSA Methods. Search candidate chemical formulas for the precursor and each fragment within a given mass matching tolerance against a PubChem-derived reference formula list (109,383,767 compounds in the version employed). Enumerate mass-difference relationships (MDRs) among the fragment formulas for each spectrum, then determine characteristic fragments as those whose MDRs are unique among the candidate fragment-formula sets — a fragment that only appears in the fragmentation path of one candidate precursor formula supports that candidate. These characteristic fragments feed the Eq. (4) scoring formula; ties in Eq. (4) scores are broken by the smaller mass difference. Optionally, additional terms can be added to Eq. (4) to encourage long fragmentation paths (consecutive MDRs in a series of fragments), which the authors tested via a genetic algorithm over weighted combinations of multi-formula fragments.

## Related tools

- **FSA** (Standalone module implementing the fragment-formula search, MDR enumeration, characteristic-fragment determination, and Eq. (4) ranking) — https://github.com/kslynn128171/FSA
- **PubChem** (Source of the reference chemical formulas and monoisotopic molecular weights used to search fragment formulas; also used to derive formulas for unmatched fragment peaks via known adducts and neutral losses)

## Evaluation signals

- Reproduces the paper-reported benchmark on the eight test datasets: top-1 accuracy 85.28% and top-5 accuracy 97.35% across 30,690 MS/MS spectra from 6792 metabolites, within reasonable tolerance
- Every spectrum in ranked_candidates.csv has a nonempty ranked candidate list
- Output schema is complete: spectrum_id, candidate_formula, score, mass_difference
- Each ranked candidate's MDR count and mass difference are reported and consistent with the characteristic fragments identified
- Average processing time per spectrum stays near the reported 0.024 s

## Limitations

- Few chemical rules have been implemented to rank possible formulas and filter out unfeasible ones; such rules can be crucial in certain compound species and in compounds with adducts
- FSA has not utilized structural information yet, so its accuracy may not be competitive with state-of-the-art tools such as Buddy and SIRIUS
- The employed PubChem version (109,383,767 compounds) is outdated relative to the current version (112,419,402 compounds as of 2022/11/27), which can affect formula-match coverage
- Some chemical rules may need to be incorporated into FSA to solve the ambiguity mentioned in Sect. 3.7

## Evidence

- [methods] Determine characteristic fragments as fragments whose MDRs are unique among candidate fragment-formula sets.: [evidence span withheld — anchor status: not_found]
- [abstract] FSA ranked the correct chemical formula as the top-1 candidate for a metabolite in 85.28% of the cases and in the top-5 candidates in 97.35% of the cases.: "FSA ranked the correct chemical formula as the top-1 candidate for a metabolite in 85.28% of the cases and in the top-5 candidates in 97.35% of the cases."
- [discussion] Many attempts to include additional terms in Eq. (4), to encourage a long fragmentation path (consecutive MDRs in a series of fragments), were tested: "Many attempts to include additional terms in Eq. (4), to encourage a long fragmentation path (consecutive MDRs in a series of fragments), were tested"
- [readme] Metabolite identification using fragment subset analysis: "Metabolite identification using fragment subset analysis"
