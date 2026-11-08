---
name: mass-error-formula-annotation
description: Use when peak-picked LC-HRMS/MS mzML data must be annotated with putative
  metabolite formulas derived solely from the parent drug's raw formula, with mass
  error reported per candidate.
when_to_use_negative:
- Input is raw (not peak-picked) profile mzML data or non-mzML vendor formats, which
  the script does not parse
- No parent drug raw formula is available, since candidate formulas are predicted
  solely from the parent formula
- The goal is de novo metabolite identification unrelated to a known parent drug formula
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetIDfyR
  role: R package that predicts in silico phase I/II biotransformations from the parent
    drug formula and performs mass-spectrometric data mining on mzML data to annotate
    metabolite candidates
  repo: github.com/agnesblch/MetIDfyR
- name: Rdisop
  role: Dependency used by MetIDfyR for exact-mass calculation of predicted metabolite
    formulas
  repo: github.com/agnesblch/MetIDfyR
- name: MSnbase
  role: Dependency used by MetIDfyR to mine the peak-picked mzML mass-spectrometric
    data
  repo: github.com/agnesblch/MetIDfyR
- name: MetApp
  role: Shiny application to visualize the out_....tsv annotation table and generate
    PDF reports of selected metabolites
  repo: https://github.com/GIELCH/MetApp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02281
    title: 'MetIDfyR: An Open-Source R Package to Decipher Small-Molecule Drug Metabolism
      through High-Resolution Mass Spectrometry'
  source_content_hash: sha256:6bfee4a869948edb61e509f7b267a9842dced89ec4dd946673fdbeacfe0ecd05
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MSnbase
  - MetApp
  - MetIDfyR
  - R
  - Rdisop
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mass-error-formula-annotation

## Summary

Annotate metabolite candidates in LC-HRMS/MS (mzML) data by matching measured m/z values against in silico predicted phase I/II biotransformation formulas of the parent drug, using Rdisop-based monoisotopic mass calculation and mass-error filtering to rank candidates.

## When to use

Use when peak-picked LC-HRMS/MS mzML data must be annotated with putative metabolite formulas derived solely from the parent drug's raw formula, with mass error reported per candidate.

## When NOT to use

- Input is raw (not peak-picked) profile mzML data or non-mzML vendor formats, which the script does not parse
- No parent drug raw formula is available, since candidate formulas are predicted solely from the parent formula
- The goal is de novo metabolite identification unrelated to a known parent drug formula

## Inputs

- Peak-picked mzML file containing the LC-HRMS/MS sample data (e.g. LGD_DIA_peak-picked.mzML)
- TSV file with parent drug formula information (TEMPLATE_start_mlc.tsv format)
- R configuration file with run parameters (e.g. config_LGD.R, based on TEMPLATE_config.R)

## Outputs

- Output table out_....tsv with m/z, retention time, predicted formula, and mass error per detected metabolite candidate
- Optional PDF report of selected metabolites generated via MetApp

## How to apply

Start from a TSV containing the parent drug formula (TEMPLATE_start_mlc.tsv format) and a peak-picked mzML file; MetIDfyR generates in silico phase I and II biotransformations of the parent formula and computes their exact masses via Rdisop. It then mines the mzML data with MSnbase, matching detected features against predicted metabolite m/z values and reporting, per candidate, the measured m/z, retention time, predicted formula, and mass error in the output out_....tsv table. All run behavior is controlled through an R configuration file (TEMPLATE_config.R), and the pipeline is invoked from the command line with -i (input TSV), -o (output directory), and -c (config) arguments. Correct application means the output table contains plausible formula assignments with small mass errors for known metabolites of the parent drug; results can be inspected interactively in the MetApp Shiny application and exported to a PDF report of selected metabolites.

## Related tools

- **MetIDfyR** (R package that predicts in silico phase I/II biotransformations from the parent drug formula and performs mass-spectrometric data mining on mzML data to annotate metabolite candidates) — github.com/agnesblch/MetIDfyR
- **Rdisop** (Dependency used by MetIDfyR for exact-mass calculation of predicted metabolite formulas) — github.com/agnesblch/MetIDfyR
- **MSnbase** (Dependency used by MetIDfyR to mine the peak-picked mzML mass-spectrometric data) — github.com/agnesblch/MetIDfyR
- **MetApp** (Shiny application to visualize the out_....tsv annotation table and generate PDF reports of selected metabolites) — https://github.com/GIELCH/MetApp

## Examples

```
Rscript MetIDfyR.R -i input/lgd_DIA_peak-picked.tsv -o LGD4033_results -c input/config_LGD.R
```

## Evaluation signals

- The output directory contains a table matching the out_....tsv pattern with per-candidate m/z, retention time, predicted formula, and mass error columns
- Known metabolites of the test parent drug (e.g. LGD-4033, cocaine, diclofenac) are recovered with small mass errors in the annotation table
- The run completes without dependency errors under R 3.6.1 through R 4.0, with pacman-installed packages MSnbase and Rdisop available
- MetApp Visualization panel successfully loads the output folder and renders putative metabolites identified in the mzML data

## Limitations

- MetIDfyR was developed and tested on R version 3.6.1 and works normally only up to R version 4.0; newer R versions may fail
- Annotation depends entirely on the parent drug's raw formula, so isobaric or formula-independent metabolites are out of scope
- Only the abstract-level claim of efficacy in semi-complex and complex mixtures is stated; no changelog is available for tracking fixes

## Evidence

- [abstract] in silico drug phase I/II biotransformation prediction and mass-spectrometric data mining: "we present MetIDfyR, an open-source and cross-platform R package for in silico drug phase I/II biotransformation prediction and mass-spectrometric data mining"
- [intro] R script to predict and detect metabolites in mzML based on parent formula: "R script to predict and detect metabolites in mass spectrometry data (mzML) based on the raw formula of the drug of interest"
- [methods] CLI invocation with input, output, and config arguments: "Rscript MetIDfyR.R -i path-to-input-file -o output-directory -c config-file"
- [other] Output table contents: m/z, RT, formula, mass error: [evidence span withheld — anchor status: not_found]
- [readme] MetApp visualization of putative metabolites: [evidence span withheld — anchor status: external_doc]
- [readme] R version constraint: [evidence span withheld — anchor status: not_found]
