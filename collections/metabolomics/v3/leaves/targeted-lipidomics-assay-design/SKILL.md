---
name: targeted-lipidomics-assay-design
description: Use when building user-defined target lists and fragment libraries for
  PRM or MRM lipidomics experiments from a set of lipid classes, or when fragment/collision-energy
  predictions and isotope-labeled transition design are needed before instrument method
  creation.
when_to_use_negative:
- When the analysis goal is untargeted screening of the whole lipidome rather than
  a knowledge-based selection of a subset of lipids of interest.
- When a dedicated false discovery rate scoring algorithm is required — LipidCreator
  intentionally does not provide one and does not address coeluting/isobaric lipids
  or peak area integration.
- When stereoisomers must be resolved at scale — due to lack of isotope-coded standards
  there are currently no large-scale methods to cope with occurring stereoisomers.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LipidCreator
  role: 'Primary tool: computes MS/MS fragment masses for >60 lipid classes, manages
    stable isotope labeling, optimizes collision energy, and generates transition
    lists and in silico spectral libraries for targeted lipidomics.'
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: 'Downstream target: LipidCreator is fully integrated with Skyline for small
    molecules, creating vendor-independent targeted assays for PRM/MRM method development.'
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: flipR
  role: Fits fragment-specific dissociation profiles with an extended log-normal kernel
    for collision energy model building.
- name: minpack.lm
  role: R package used for nonlinear regression (Levenberg-Marquardt least squares)
    in collision energy prediction.
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: R package performing bounded grid search optimization for the CE regression
    models.
  repo: https://rdrr.io/cran/nls.multstart/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:75ad7ac9c4e913c158ca515d0268ec1792049a32d291c2bdad55ff989b7f98c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# targeted-lipidomics-assay-design

## Summary

Design targeted lipidomics assays (PRM/MRM transition lists) with LipidCreator, a knowledge-based workbench and Skyline plugin that computes MS/MS fragment masses for over 60 lipid classes, manages stable isotope labeling, optimizes collision energy, and exports vendor-independent spectral libraries.

## When to use

Use when building user-defined target lists and fragment libraries for PRM or MRM lipidomics experiments from a set of lipid classes, or when fragment/collision-energy predictions and isotope-labeled transition design are needed before instrument method creation.

## When NOT to use

- When the analysis goal is untargeted screening of the whole lipidome rather than a knowledge-based selection of a subset of lipids of interest.
- When a dedicated false discovery rate scoring algorithm is required — LipidCreator intentionally does not provide one and does not address coeluting/isobaric lipids or peak area integration.
- When stereoisomers must be resolved at scale — due to lack of isotope-coded standards there are currently no large-scale methods to cope with occurring stereoisomers.

## Inputs

- Lipid class/species selections (names or shorthand annotations) drawn from LipidCreator's lipid knowledge-base covering >60 lipid classes
- Stable isotope labeling definitions for internal standards
- Instrument-specific collision energy configuration (minimum CE threshold above which model-based values are reported)
- Optionally: fragment dissociation measurements for CE-model fitting (e.g. from Thermo QExactive HF or Agilent QTOF data)

## Outputs

- User-defined targeted transition lists (precursor-product ion pairs) for PRM and MRM experiments
- In silico spectral libraries in blib format (with collision energy annotations) importable into Skyline
- Fragment definitions with stable isotope labels and predicted collision energies
- Large targeted lipidomics experiment designs suitable for blood analysis and lipid-signaling pathway dissection (e.g. >60,000 pairs/s batch generation)

## How to apply

