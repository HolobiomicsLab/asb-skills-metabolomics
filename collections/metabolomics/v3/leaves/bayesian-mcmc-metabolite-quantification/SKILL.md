---
name: bayesian-mcmc-metabolite-quantification
description: Use when quantifying metabolites in 1H NMR plasma spectra with a Bayesian
  approach, or when benchmarking a deep-learning or targeted-profiling quantification
  method against an MCMC-based baseline.
when_to_use_negative:
- A deep-learning pipeline is desired instead of a Bayesian baseline (NMRQNet was
  designed because 'automatic and accurate metabolite quantification remains an unsolved
  question' and outperformed Batman)
- Targeted profiling with a fixed ROI library is preferred and manual library tuning
  for MCMC is impractical
- The sample matrix is not plasma NMR (Batman was benchmarked here on human EDTA plasma
  spectra)
edam_operation: http://edamontology.org/operation_3799
edam_topics: []
tools:
- name: Batman
  role: Bayesian MCMC-based quantification engine; fits spectra with a customized
    31-metabolite library to produce reconstructed spectra and concentrations for
    the Fig. 4a comparison
- name: NMRQNet
  role: Provides the preprocessed spectra and the candidate metabolite list used to
    customize the Batman library; its EDTA plasma tutorial (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb)
    defines the shared analysis context
  repo: github.com/LiuzLab/NMRQNet
- name: rDolphin
  role: Comparator baseline run as targeted profiling with the blood ROI library at
    0.02 ppm tolerance on the same 40 spectra
  repo: danielcanueto/rDolphin
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:1b0c0ab1ebd17df38e3f42d5f0bb1e4a67525748ebb2daafbc1b8bcc64aee886
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - Batman
  - NMRQNet
  - rDolphin
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bayesian-mcmc-metabolite-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Bayesian metabolite quantification of 1H NMR spectra via Batman, which customizes the reference metabolite library based on specialized knowledge and fits spectra using Markov chain Monte Carlo (MCMC), producing reconstructed spectra and concentration estimates for benchmarking against other quantification methods.

## When to use

Use when quantifying metabolites in 1H NMR plasma spectra with a Bayesian approach, or when benchmarking a deep-learning or targeted-profiling quantification method against an MCMC-based baseline.

## When NOT to use

- A deep-learning pipeline is desired instead of a Bayesian baseline (NMRQNet was designed because 'automatic and accurate metabolite quantification remains an unsolved question' and outperformed Batman)
- Targeted profiling with a fixed ROI library is preferred and manual library tuning for MCMC is impractical
- The sample matrix is not plasma NMR (Batman was benchmarked here on human EDTA plasma spectra)

## Inputs

- 1H NMR spectra as a csv file (first column chemical shift values, one column per plasma sample), e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv format
- Preprocessed spectra (per NMRQNet data_preprocessing tutorial at https://liuzlab.github.io/NMRQNet/tutorial/data_preprocessing.html)
- Customized Batman metabolite reference library (31 metabolites overlapping NMRQNet's candidates)

## Outputs

- Reconstructed NMR spectra for each sample
- Estimated metabolite relative concentrations per sample
- Per-sample RMSE and Pearson correlation scores (e.g. fig4a_method_comparison.csv)

## How to apply

Configure Batman with a customized metabolite library restricted to metabolites overlapping the candidate list of the comparison method (in the NMRQNet benchmark, 31 metabolites overlapping NMRQNet's candidates). Run Batman's MCMC-based Bayesian fitting on each input spectrum — in the reference comparison, the 40 Mecp2 duplication syndrome (MDS) family plasma spectra loaded from a csv file (first column chemical shift values, one 1H NMR spectrum per sample column). Preprocess the spectra first following the NMRQNet data_preprocessing tutorial so all methods see the same input. For each spectrum, take Batman's reconstructed spectrum and estimated concentrations, then compute RMSE and Pearson correlation between reconstructed and original spectra (optionally within manually annotated peak clusters) to score quantification accuracy. Compare these scores against baselines such as rDolphin targeted profiling (blood ROI library, 0.02 ppm tolerance) and NMRQNet.

## Related tools

- **Batman** (Bayesian MCMC-based quantification engine; fits spectra with a customized 31-metabolite library to produce reconstructed spectra and concentrations for the Fig. 4a comparison)
- **NMRQNet** (Provides the preprocessed spectra and the candidate metabolite list used to customize the Batman library; its EDTA plasma tutorial (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) defines the shared analysis context) — github.com/LiuzLab/NMRQNet
- **rDolphin** (Comparator baseline run as targeted profiling with the blood ROI library at 0.02 ppm tolerance on the same 40 spectra) — danielcanueto/rDolphin

## Examples

```
Rscript -e 'library(Batman); batmanfit <- batman(metabolites=31)'  # MCMC Bayesian fit of 40 MDS plasma spectra with the customized 31-metabolite library
```

## Evaluation signals

- Per-sample RMSE and Pearson correlation between reconstructed and original spectra fall within manually annotated cluster regions and are recorded for all 40 MDS samples
- Comparison table (fig4a_method_comparison.csv) contains scores for all three methods (Batman, rDolphin, NMRQNet) on the same preprocessed spectra
- Expected benchmark outcome: Batman does not achieve the lowest RMSE and highest Pearson correlation for most samples — NMRQNet does ('significantly better quantification results than the earlier computational methods')
- Batman library is restricted to the 31 metabolites overlapping NMRQNet's candidates, so coverage is consistent across methods
- Reconstructed spectra show consistency across samples is weaker than NMRQNet's ('better consistency among samples')

## Limitations

- Batman 'customizes the library based on specialized knowledge first and uses Markov chain Monte Carlo (MCMC)', so results depend on manual library curation
- In the NMRQNet benchmark, Batman underperformed: NMRQNet 'obtained significantly better quantification results than the earlier computational methods'
- MCMC fitting is computationally heavier than linear-regression alternatives and may be less efficient ('far more efficient than the traditional approach')
- NMRQNet 'reconstructs the whole NMR spectra with very low residuals compared to the original spectra', implying larger residuals for Batman reconstructions

## Evidence

- [intro] Batman customizes the library based on specialized knowledge first and uses Markov chain Monte Carlo (MCMC): "For Bayesian-based methods, Batman customizes the library based on specialized knowledge first and uses Markov chain Monte Carlo (MCMC)"
- [abstract] NMRQNet obtained significantly better quantification results than earlier computational methods: "NMRQNet accurately reconstructed the original spectra and obtained significantly better quantification results than the earlier computational methods."
- [methods] The input file provided to NMRQNet is a csv file.: "The input file provided to NMRQNet is a csv file."
- [intro] we tested on 40 plasma samples from Mecp2 duplication syndrome (MDS) families: "we tested on 40 plasma samples from Mecp2 duplication syndrome (MDS) families"
- [readme] The first column is chemical shift values: "The first column is chemical shift values
- For the rest of columns, each column is the 1H NMR spectrum for one plasma sample"
