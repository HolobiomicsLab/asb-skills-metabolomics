---
name: formula-ranking-score-validation
description: Use when replicating or validating FSA formula-ranking behavior — e.g.
  sweeping the MaxComb setting for rank#1 accuracy, or verifying a worked metabolite
  example such as salbutamol (HMDB0001937) precursor C13H21NO3 scoring 1.8333.
when_to_use_negative:
- Molecules composed of elements outside C, H, O, N, S, and P — FSA has only been
  validated on those elements
- Compounds forming adduct types beyond those validated — species/adduct-specific
  chemical rules are not yet implemented in FSA
- Analyses requiring structural information or state-of-the-art rank#1 accuracy, where
  tools like Buddy or SIRIUS outperform FSA
tools:
- name: FSA
  role: Enumerates candidate fragment and precursor formulas, generates MDR-based
    formula subsets under the MaxComb=1 rule, and scores precursor formulas via Eq.
    4 for the rank#1 validation
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:8de0ac36fd108f7c52dd7b84742ff85dcc167810e9eef3f8eee73674770f89aa
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# formula-ranking-score-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the FSA (formula subset analysis) precursor-formula ranking by reproducing the MaxComb=1 rule (dropping fragments carrying multiple maximum-difference-redundancy, MDR, combinations) and checking the salbutamol worked example, where fragment formula C13H21NO3 must score 1.8333 (5/6 + 1 per Eq. 4) and rank #1.

## When to use

Use when replicating or validating FSA formula-ranking behavior — e.g. sweeping the MaxComb setting for rank#1 accuracy, or verifying a worked metabolite example such as salbutamol (HMDB0001937) precursor C13H21NO3 scoring 1.8333.

## When NOT to use

- Molecules composed of elements outside C, H, O, N, S, and P — FSA has only been validated on those elements
- Compounds forming adduct types beyond those validated — species/adduct-specific chemical rules are not yet implemented in FSA
- Analyses requiring structural information or state-of-the-art rank#1 accuracy, where tools like Buddy or SIRIUS outperform FSA

## Inputs

- MS/MS spectrum of a target metabolite with precursor mass and daughter-fragment masses (e.g. salbutamol, HMDB0001937)
- Candidate chemical formulas with monoisotopic molecular weights from PubChem/ChemSpider (FSA's employed PubChem snapshot: 109,383,767 compounds)
- MaxComb sweep parameter settings (including MaxComb=1)
- MDR (maximum-difference-redundancy) combination counts per fragment formula

## Outputs

- Ranked precursor formula list with rank#1 and top-5 positions (target rank#1 accuracy 85.28%, top-5 97.35% across the benchmark)
- MaxComb sweep table of rank#1 accuracy per setting, peaking at MaxComb=1
- Salbutamol validation score for C13H21NO3 (expected 1.8333 = 5/6 + 1 by Eq. 4, exact to 4 decimal places)
- Results table combining sweep accuracies and worked-example validation outputs

## How to apply

Load the MS/MS spectrum (precursor plus daughter fragments) and candidate formula sets from chemical databases (PubChem, ChemSpider), then replicate the Supplementary Fig S1 experiment by sweeping the MaxComb setting across its tested range and computing rank#1 accuracy at each setting, confirming the curve peaks at MaxComb=1. Apply the MaxComb=1 rule during subset generation: any fragment formula carrying more than one MDR combination must be dropped, matching the CF determination step that removed ambiguous daughter fragments. Score precursor formulas with Eq. 4 so that the salbutamol combination C13H21NO3 (with C13H17NO2, C12H17NO, and C9H10NO2) — the combination with the most MDRs after ambiguous fragments are removed — reproduces the reported score of 1.8333 (5/6 + 1). Finally, record the MaxComb sweep accuracies and the salbutamol ranking into a results table for comparison against the paper's reported outputs.

## Related tools

- **FSA** (Enumerates candidate fragment and precursor formulas, generates MDR-based formula subsets under the MaxComb=1 rule, and scores precursor formulas via Eq. 4 for the rank#1 validation) — https://github.com/kslynn128171/FSA

## Evaluation signals

- Rank#1 accuracy curve from the MaxComb sweep must peak at MaxComb=1, as in Supplementary Fig S1
- Salbutamol precursor C13H21NO3 (or its protonated form) must rank #1 in the FSA candidate enumeration
- Score computed for C13H21NO3 must match the reported 1.8333 (5/6 + 1 per Eq. 4) exactly to 4 decimal places
- After CF determination, ambiguous daughter fragments (those with multiple MDR combinations) must be excluded from subset generation
- FSA per-spectrum processing time should be on the order of 0.024 s as reported for the benchmark

## Limitations

- FSA's rank#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS because it does not use structural information
- Few chemical rules are implemented to filter unfeasible formulas, which can matter for certain compound species and adducts
- The bundled PubChem snapshot is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so formula candidates may be missing
- Validation covered only compounds composed of C, H, O, N, S, and P; extension to more complex molecules and adduct types is future work

## Evidence

- [results] salbutamol worked example: [evidence span withheld — anchor status: not_found]