Compose a targeted transition list in LipidCreator by selecting lipids from its knowledge-base — which covers over 60 lipid classes — and let it compute precursor-product ion pairs for both positive and negative polarity modes; for unvalidated assays use multiple fragments and both polarity modes to ensure unequivocal lipid species identification. Manage stable isotope labeling for internal standards, then activate the collision energy module so LipidCreator reports model-based, instrument-specific predicted values above the configurable minimum CE, with fragment-specific dissociation profiles fitted via nonlinear regression (extended log-normal kernel using flipR, or Levenberg-Marquardt via minpack.lm with bounded grid search via nls.multstart). Export the finished assay as a blib spectral library from the lipid review dialog and import it into Skyline for small molecules so the method is vendor-independent; for batch generation LipidCreator's stand-alone mode computes ~60,000 precursor-product pairs per second on a standard notebook. Validate computationally and analytically — e.g. against NIST SRM 1950 plasma or platelet lipid-signaling samples — remembering that retention time, elution order and isotope-labeled standards must be considered to avoid mis-annotations since no dedicated FDR scoring is provided.

## Related tools

- **LipidCreator** (Primary tool: computes MS/MS fragment masses for >60 lipid classes, manages stable isotope labeling, optimizes collision energy, and generates transition lists and in silico spectral libraries for targeted lipidomics.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Downstream target: LipidCreator is fully integrated with Skyline for small molecules, creating vendor-independent targeted assays for PRM/MRM method development.) — https://skyline.ms/project/home/software/Skyline/begin.view
- **flipR** (Fits fragment-specific dissociation profiles with an extended log-normal kernel for collision energy model building.)
- **minpack.lm** (R package used for nonlinear regression (Levenberg-Marquardt least squares) in collision energy prediction.) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (R package performing bounded grid search optimization for the CE regression models.) — https://rdrr.io/cran/nls.multstart/

## Examples

```
mono LipidCreator.exe  # start stand-alone; compose transitions, activate CE module, then export blib from the lipid review dialog
```

## Evaluation signals

- Transition list throughput in stand-alone batch mode reaches ~60,000 precursor-product pairs per second on a standard notebook (i5-4310M @ 2.70 GHz, 8 GB RAM).
- A blib spectral library export from the lipid review dialog (with the collision energy module activated and transitions added) completes successfully — requires sqlite3 installed on Linux.
- Assays are validated computationally and analytically, e.g. direct correlation analysis of NIST SRM 1950 vs Asian control plasma yields R² = 0.98.
- For unvalidated assays in simple model systems (e.g. yeast), both polarity modes and multiple fragments are used so species identification is unequivocal.
- The instrument-specific minimum collision energy is configured so model-based predicted CE values are only reported above that threshold.

## Limitations

- Collision energy prediction models currently cover lipid mediators only; extending to other lipid classes requires more structurally diverse lipid standards and CE-dependent fragmentation measurements.
- Coverage of eukaryotic lipid classes is broad but not complete (Supplementary Data 1 lists exceptions); rare bacterial/archaeal lipids (e.g. acyl-PG, acyl-PE in E. coli) lack standards.
- No dedicated scoring algorithm for false discovery rates; coeluting/isobaric lipids and peak area integration are intentionally out of scope.
- Retention time, lipid elution order and isotope-labeled standards must always be considered to avoid mis-annotations.
- Large-scale stereoisomer handling is not possible due to lack of isotope-coded standards.
- Mono/Ubuntu rendering may have repainting issues in scrollable windows; blib export on Linux needs a working native libSQLite.Interop.so (may require custom compilation).
- Tested instrument coverage is limited to Thermo QExactive HF and Agilent QTOF.

## Evidence

- [abstract] LipidCreator knowledge-base coverage: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [abstract] LipidCreator functionality suite: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
- [methods] Standalone batch throughput benchmark: [evidence span withheld — anchor status: not_found]
- [readme] Blb export prerequisite on Linux: "In order to use the blib spectral library export functionality, please install the appropriate sqlite (>3) package for your distribution."
- [discussion] Scope of CE models: [evidence span withheld — anchor status: not_found]
- [discussion] Validation result: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [discussion] Integration with Skyline: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
- [discussion] CE reporting configuration: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [intro] Purpose of LipidCreator: "we here present a targeted lipidomics workbench and lipid knowledge-base that is fully integrated with Skyline"
