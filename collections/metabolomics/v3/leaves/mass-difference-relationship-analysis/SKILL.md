---
name: mass-difference-relationship-analysis
description: Use when ranking precursor chemical formula candidates from LC-MS/MS
  spectra without training data or MS/MS database matching, i.e., when spectra must
  be scored via mass-difference relationships among fragment formulas.
when_to_use_negative:
- The molecules of interest are outside the validated element set (C, H, O, N, S,
  P) or form adduct types not yet handled — FSA 'has so far been validated on datasets
  containing compounds composed of C, H, O, N, S, and P'
- 'Maximum identification accuracy is required over speed — the paper notes rank #1
  accuracy ''was not yet comparable to that of state-of-the-art tools such as Buddy
  and SIRIUS'' because FSA does not use structural information'
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: FSA
  role: Reference implementation of the MDR-based formula search, characteristic-fragment
    determination, and Eq. (4) ranking pipeline
  repo: kslynn128171/FSA
- name: PubChem
  role: Source of the reference list of chemical formulas and corresponding monoisotopic
    molecular weights used for fragment formula search
- name: ProteoWizard
  role: msconvert used for format conversion of raw MS/MS data into processable spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:681124dce7d59ac217ca92941ec70beb25db9e0f3c455211daeb508392dcee08
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
  - ProteoWizard
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-difference-relationship-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the FSA (formula subset analysis) ranking pipeline: convert MS/MS fragment m/z to monoisotopic masses, search candidate formulas within a mass tolerance against a PubChem-derived reference list, enumerate mass-difference relationships (MDRs), and rank precursor formulas with the Eq. (4) score.

## When to use

Use when ranking precursor chemical formula candidates from LC-MS/MS spectra without training data or MS/MS database matching, i.e., when spectra must be scored via mass-difference relationships among fragment formulas.

## When NOT to use

- The molecules of interest are outside the validated element set (C, H, O, N, S, P) or form adduct types not yet handled — FSA 'has so far been validated on datasets containing compounds composed of C, H, O, N, S, and P'
- Maximum identification accuracy is required over speed — the paper notes rank #1 accuracy 'was not yet comparable to that of state-of-the-art tools such as Buddy and SIRIUS' because FSA does not use structural information

## Inputs

- MS/MS spectra (LC-MS/MS, e.g., converted via ProteoWizard msconvert) with precursor and fragment peak lists
- Mass matching tolerance for fragment-to-formula assignment
- Reference chemical formula list with monoisotopic molecular weights derived from PubChem (109,383,767-compound snapshot in the paper)

## Outputs

- ranked_candidates.csv with schema (spectrum_id, candidate_formula, score, mass_difference)
- Per-spectrum ranked precursor formula list including Eq. (4) scores, MDR counts, and mass differences

## How to apply

Preprocess each MS/MS spectrum by converting every fragment m/z to its monoisotopic mass, then search possible chemical formulas for the precursor and each fragment under a given mass matching tolerance against a PubChem-derived reference formula list. For each spectrum, enumerate the mass-difference relationships (MDRs) among fragment formulas and flag characteristic fragments as those whose MDRs are unique among the candidate fragment-formula sets. Score each precursor candidate formula using the Eq. (4) score from the FSA Methods, breaking ties by the smaller mass difference. Write the ranked candidate list per spectrum — including scores, MDR counts, and mass differences — to ranked_candidates.csv. The reference implementation validated on eight test datasets (30,690 spectra from 6792 metabolites) used a PubChem snapshot of 109,383,767 compounds and compounds composed of C, H, O, N, S, and P.

## Related tools

- **FSA** (Reference implementation of the MDR-based formula search, characteristic-fragment determination, and Eq. (4) ranking pipeline) — kslynn128171/FSA
- **PubChem** (Source of the reference list of chemical formulas and corresponding monoisotopic molecular weights used for fragment formula search)
- **ProteoWizard** (msconvert used for format conversion of raw MS/MS data into processable spectra)

## Evaluation signals

- Standalone module reproduces the paper benchmark: correct formula ranked top-1 in 85.28% and top-5 in 97.35% of cases across 30,690 MS/MS spectra from 6792 metabolites, within reasonable tolerance on the same inputs
- Every spectrum in ranked_candidates.csv has a nonempty ranked candidate list
- Output schema is complete: spectrum_id, candidate_formula, score, mass_difference columns all populated
- Tie handling matches the paper: candidate formulas with equal Eq. (4) scores are ordered by smaller mass difference
- Throughput sanity check: average processing time per spectrum is on the order of the reported 0.024 s

## Limitations

- Few chemical rules are implemented to rank or filter out unfeasible formulas; species/adduct-specific rules are not yet incorporated and may be crucial for certain compound species and adducts
- The embedded PubChem snapshot is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27)
- FSA does not yet utilize structural information, so accuracy may not be competitive with state-of-the-art identification tools such as Buddy and SIRIUS
- Only validated on compounds composed of C, H, O, N, S, and P; applicability to more complex molecules and various adduct types is future work

## Evidence

- [methods] FSA preprocesses the spectrum by converting m/z to monoisotopic masses, then searches possible chemical formulas for the precursor and each fragment under a given mass matching tolerance.: [evidence span withheld — anchor status: not_found]
