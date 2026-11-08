---
name: mass-tolerance-based-peak-matching
description: Use when ranking candidate molecular formulas for LC-MS/MS spectra by
  matching fragment peak masses against formula-derived masses under dataset-specific
  tolerances, as in the FSA pipeline across eight test datasets (HMDB_QTOF, HMDB_ITFT,
  HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT).
when_to_use_negative:
- When structural information is needed to disambiguate isomeric formulas — the article
  notes FSA 'has not utilized structural information yet and therefore its accuracy
  may not be competitive with some of the state-of-the-art identification tools'
- When compounds contain elements beyond C, H, O, N, S, and P, since 'FSA has so far
  been validated on datasets containing compounds composed of C, H, O, N, S, and P'
- When peak-formula assignment depends on species/adduct-specific chemical rules that
  are not yet implemented in FSA
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: FSA
  role: Core formula-subset algorithm that performs tolerance-based mass matching
    and formula ranking on selected fragment peaks
  repo: https://github.com/kslynn128171/FSA
- name: ProteoWizard
  role: Format conversion of raw vendor MS/MS files (msconvert) into open formats
    usable downstream
- name: PubChem
  role: Source of publicly available chemical formulas and their monoisotopic molecular
    weights used as candidate reference space
- name: ChemSpider
  role: Alternative source of publicly available chemical formulas and monoisotopic
    weights for candidate generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:9abadc2c5264c0b680b1fb2ef4b2ef263b6084956eaad10501d735014a93396c
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
  - ChemSpider
  - FSA
  - ProteoWizard
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-tolerance-based-peak-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches observed LC-MS/MS fragment peak m/z values to candidate chemical formulas by applying dataset-specific mass matching tolerances within the FSA (formula subset analysis) workflow, using only the top-30 most abundant fragments per spectrum.

## When to use

Use when ranking candidate molecular formulas for LC-MS/MS spectra by matching fragment peak masses against formula-derived masses under dataset-specific tolerances, as in the FSA pipeline across eight test datasets (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT).

## When NOT to use

- When structural information is needed to disambiguate isomeric formulas — the article notes FSA 'has not utilized structural information yet and therefore its accuracy may not be competitive with some of the state-of-the-art identification tools'
- When compounds contain elements beyond C, H, O, N, S, and P, since 'FSA has so far been validated on datasets containing compounds composed of C, H, O, N, S, and P'
- When peak-formula assignment depends on species/adduct-specific chemical rules that are not yet implemented in FSA

## Inputs

- Qualified MS/MS spectra from the eight deposited test datasets (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPIDS_OT, CASMI_OT)
- Ground-truth chemical formulas for each metabolite/spectrum
- Dataset-specific mass-matching tolerance parameters from the paper
- Chemical formula database with monoisotopic molecular weights (e.g., PubChem, ChemSpider)

## Outputs

- Ranked list of candidate molecular formulas per spectrum
- Per-dataset rank-1, top-3, and top-5 accuracy percentages at the spectrum and metabolite levels
- Rank of the correct formula for each spectrum

## How to apply

Load qualified MS/MS spectra and ground-truth formulas for each dataset, then select the top-30 most-abundant fragment peaks per spectrum for analysis. Run FSA using the dataset-specific mass-matching tolerances reported in the paper to generate ranked candidate molecular formulas drawn from PubChem/ChemSpider formula-monoisotopic-weight pairs. Record the rank of the correct formula per spectrum and compute the percentage ranked #1, within top-3, and within top-5 per dataset. Validate against reported Table 2 values (85.28% rank-#1 at the metabolite level, 97.35% top-5, over 30,690 spectra from 6792 metabolites).

## Related tools

- **FSA** (Core formula-subset algorithm that performs tolerance-based mass matching and formula ranking on selected fragment peaks) — https://github.com/kslynn128171/FSA
- **ProteoWizard** (Format conversion of raw vendor MS/MS files (msconvert) into open formats usable downstream)
- **PubChem** (Source of publicly available chemical formulas and their monoisotopic molecular weights used as candidate reference space)
- **ChemSpider** (Alternative source of publicly available chemical formulas and monoisotopic weights for candidate generation)

## Examples

```
python fsa_rank.py --spectrum <input.mzML> --top-n-fragments 30 --tolerance <dataset-specific> --formula-db pubchem --output ranks.csv
```

## Evaluation signals

- Rank-#1 accuracy at the metabolite level should reach ~85.28% and top-5 ~97.35% across 30,690 spectra from 6792 metabolites when reproducing Table 2
- Top-3 spectrum-level accuracy should be ~96.56% and metabolite-level ~95.27%
- Average per-spectrum processing time should be on the order of 0.024 s, flagging deviations as misconfiguration
- Correct formula rank should be recorded per spectrum and aggregated per dataset; any dataset deviating substantially from Table 2 values warrants tolerance-parameter review
- All fragment peaks beyond the top-30 most abundant per spectrum must be excluded before matching

## Limitations

- Few chemical rules are implemented to rank or filter out unfeasible formulas; species/adduct-specific rules not yet incorporated, which can be crucial for certain compound species and adducts
- FSA has not utilized structural information, limiting accuracy versus state-of-the-art tools such as Buddy and SIRIUS
- The embedded PubChem snapshot is outdated (109,383,767 compounds vs. 112,419,402 as of 2022/11/27), which can affect candidate coverage
- Validated only on compounds composed of C, H, O, N, S, and P; extension to more complex molecules and diverse adduct types is future work
- Formulas for fragment peaks without a PubChem match require manual derivation by adding/subtracting known adducts and neutral losses

## Evidence

- [methods] top-30 fragment selection rationale: [evidence span withheld — anchor status: not_found]
- [methods] tolerance application: [evidence span withheld — anchor status: not_found]
