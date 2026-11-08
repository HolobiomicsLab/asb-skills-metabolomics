---
name: in-silico-biotransformation-prediction
description: Use when you have a peak-picked mzML file of an LC-HRMS/MS run, the parent
  drug's molecular formula, and need to predict in silico phase I/II biotransformations
  and detect the corresponding metabolites.
when_to_use_negative:
- Input is already a curated metabolite feature table rather than raw peak-picked
  mzML data — MetIDfyR mines mzML spectra itself
- The parent drug's molecular formula is unknown, since MetIDfyR predicts biotransformations
  based solely on the raw formula of the drug of interest
- The MS data is not in mzML format or is not peak-picked
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetIDfyR
  role: Core R script that predicts phase I/II biotransformations from the parent
    drug formula and mines mzML spectra for matching metabolite masses
  repo: github.com/agnesblch/MetIDfyR
- name: MSnbase
  role: R dependency used for mass-spectrometric data mining of the mzML sample files
  repo: github.com/agnesblch/MetIDfyR
- name: Rdisop
  role: R dependency used for accurate-mass/formula handling of predicted biotransformations
  repo: github.com/agnesblch/MetIDfyR
- name: MetApp
  role: Shiny application to generate inputs/configs, launch runs, and visualize the
    out_*.tsv metabolite table and export PDF reports
  repo: https://github.com/GIELCH/MetApp
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02281
    title: 'MetIDfyR: An Open-Source R Package to Decipher Small-Molecule Drug Metabolism
      through High-Resolution Mass Spectrometry'
  source_content_hash: sha256:325ece598ea89b47d11efca4655f9622fa082735a41949f747dc6f51db50408f
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
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# in-silico-biotransformation-prediction

## Summary

Predict and detect phase I/II metabolites of a parent drug directly in LC-HRMS/MS mzML data using MetIDfyR, an open-source R package that generates biotransformations from the raw formula of the drug of interest and mines peak-picked spectra for matching accurate masses.

## When to use

Use when you have a peak-picked mzML file of an LC-HRMS/MS run, the parent drug's molecular formula, and need to predict in silico phase I/II biotransformations and detect the corresponding metabolites.

## When NOT to use

- Input is already a curated metabolite feature table rather than raw peak-picked mzML data — MetIDfyR mines mzML spectra itself
- The parent drug's molecular formula is unknown, since MetIDfyR predicts biotransformations based solely on the raw formula of the drug of interest
- The MS data is not in mzML format or is not peak-picked

## Inputs

- Peak-picked LC-HRMS/MS mzML file containing the sample (e.g. input/S9_Diclofenac_PhI mzML, input/LGD_DIA_peak-picked.mzML)
- TSV file with parent drug formula information (TEMPLATE_start_mlc.tsv)
- R configuration file with mass and biotransformation search parameters (TEMPLATE_config.R, e.g. config_diclofenac.R, config_LGD.R)

## Outputs

- out_*.tsv table in the results directory listing detected metabolites with m/z, retention time, assigned formula, and mass error
- Optional PDF report of selected metabolites generated via MetApp

## How to apply

Prepare three inputs: a configuration file with run parameters (following TEMPLATE_config.R, e.g. config_diclofenac.R or config_LGD.R), a peak-picked mzML file containing the sample data, and a TSV with the parent drug's formula (following TEMPLATE_start_mlc.tsv). Run MetIDfyR via command line with the -i (input TSV), -o (output directory), and -c (config) arguments; the package uses MSnbase to mine the mzML spectra and Rdisop for accurate-mass/formula handling, matching predicted biotransformation masses against observed features. Filter candidate assignments by accurate-mass tolerance and retention-time consistency relative to the parent compound. Collect the per-metabolite results (m/z, retention time, assigned formula, mass error) in the out_*.tsv file, and optionally inspect/visualize hits with the MetApp Shiny application.

## Related tools

- **MetIDfyR** (Core R script that predicts phase I/II biotransformations from the parent drug formula and mines mzML spectra for matching metabolite masses) — github.com/agnesblch/MetIDfyR
- **MSnbase** (R dependency used for mass-spectrometric data mining of the mzML sample files) — github.com/agnesblch/MetIDfyR
- **Rdisop** (R dependency used for accurate-mass/formula handling of predicted biotransformations) — github.com/agnesblch/MetIDfyR
- **MetApp** (Shiny application to generate inputs/configs, launch runs, and visualize the out_*.tsv metabolite table and export PDF reports) — https://github.com/GIELCH/MetApp

## Examples

```
Rscript MetIDfyR.R -i input/diclofenac-picked.tsv -o Diclofenac_results -c input/config_diclofenac.R
```

## Evaluation signals

- The out_*.tsv file exists in the specified output directory and contains the expected columns: m/z, retention time (RT), formula, and mass error
- Detected metabolites correspond to phase I biotransformation types consistent with the parent drug dataset (e.g. diclofenac phase I metabolites in the S9_Diclofenac_PhI sample)
- Candidate assignments fall within the accurate-mass tolerance set in the config file and show retention-time consistency against the parent compound
- The parent compound itself is detected at its expected m/z and RT, confirming correct config and TSV setup
- MetApp Visualization panel can open the output folder and display the out_....tsv table

## Limitations

- MetIDfyR was developed and tested on R version 3.6.1 and works normally only up to R version 4.0
- Availability of free open-source software for metabolite discovery is still limited; MetIDfyR addresses this but relies on formula-based in silico prediction only
- MetApp has hard-coded default paths (e.g. '/usr/MetIDfyR' and '/datas/output_MetIDfyR/') in server.R that must be edited to match the local configuration

## Evidence

- [abstract] In silico biotransformation prediction from parent formula: "we present MetIDfyR, an open-source and cross-platform R package for in silico drug phase I/II biotransformation prediction and mass-spectrometric data mining"
- [intro] Formula-based prediction scope: "R script to predict and detect metabolites in mass spectrometry data (mzML) based on the raw formula of the drug of interest"
- [methods] CLI invocation pattern: "Rscript MetIDfyR.R -i path-to-input-file -o output-directory -c config-file"
- [methods] Required inputs: "a tsv file containing the parent drug informations (see TEMPLATE_start_mlc.tsv)"
- [readme] R version constraint: [evidence span withheld — anchor status: not_found]
- [readme] Example diclofenac run: "system("Rscript MetIDfyR.R -i input/diclofenac-picked.tsv -o Diclofenac_results -c input/config_diclofenac.R")"
- [abstract] Efficacy finding: "MetIDfyR has proven its efficacy for advanced metabolite identification in semi-complex and complex mixtures in in vitro or in vivo drug studies"
