---
name: adduct-ion-species-prediction
description: Use when you have a list of molecular formulae for LC-IM-MS / drift tube
  IM-MS data and need predicted ion species (primary ion plus adducts, positive or
  negative mode, multiply charged) to build targeted m/z search traces.
when_to_use_negative:
- The data are already a feature table or annotated peak list — this skill predicts
  theoretical species from formulae, not from measured peaks.
- No molecular formulae are available for the compounds of interest; CCSfind only
  accepts formulae as search targets, not structures or spectra.
- The aim is untargeted peak detection without a priori formula targets — isotopologue-confirmed
  targeted search is not designed for that.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: CCSfind
  role: Tool that consumes the predicted ion species (primary ion plus adducts) and
    searches their m/z traces in the parsed mzML file with isotopologue matching confirmation
  repo: https://github.com/sangeeta97/ccs_find.git
- name: molmass
  role: Python library used to compute monoisotopic masses and isotopologue distribution
    patterns from input molecular formulae
- name: pandas
  role: Assembles per-compound/per-adduct mass and isotopologue results into a DataFrame
    written to CSV
- name: Python 3.9
  role: Runtime environment (created via conda) required by CCSfind and its dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:b982f81768da8270588819a147d888a55b68a8d0de09559edf17c3591e44cfaf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - Python 3.9
  - molmass
  - pandas
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# adduct-ion-species-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates theoretical m/z targets — monoisotopic masses and isotopologue distribution patterns (A, A+1, A+2) — for primary ions and additional adducts, including multiply charged variants, directly from input molecular formulae using the molmass library, as implemented in CCSfind.

## When to use

Use when you have a list of molecular formulae for LC-IM-MS / drift tube IM-MS data and need predicted ion species (primary ion plus adducts, positive or negative mode, multiply charged) to build targeted m/z search traces.

## When NOT to use

- The data are already a feature table or annotated peak list — this skill predicts theoretical species from formulae, not from measured peaks.
- No molecular formulae are available for the compounds of interest; CCSfind only accepts formulae as search targets, not structures or spectra.
- The aim is untargeted peak detection without a priori formula targets — isotopologue-confirmed targeted search is not designed for that.

## Inputs

- List of molecular formulae for target compounds (e.g. the eight compounds in Table 1 of the CCSfind study)
- Selected primary ion species (compulsory CCSfind input)
- Ion mode context (positive and/or negative) and whether multiply charged ions should be screened
- mzML profile data file from LC-DTIM-MS that the predicted targets will be searched against

## Outputs

- Per-compound monoisotopic masses and isotopologue distribution patterns for primary ions and adducts (e.g. monoisotopic_masses_isotopologues.csv, a pandas DataFrame)
- Enumerated predicted ion species list (primary ion, additional adducts, multiply charged variants) used as targeted m/z entries for downstream CCS database building

## How to apply

For each input molecular formula, instantiate a molmass Formula object in Python 3.9 and compute the monoisotopic mass from the elemental composition; then use molmass's isotopologue calculation to derive the isotope distribution pattern and enumerate predicted ion species, including the user-selected primary ion, common adducts in positive and negative mode, and multiply charged variants. Assemble per-compound, per-adduct masses and isotopologue entries into a pandas DataFrame (e.g. monoisotopic_masses_isotopologues.csv). The predicted species then drive targeted m/z trace extraction over the retention and arrival time dimensions of the parsed mzML file, where filtering relies on isotopologue matching confirmation (A, A+1, A+2) after peak picking, so the accuracy of the predicted patterns directly controls downstream detection. Rationale: CCSfind requires the primary ion species as a compulsory input, and choosing the correct predicted species set avoids missed detections while the isotopologue filter guards against false positives.

## Related tools

- **CCSfind** (Tool that consumes the predicted ion species (primary ion plus adducts) and searches their m/z traces in the parsed mzML file with isotopologue matching confirmation) — https://github.com/sangeeta97/ccs_find.git
- **molmass** (Python library used to compute monoisotopic masses and isotopologue distribution patterns from input molecular formulae)
- **pandas** (Assembles per-compound/per-adduct mass and isotopologue results into a DataFrame written to CSV)
- **Python 3.9** (Runtime environment (created via conda) required by CCSfind and its dependencies)

## Examples

```
conda create --name ccs_find python=3.9; conda activate ccs_find; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- The output CSV exists and contains rows for all input formulae (e.g. all eight Table 1 compounds).
- Each primary ion and adduct species entry includes a monoisotopic mass field and isotopologue distribution entries.
- Predicted m/z values are consistent with the elemental composition computed by molmass for both singly and multiply charged variants.
- Downstream search traces in the mzML data survive the isotopologue matching confirmation filter (A, A+1, A+2) after peak picking.
- Recomputing masses for a known compound reproduces the reference monoisotopic mass within the configured mass extraction window.

## Limitations

- Prediction is limited to species expressible as formula-based ion species; multiply charged ions must be explicitly enabled via CCSfind's option to search for multiply charged ions.
- The isotopologue confirmation window (mass extraction window for A, A+1, A+2), expected LC peak width, and expected peak abundance must be tuned; misconfigured settings can miss noisy or well-defined peaks.
- Targets are computed from the input formula list only — compounds absent from the list are never searched.

## Evidence

- [results] Isotopologue patterns from molmass: [evidence span withheld — anchor status: not_found]
- [readme] README adduct and charge features: "Supports screening of additional adducts ions for both positive and negative modes."
- [readme] README multiply charged option: "Option to search for multiply charged ions."
- [intro] Isotopologue confirmation filter: "The tool uses a mzML input files to provide a unified set of results within a single data processing step which includes filtering using isotopologue matching confirmation after peak picking."
- [readme] Primary ion species compulsory input: "The primary ion species to be searched for."
