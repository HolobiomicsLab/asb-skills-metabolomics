---
name: small-molecule-mass-shift-computation
description: Use when you know only a parent drug's molecular formula (e.g. diclofenac
  C14H11Cl2NO2) and need a list of phase I/II metabolite formulas with exact masses
  to search against LC-HRMS/MS (mzML) data.
when_to_use_negative:
- The peak list or feature table already exists and only mass-spectrometric data mining
  of the mzML file is needed rather than de novo biotransformant enumeration
- The parent molecule's structure is unknown beyond the formula and stereochemistry/structure-aware
  metabolite prediction is required — MetIDfyR works from the raw formula only
tools:
- name: MetIDfyR
  role: R package whose biotransformation-prediction engine enumerates phase I/II
    products from the parent formula and computes their exact masses
  repo: github.com/agnesblch/MetIDfyR
- name: Rdisop
  role: R package used to compute the monoisotopic exact mass of each derived biotransformant
    formula
- name: R
  role: Runtime for MetIDfyR (developed and tested on R 3.6.1, works up to R 4.0)
  repo: https://www.r-project.org/
- name: MetApp
  role: Shiny application that can generate the input and configuration files and
    visualize the resulting metabolites table
  repo: https://github.com/GIELCH/MetApp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02281
    title: 'MetIDfyR: An Open-Source R Package to Decipher Small-Molecule Drug Metabolism
      through High-Resolution Mass Spectrometry'
  source_content_hash: sha256:5054bd448d9828513a7c0357e11d3b85df42c88f2abacb34e5d1d8b87c6b2ee7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetApp
  - MetIDfyR
  - R
  - Rdisop
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# small-molecule-mass-shift-computation

## Summary

Enumerate in silico phase I/II biotransformations of a parent drug from its raw molecular formula and compute the monoisotopic exact mass of every product plus its mass shift relative to the parent, as implemented in MetIDfyR's parent-drug prediction step.

## When to use

Use when you know only a parent drug's molecular formula (e.g. diclofenac C14H11Cl2NO2) and need a list of phase I/II metabolite formulas with exact masses to search against LC-HRMS/MS (mzML) data.

## When NOT to use

- The peak list or feature table already exists and only mass-spectrometric data mining of the mzML file is needed rather than de novo biotransformant enumeration
- The parent molecule's structure is unknown beyond the formula and stereochemistry/structure-aware metabolite prediction is required — MetIDfyR works from the raw formula only

## Inputs

- Parent drug molecular formula (e.g. C14H11Cl2NO2 for diclofenac) in a tsv file following TEMPLATE_start_mlc.tsv
- MetIDfyR configuration file defining the phase I/II biotransformation rules (TEMPLATE_config.R)
- R environment with Rdisop (and pacman, MSnbase, optparse per MetIDfyR dependencies)

## Outputs

- Tab-separated parent-drug table listing each biotransformant's formula, monoisotopic exact mass, and mass shift relative to the parent
- Enriched input consumed by the mzML mass-spectrometric data-mining step of MetIDfyR

## How to apply

Parse the parent formula from the parent-drug tsv file (TEMPLATE_start_mlc.tsv format), then apply each phase I and phase II biotransformation rule defined in the MetIDfyR configuration file (TEMPLATE_config.R) to derive product formulas. Compute the monoisotopic exact mass of every derived formula with Rdisop and record the mass shift relative to the parent drug. Write the enumerated biotransformants — formula, exact mass, and mass shift — as a parent-drug table consumed downstream when MetIDfyR mines the mzML data. This is the prediction half of MetIDfyR, which runs via the MetIDfyR.R command-line entry point with -i (input), -o (output directory), and -c (config) arguments.

## Related tools

- **MetIDfyR** (R package whose biotransformation-prediction engine enumerates phase I/II products from the parent formula and computes their exact masses) — github.com/agnesblch/MetIDfyR
- **Rdisop** (R package used to compute the monoisotopic exact mass of each derived biotransformant formula)
- **R** (Runtime for MetIDfyR (developed and tested on R 3.6.1, works up to R 4.0)) — https://www.r-project.org/
- **MetApp** (Shiny application that can generate the input and configuration files and visualize the resulting metabolites table) — https://github.com/GIELCH/MetApp

## Examples

```
Rscript MetIDfyR.R -i input/diclofenac-picked.tsv -o Diclofenac_results -c input/config_diclofenac.R
```

## Evaluation signals

- The enumerated biotransformant list covers both phase I and phase II transformations defined in the config (e.g. S9_Diclofenac_PhI and S9_Diclofenac_PhII test cases)
- Each predicted formula is chemically consistent and its Rdisop monoisotopic exact mass equals the parent drug's exact mass plus the defined transformation mass shift
- The parent-drug tsv matches the TEMPLATE_start_mlc.tsv schema expected by MetIDfyR's -i argument
- Downstream, the out_....tsv produced after mzML mining contains metabolite hits whose exact masses correspond to entries in the predicted biotransformant table

## Limitations

- Prediction is based on the raw drug formula, so structural/stereochemical constraints on which biotransformations are chemically plausible are not enforced
- MetIDfyR was developed and tested on R 3.6.1 and works normally only up to R version 4.0; newer R versions may break dependencies
- Coverage is limited to the phase I/II biotransformation rules encoded in the configuration file (TEMPLATE_config.R)

## Evidence
