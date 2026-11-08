---
name: metabolite-panel-integration-quantitation
description: Use when analyzing standard one-dimensional 1H NMR spectra (e.
when_to_use_negative:
- The input is not a standard 1D 1H NMR spectrum of serum/plasma (e.g., it is already
  a feature/concentration table or a 2D spectrum such as 2D-Jres, which the paper
  only uses for validation rather than primary assignment).
- The metabolites of interest are outside the 22-metabolite targeted panel, or the
  spectra require complex signal deconvolution or spectral baseline fitting that SMolESY-select
  deliberately precludes.
- The spectra are macromolecule-rich CPMG spectra rather than SMolESY-transformed
  data, since SMolESY-select relies on the SMolESY pattern within predefined spectral
  windows and on s/n trade-offs relative to CPMG (at most ~10% less).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SMolESY-select
  role: 'Automated assignment and integration engine: searches SMolESY data for predefined
    spin-system patterns within spectral windows, applies an 11-point moving average
    filter in selected windows, and directly integrates assigned spectral components
    for the 22-metabolite panel.'
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Preprocessing step that transforms the 1D 1H NMR spectra into resolution-enhanced,
    macromolecule-suppressed data on which SMolESY-select performs its pattern search
    and integration.
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to assess the overlap of each spin system's SMolESY pattern with other
    serum/plasma metabolite signals, informing bin narrowing via singlet correlation.
- name: 2D-Jres
  role: Used (together with STOCSY) for validation of the assignments for the 22 metabolites;
    not part of the primary assignment workflow.
- name: principal component analysis (PCA)
  role: Downstream analysis applied to the 22 plasma metabolites' relative concentration
    values produced by the skill, demonstrating reliable performance among challenging
    phenotypes.
- name: Bruker IVDr 600 MHz spectrometer
  role: Instrument used to acquire the 1D 1H NMR spectra that constitute the primary
    input to the skill.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:83ba1554b9ff790886a6da400a15196e62c4cb95788b3b8522c802e441d688f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  - STOCSY
  - principal component analysis (PCA)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# metabolite-panel-integration-quantitation

## Summary

A computational skill for assigning and quantifying a targeted panel of 22 metabolites in 1D 1H NMR spectra of blood serum/plasma by searching SMolESY-enhanced data for predefined spin-system patterns (bins) and directly integrating the assigned spectral components without deconvolution or baseline fitting.

## When to use

Use when analyzing standard one-dimensional 1H NMR spectra (e.g., from a Bruker IVDr 600 MHz spectrometer) of blood serum or plasma to produce reliable metabolite panel measurements for the 22 targeted metabolites, avoiding sample preprocessing, complex signal deconvolution, and spectral baseline fitting.

## When NOT to use

- The input is not a standard 1D 1H NMR spectrum of serum/plasma (e.g., it is already a feature/concentration table or a 2D spectrum such as 2D-Jres, which the paper only uses for validation rather than primary assignment).
- The metabolites of interest are outside the 22-metabolite targeted panel, or the spectra require complex signal deconvolution or spectral baseline fitting that SMolESY-select deliberately precludes.
- The spectra are macromolecule-rich CPMG spectra rather than SMolESY-transformed data, since SMolESY-select relies on the SMolESY pattern within predefined spectral windows and on s/n trade-offs relative to CPMG (at most ~10% less).

## Inputs

- Standard one-dimensional 1H NMR spectra of blood serum or plasma (e.g., Bruker IVDr 600 MHz spectra, as used for the MTBLS395 MetaboLights serum cohort)
- SMolESY-transformed spectra derived from the 1D 1H NMR data (resolution-enhanced, macromolecule-suppressed) produced by the SMolESY algorithm (github.com/pantakis/SMolESY)

## Outputs

- Per-metabolite assignment results for the 22 targeted metabolites, compiled into a single table (e.g., smolesy_select_assignments.csv)
- Directly integrated, assigned spectral components yielding relative concentration values for the 22 metabolites, suitable for downstream analyses such as PCA of the 22 plasma metabolites' relative concentrations

## How to apply

Start with a set of 1D 1H NMR spectra from serum/plasma (e.g., a cohort like MTBLS395) acquired on a 600 MHz instrument, and automatically calibrate all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm. Transform each spectrum via SMolESY (Small Molecule Enhancement SpectroscopY, github.com/pantakis/SMolESY) to obtain resolution-enhanced, macromolecule-suppressed data, then run SMolESY-select: search the SMolESY data for each of 24 predefined 1H NMR spin-system patterns within spectral windows (bins) defined from a training cohort of 4023 plasma/serum samples. Apply a moving average filter spanning 11 points across the high-resolution data of selected windows to raise the s/n of signals for tyrosine, phenylalanine, histidine, creatinine, and choline; where singlet overlap is severe (e.g., glycine, creatinine, choline, DMSO2, acetone, acetic acid), correlate the singlet δ values with signals from the most abundant and frequently occurring serum/plasma metabolites (such as lactic acid and alanine) to narrow bins to a maximum width of 3–5 SMolESY Δv1/2 (>0.01 ppm for the 600 MHz instrument). Finally, align each assigned spin system to a random ppm value within its corresponding predefined spectral window to ease visual detection of misassignment, and directly integrate the assigned spectral components to obtain relative concentrations. Validate against the reported performance band: assignment accuracy greater than 99.5% for all 22 metabolites and a false-positive/nonassigned signal rate of at most 0.45%.

## Related tools

- **SMolESY-select** (Automated assignment and integration engine: searches SMolESY data for predefined spin-system patterns within spectral windows, applies an 11-point moving average filter in selected windows, and directly integrates assigned spectral components for the 22-metabolite panel.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Preprocessing step that transforms the 1D 1H NMR spectra into resolution-enhanced, macromolecule-suppressed data on which SMolESY-select performs its pattern search and integration.) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to assess the overlap of each spin system's SMolESY pattern with other serum/plasma metabolite signals, informing bin narrowing via singlet correlation.)
- **2D-Jres** (Used (together with STOCSY) for validation of the assignments for the 22 metabolites; not part of the primary assignment workflow.)
- **principal component analysis (PCA)** (Downstream analysis applied to the 22 plasma metabolites' relative concentration values produced by the skill, demonstrating reliable performance among challenging phenotypes.)
- **Bruker IVDr 600 MHz spectrometer** (Instrument used to acquire the 1D 1H NMR spectra that constitute the primary input to the skill.)

## Examples

```
python smolesy_select.py --input mtbls395_spectra/ --calibrate-glucose 5.233 --bins 24 --smoothing-window 11 --output smolesy_select_assignments.csv
```

## Evaluation signals

- Assignment accuracy for all 22 metabolites must exceed 99.5% on the validation cohort (e.g., the 8834 blood serum/plasma spectra including MTBLS395).
- False-positive or nonassigned NMR signals for the 22 metabolites must be zero to a maximum of 40 per cohort (a 0–0.45% failure rate).
- Bins for singlets like glycine, creatinine, choline, DMSO2, acetone, and acetic acid must be narrowed to a maximum width of 3–5 SMolESY Δv1/2 (>0.01 ppm at 600 MHz) via correlation with abundant metabolite singlets.
- The moving average filter spanning 11 points must measurably increase the s/n of signals for tyrosine, phenylalanine, histidine, creatinine, and choline relative to the unfiltered SMolESY data.
- The denoised SMolESY signals should retain slightly narrower Δv1/2 than standard 1D 1H NMR, with s/n at most ~10% less than the corresponding CPMG spectra.

## Limitations

- The validation dataset of >8800 serum/plasma samples is not named or given an accession/repository in the article, limiting independent reproduction of the full performance band.
- A GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1), which are not included in the article text; practitioners must rely on the GitHub repository (pantakis/SMolESY-select, GPL-3.0).
- Comparison to fitting and deconvolution approaches cites only qualitative timing (several minutes) without naming the specific competing software, and no software version is stated for SMolESY-select or SMolESY.
- The algorithm was developed and tuned on a specific training cohort (4023 unique plasma/serum samples, 3023 plasma and 1000 serum); performance on cohorts with very different composition or phenotypes is not characterized beyond the reported challenging-phenotype validation.
- SMolESY-select requires only standard 1D 1H NMR spectra and deliberately avoids sample preprocessing, complex deconvolution, and spectral baseline fitting; spectra requiring such preprocessing fall outside the skill's intended scope.

## Evidence

- [abstract] The algorithm was developed using >4000 NMR serum and plasma spectra and evaluated on an additional >8800 spectra, achieving assignment accuracy greater than 99.5% for all 22 metabolites targeted.: "The performance of the algorithm, developed using >4000 NMR
serum and plasma spectra, was evaluated using an additional >8800
spectra, yielding an assignment accuracy greater than 99.5% for all
22"
- [abstract] The approach requires only standard one-dimensional 1H NMR spectra, avoiding sample preprocessing, complex deconvolution, and spectral baseline fitting.: "The approach is highly efficient, requiring only standard
one-dimensional 1H NMR spectra and avoiding the need for
sample preprocessing, complex deconvolution, and spectral baseline
fitting."
- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [results] Search the SMolESY data for each spin system pattern within defined spectral windows (bins): "the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins)"
- [results] Moving average filter spanning 11-points is applied within selected NMR spectral windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data"
- [results] Moving average filter increases s/n of signals for tyrosine, phenylalanine, histidine, creatinine, and choline: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [results] Correlating singlets of glycine, creatinine, choline, and DMSO2 with lactic acid and alanine signals yields smaller bins of maximum width 3–5 SMolESY Δv1/2: "with a maximum width of 3–5 SMolESY Δv1/2 (Figure 2c,d)"
- [results] SMolESY-select aligns each assigned spin system to a random ppm value within its predefined spectral window to ease visual detection of misassignment: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
- [results] Direct integration of assigned spectral components without complex signal deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [results] Deniosed SMolESY signals maintain slightly narrower Δv1/2 than standard 1D 1H NMR, with s/n at most ∼10% less than CPMG spectra: "their s/n is a maximum of ∼10% less than that of the CPMG spectra"
- [results] Bins were defined for 24 1H NMR spin systems using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum): "using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum)"
- [methods] NMR samples were acquired using Bruker IVDr 600 MHz spectrometers: "NMR samples were acquired using Bruker IVDr 600 MHz spectrometers"
- [results] one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395): "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
- [results] The use of SMolESY, with its baseline suppression and resolution enhancement features: "The use of SMolESY, with its baseline suppression and resolution enhancement features"
- [results] overlap of each spin system’s SMolESY pattern with other SM signals using STOCSY: "overlap of each spin system’s SMolESY pattern with other SM signals using STOCSY"
- [results] principal component analysis (PCA) of the 22 plasma metabolites’ relative concentration values: "principal component analysis (PCA) of the 22 plasma metabolites’ relative concentration values"
- [discussion] its use is supported by a graphical user interface (GUI) (Figure S9, Video S1) requiring minimal prerequisite knowledge to operate: "Its use is supported by a graphical user interface (GUI) (Figure S9, Video S1) requiring minimal prerequisite knowledge to operate"
- [discussion] dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set: "dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set"
- [results] 0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
