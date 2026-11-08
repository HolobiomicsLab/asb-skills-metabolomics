---
name: stable-isotope-tracing-spectra-evaluation
description: Use when selecting an ionization method/reagent gas for GC-MS isotopologue
  detection in stable-isotope labeling experiments, or when [M+H]+ molecular ions
  fragment excessively under the current source settings.
when_to_use_negative:
- When the goal is already quantifying isotopologues with a chosen source (e.g., GC-CI-MS
  with isobutane) rather than choosing among ionization methods — use isoSCAN processing
  instead
- When the data are from LC-MS or other non-GC separations, where the article's EI/CI-reagent-gas
  comparison does not apply
- When the analytes fall outside volatile/derivatizable GC space (the article's methoxyamination
  + MSTFA/TMCS silylation workflow) so no [M+H]+ envelope can be formed
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: isoSCAN
  role: R package used to process the GC-CI-MS data and automatically quantify isotopologues
    after the ionization method has been selected
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Environment in which isoSCAN is installed and run for isotopologue quantification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:cded4c94e96c966e69dad90fc230397e1d086c638b2aceb1d1c080eb54e85e00
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# stable-isotope-tracing-spectra-evaluation

## Summary

A method for systematically comparing GC-MS ionization configurations (EI at 70 eV, low-energy EI at 15 eV, CI with methane, CI with isobutane) by their ability to concentrate isotopologue signal in intact [M+H]+ envelopes, to select the best ionization source for stable-isotope tracing experiments.

## When to use

Use when selecting an ionization method/reagent gas for GC-MS isotopologue detection in stable-isotope labeling experiments, or when [M+H]+ molecular ions fragment excessively under the current source settings.

## When NOT to use

- When the goal is already quantifying isotopologues with a chosen source (e.g., GC-CI-MS with isobutane) rather than choosing among ionization methods — use isoSCAN processing instead
- When the data are from LC-MS or other non-GC separations, where the article's EI/CI-reagent-gas comparison does not apply
- When the analytes fall outside volatile/derivatizable GC space (the article's methoxyamination + MSTFA/TMCS silylation workflow) so no [M+H]+ envelope can be formed

## Inputs

- Raw GC-MS spectra of a reference metabolite (lactate) from 13C-U-glucose-labeled and non-labeled cell cultures acquired under each ionization method
- Spectra for glycolytic pathway and TCA cycle intermediates used for the quantitative reagent-gas comparison
- Metadata per run: ionization technique (EI 70 eV, EI 15 eV, CI-methane, CI-isobutane), MS configuration (Agilent 7000 QqQ LRMS in SIM/NS modes)

## Outputs

- ionization_method_comparison.csv with one row per method (CI-methane, CI-isobutane, EI 70 eV, EI 15 eV) and columns: reagent/technique, envelope behavior, fragmentation assessment, and reported supporting metric/figure

## How to apply

Acquire spectra for a reference metabolite (here lactate) from 13C-U-glucose-labeled and non-labeled cell cultures under each candidate configuration: EI at 70 eV, low-energy EI at 15 eV, CI with methane, and CI with isobutane on an Agilent 7890A GC coupled to a 7000 QqQ (LRMS, SIM and NS acquisition modes). For each method, inspect whether metabolite abundance is concentrated in the [M+H]+ isotopic envelope versus dispersed into fragments, and extract quantitative support (e.g., the [M+H]+ ion was on average >5-fold more intense with isobutane than methane across glycolytic and TCA cycle intermediates, Figure 1C; methane additionally induced fragmentation of [M+H]+). Record the qualitative outcome (suitable spectra for isotopic enrichment detection or not) and the supporting metric/figure per row — EI 70 eV is a hard ionization causing extensive fragmentation, and EI 15 eV still fragments the molecular ion more than CI-isobutane (Suppl. Figure S1). Assemble the rows into ionization_method_comparison.csv with columns for reagent/technique, envelope behavior, fragmentation assessment, and supporting metric/figure, and verify every row traces to an explicit statement in the article text. Isotopologue quantification can then be performed with the isoSCAN R package.

## Related tools

- **isoSCAN** (R package used to process the GC-CI-MS data and automatically quantify isotopologues after the ionization method has been selected) — https://github.com/jcapelladesto/isoSCAN
- **R** (Environment in which isoSCAN is installed and run for isotopologue quantification)

## Examples

```
install.packages("devtools", dependencies=TRUE); library(devtools); install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- Every row of ionization_method_comparison.csv traces to an explicit statement in the article text (no unsourced cells)
- The CI-isobutane row carries the quantitative metric that [M+H]+ was on average >5-fold more intense than with methane (Figure 1C)
- The EI 70 eV row is annotated as hard ionization causing extensive fragmentation in both non-labeled and labeled samples
- The EI 15 eV row cites Suppl. Figure S1 as evidence of greater molecular-ion fragmentation than CI-isobutane
- The table contains exactly four method rows (methane CI, isobutane CI, EI 70 eV, EI 15 eV) with all required columns populated

## Limitations

- The quantitative comparison is grounded in lactate as reference metabolite and intermediates of the glycolytic pathway and TCA cycle; generalization to other compound classes is not established here
- The evaluation depends on derivatized metabolites (methoxyamination then silylation with MSTFA + 1% TMCS) forming detectable protonated molecular ions under CI; non-derivatizable or non-volatile analytes cannot be assessed this way
- Cell culturing details were referenced from previously published work rather than fully described, so exact culture conditions must be sourced separately when replicating

## Evidence

- [results] The [M+H]+ ion was, on average, >5-fold more intense using isobutane than methane for all intermediates of the glycolytic pathway and TCA cycle: [evidence span withheld — anchor status: not_found]
- [results] EI at 70 eV caused extensive fragmentation of compound structures in both non-labeled and labeled samples: [evidence span withheld — anchor status: not_found]
- [results] Low-energy EI at 15 eV also produced greater fragmentation of the molecular ion than CI-isobutane: [evidence span withheld — anchor status: not_found]
- [methods] Instrument configuration used for the LRMS isotopologue comparison across ionization methods: [evidence span withheld — anchor status: not_found]
