---
name: parent-drug-formula-annotation
description: Use when a peak-picked mzML file from LC-HRMS/MS is available for a drug
  of interest whose exact molecular formula is known, and in silico metabolite prediction
  (e.g., with MetIDfyR) is the goal.
when_to_use_negative:
- The input is already a MetIDfyR output table (out_....tsv) — formula annotation
  is an upstream preparation step
- The molecular formula of the parent drug is unknown or uncertain — in silico biotransformation
  prediction requires the raw formula as its starting point
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetIDfyR
  role: Consumes the parent-drug TSV to predict and detect metabolites in mzML data
    based on the raw formula of the drug of interest
  repo: github.com/agnesblch/MetIDfyR
- name: MetApp
  role: Shiny application that can generate input (parent-drug TSV) and configuration
    files, launch a run, and visualize resulting metabolites
  repo: https://github.com/GIELCH/MetApp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02281
    title: 'MetIDfyR: An Open-Source R Package to Decipher Small-Molecule Drug Metabolism
      through High-Resolution Mass Spectrometry'
  source_content_hash: sha256:e56bfc86ad8c28f7ac3832b7c6f45c595e01b1f90c139004f43328779d0389c3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetApp
  - MetIDfyR
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# parent-drug-formula-annotation

## Summary

Annotate the parent drug of interest (name and raw molecular formula, e.g., cocaine) in a TSV input file so MetIDfyR can perform in silico phase I/II biotransformation prediction and mass-spectrometric data mining against an mzML file.

## When to use

Use when a peak-picked mzML file from LC-HRMS/MS is available for a drug of interest whose exact molecular formula is known, and in silico metabolite prediction (e.g., with MetIDfyR) is the goal.

## When NOT to use

- The input is already a MetIDfyR output table (out_....tsv) — formula annotation is an upstream preparation step
- The molecular formula of the parent drug is unknown or uncertain — in silico biotransformation prediction requires the raw formula as its starting point

## Inputs

- Parent drug identity and exact molecular formula (e.g., cocaine), typically as a TSV modeled on TEMPLATE_start_mlc.tsv
- Peak-picked mzML file containing the sample information (e.g., U_H_COCA_peak-picked.mzML)

## Outputs

- Parent-drug information TSV file ready for MetIDfyR input
- Downstream: out_....tsv detection table with per-candidate metabolite m/z, retention time, predicted formula, and mass error

## How to apply

Retrieve the exact raw molecular formula of the parent drug (e.g., cocaine) from a trusted source and record it along with the parent drug information in a TSV file modeled on the provided TEMPLATE_start_mlc.tsv. This TSV is one of the three required MetIDfyR inputs, alongside a peak-picked mzML file containing the sample information and a configuration file based on TEMPLATE_config.R. The formula is the basis from which MetIDfyR generates in silico phase I/II biotransformations, so an incorrect formula propagates to every predicted metabolite m/z and mass-error value. Once the TSV is prepared, launch the pipeline (Rscript MetIDfyR.R -i ... -o ... -c ...) and inspect the resulting out_....tsv for candidate metabolites with m/z, retention time, predicted formula, and mass error.

## Related tools

- **MetIDfyR** (Consumes the parent-drug TSV to predict and detect metabolites in mzML data based on the raw formula of the drug of interest) — github.com/agnesblch/MetIDfyR
- **MetApp** (Shiny application that can generate input (parent-drug TSV) and configuration files, launch a run, and visualize resulting metabolites) — https://github.com/GIELCH/MetApp

## Examples

```
system("Rscript MetIDfyR.R -i input/cocaine_peak-picked.tsv -o Cocaine_results -c input/config_cocaine.R")
```

## Evaluation signals

- The TSV conforms to the TEMPLATE_start_mlc.tsv schema and contains the exact molecular formula of the parent drug
- MetIDfyR accepts the TSV without input-validation errors and proceeds to in silico phase I/II biotransformation prediction
- Predicted metabolite formulas in out_....tsv are biotransformations consistent with the annotated parent formula
- A non-empty out_....tsv with plausible mass errors is produced (e.g., the cocaine example run on U_H_COCA_peak-picked.mzML)

## Limitations

- MetIDfyR was developed and tested on R 3.6.1 and works normally only up to R 4.0, constraining the environment for the run
- Prediction quality depends entirely on the correctness of the parent drug's raw formula; no formula, no metabolite candidates
- The TSV must be accompanied by a matching configuration file (TEMPLATE_config.R) with run parameters for the specific LC-HRMS/MS acquisition

## Evidence

- [intro] MetIDfyR predicts metabolites from parent formula: "R script to predict and detect metabolites in mass spectrometry data (mzML) based on the raw formula of the drug of interest"
- [readme] Required inputs include parent-drug TSV: "a tsv file containing the parent drug informations (see TEMPLATE_start_mlc.tsv)"
- [readme] Cocaine example run command: "system("Rscript MetIDfyR.R -i input/cocaine_peak-picked.tsv -o Cocaine_results -c input/config_cocaine.R")"
- [readme] Output table contents: [evidence span withheld — anchor status: external_doc]
