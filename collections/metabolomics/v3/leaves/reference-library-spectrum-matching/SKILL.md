---
name: reference-library-spectrum-matching
description: Use when a 1H NMR mixture spectrum must be decomposed into molar proportions
  of known library components (e.g., BCAAs leucine/isoleucine/valine), especially
  with overlapping, shifted, or distorted peaks.
when_to_use_negative:
- The components of the mixture are unknown and no reference library spectra are available
  — Magnetstein requires candidate component spectra in the library.
- Absolute (not relative) concentrations are needed and no quantitative external-standard
  spectrum is available to add algebraically to the mixture spectrum.
- The spectrum is a clean, well-separated 1:10 two-component mixture where simpler
  peak-based tools suffice (though Magnetstein still achieves AvRE 0.0% there).
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Magnetstein
  role: 'Primary quantification engine: treats mixture and library spectra as probability
    distributions and solves proportions via optimal transport; tunable κmixture and
    κcomponents parameters.'
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MasserFront
  role: React and Flask-based UI application associated with Masserstein-family result
    analysis; cloned with submodules to serve the analysis interface.
  repo: https://gitlab.mimuw.edu.pl/ag181044/masserfront
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:bd6118c8ceb41ea5f6b9f68bbfd8a2b56dad29fc785320056a7694fcdb8ed4cd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - Magnetstein
  - MasserFront
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reference-library-spectrum-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify the molar composition of a 1H NMR mixture by matching the sample spectrum against reference library spectra of known components using Magnetstein, which treats spectra as probability distributions and solves quantification via optimal transport theory.

## When to use

Use when a 1H NMR mixture spectrum must be decomposed into molar proportions of known library components (e.g., BCAAs leucine/isoleucine/valine), especially with overlapping, shifted, or distorted peaks.

## When NOT to use

- The components of the mixture are unknown and no reference library spectra are available — Magnetstein requires candidate component spectra in the library.
- Absolute (not relative) concentrations are needed and no quantitative external-standard spectrum is available to add algebraically to the mixture spectrum.
- The spectrum is a clean, well-separated 1:10 two-component mixture where simpler peak-based tools suffice (though Magnetstein still achieves AvRE 0.0% there).

## Inputs

- 1H NMR spectrum of the unknown mixture (deposited experimental data, e.g., BCAA dietary supplement spectrum)
- Reference library 1H NMR spectra of the known candidate components (leucine, isoleucine, valine)
- Optional reported reference proportions/errors for validation (e.g., Table S1 values)

## Outputs

- Estimated molar proportions p_i for each library component (e.g., Ile, Leu, Val)
- Relative errors of the estimated proportions vs. reference values
- Result table file (e.g., bcaa_quantification.csv)

## How to apply

Load the deposited mixture 1H NMR spectrum together with reference library spectra for each expected component (e.g., leucine, isoleucine, and valine for the BCAA supplement of Experiment 11). Run Magnetstein, which models spectra as probability distributions of signal intensity and solves the quantification as an optimal transport problem, treating library spectra as candidate components. Start with default settings for the tunable parameters κmixture and κcomponents; if the mixture contains contaminating or unexpected resonances, lower κmixture, and generally prefer κcomponents higher than κmixture. Extract the estimated molar proportions p_i from the output, compute relative errors against reference values (e.g., Ile 0.25 at 4.2%, Leu 0.25 at 4.2%, Val 0.25 at 10.7% relative error in Experiment 11), and write the results to a table such as bcaa_quantification.csv.

## Related tools

- **Magnetstein** (Primary quantification engine: treats mixture and library spectra as probability distributions and solves proportions via optimal transport; tunable κmixture and κcomponents parameters.) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MasserFront** (React and Flask-based UI application associated with Masserstein-family result analysis; cloned with submodules to serve the analysis interface.) — https://gitlab.mimuw.edu.pl/ag181044/masserfront

## Evaluation signals

- Output table contains three molar proportions matching the reported 0.25 (Ile), 0.25 (Leu), and 0.25 (Val) within the reported relative errors of 4.2%, 4.2%, and 10.7% (Table S1).
- Estimated proportions sum to 1 (they represent a normalized composition of the mixture).
- Reproduced relative errors are of the same order as the consolidated reported errors, not order-of-magnitude larger.
- Results are stable under small perturbations of κmixture; large errors at small κmixture indicate the smaller component was assigned to vortex ω.
- CSV output (bcaa_quantification.csv) has one row per component with proportion and relative-error columns populated.

## Limitations

- For small values of κmixture, Magnetstein tends to put the smaller component into vortex ω, causing a large relative error.
- Mixtures contaminated with unexpected resonances (e.g., limonene/α-pinene degradation peaks) require lowering the default κmixture to remove contaminating peaks.
- Optimal performance is configuration-dependent: best results are obtained when κcomponents is higher than κmixture (e.g., Experiment 3).
- No version number, Zenodo DOI, or explicit hosting location is given for the Magnetstein tool in the article.

## Evidence
