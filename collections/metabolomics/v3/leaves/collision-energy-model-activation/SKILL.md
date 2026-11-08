---
name: collision-energy-model-activation
description: Use when building PRM/MRM transition lists in LipidCreator where fragment-specific
  collision energies must be predicted by the flipR nonlinear regression model (minpack.lm
  / nls.multstart) and exported with the transition list rather than hand-tuned per
  precursor.
when_to_use_negative:
- Your target lipid class has no structurally diverse standards or CE-dependent fragmentation
  measurements — the CE models in LipidCreator are currently exclusive to lipid mediators
  and extending them requires new standards and CE-dependent measurements
- You want LipidCreator to assign a false-discovery-rate score or resolve coeluting/isobaric
  lipids and peak areas — the tool intentionally does not provide a dedicated scoring
  algorithm for this
- You are targeting stereoisomer-resolved assays at scale — no large-scale methods
  are available due to the lack of isotope-coded standards
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: LipidCreator
  role: Hosts the collision energy module in the lipid review dialog; applies the
    flipR-derived CE model to transitions and exports the resulting transition list
    as a .blib spectral library
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Consumes the exported .blib to perform spectral library matching and validate
    fragment ion traces in DIA/targeted lipidomics method preparation
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: flipR
  role: Fits fragment-specific collision-energy dissociation profiles with an extended
    log-normal kernel; supplies the model behind LipidCreator's CE predictions
- name: minpack.lm
  role: Levenberg-Marquardt nonlinear least-squares optimizer used by flipR to fit
    the CE dissociation model
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Performs the bounded grid search optimization used by flipR when fitting CE-dependent
    fragmentation data
  repo: https://rdrr.io/cran/nls.multstart/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:d8bcd2c708d6c011753ae5ddedba18d05e7ab4e29da95f7cdf3ac6ba6cf42f0e
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
  - sqlite3
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# collision-energy-model-activation

## Summary

Enable LipidCreator's collision energy module so that every transition in a targeted lipidomics assay carries a model-based, instrument-specific collision energy before export to a Bibliospec (.blib) spectral library or Skyline method.

## When to use

Use when building PRM/MRM transition lists in LipidCreator where fragment-specific collision energies must be predicted by the flipR nonlinear regression model (minpack.lm / nls.multstart) and exported with the transition list rather than hand-tuned per precursor.

## When NOT to use

- Your target lipid class has no structurally diverse standards or CE-dependent fragmentation measurements — the CE models in LipidCreator are currently exclusive to lipid mediators and extending them requires new standards and CE-dependent measurements
- You want LipidCreator to assign a false-discovery-rate score or resolve coeluting/isobaric lipids and peak areas — the tool intentionally does not provide a dedicated scoring algorithm for this
- You are targeting stereoisomer-resolved assays at scale — no large-scale methods are available due to the lack of isotope-coded standards

## Inputs

- LipidCreator lipid review dialog containing a user-defined set of transitions for target lipid classes/mediators
- Instrument configuration (e.g. Thermo QExactive HF, Agilent QTOF) with a user-defined minimum collision energy threshold
- Collision-energy-dependent fragmentation measurements for lipid mediator standards used to fit the flipR log-normal model

## Outputs

- Transition list annotated with per-transition model-based collision energies
- Bibliospec .blib spectral library file consumable by Skyline
- Skyline targeted (PRM/MRM) or DIA method with spectral library backing for fragment ion trace validation

## How to apply

In LipidCreator, activate the collision energy module in the lipid review dialog after adding the desired transitions for your lipid classes (over 60 supported) so that each transition is annotated with its model-based CE instead of a default value. The CE model derives from flipR, which fits fragment-specific dissociation profiles with an extended log-normal kernel using Levenberg-Marquardt nonlinear least squares (minpack.lm) plus bounded grid search (nls.multstart); the model currently covers lipid mediators measured on instruments such as the Thermo QExactive HF and Agilent QTOF. Decide on the instrument-specific minimum CE: LipidCreator's instrument configuration lets you define a minimum collision energy above which the software reports the model-predicted values, so set this threshold to match your platform's low-CE behavior. Once CEs are populated, export the transition list as a Bibliospec .blib spectral library (requires sqlite3 on Linux/Ubuntu; on non-Debian distros rebuild libSQLite.Interop.so from sqlite-netFx-source-1.0.111.0 via Setup/compile-interop-assembly-release.sh) and load it into Skyline to drive spectral library matching for DIA or targeted method preparation. Rationale: CE assignment at export time encodes the measured fragmentation behavior into the assay, which is what allows Skyline's dot-product spectral matching to validate the fragment ion traces later.

## Related tools

- **LipidCreator** (Hosts the collision energy module in the lipid review dialog; applies the flipR-derived CE model to transitions and exports the resulting transition list as a .blib spectral library) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Consumes the exported .blib to perform spectral library matching and validate fragment ion traces in DIA/targeted lipidomics method preparation) — https://skyline.ms/project/home/software/Skyline/begin.view
- **flipR** (Fits fragment-specific collision-energy dissociation profiles with an extended log-normal kernel; supplies the model behind LipidCreator's CE predictions)
- **minpack.lm** (Levenberg-Marquardt nonlinear least-squares optimizer used by flipR to fit the CE dissociation model) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Performs the bounded grid search optimization used by flipR when fitting CE-dependent fragmentation data) — https://rdrr.io/cran/nls.multstart/

## Examples

```
mono LipidCreator.exe  # then: lipid review dialog → activate CE module → export .blib
```

## Evaluation signals

- The exported lipid_mediator_assay.blib loads into Skyline without error and reproduces the transition set used in the reported platelet mediator validation
- Every transition in the exported list carries a model-based CE value rather than a default/instrument-generic value, respecting the configured instrument-specific minimum CE
- The .blib opens with sqlite3 and contains CE-bearing spectra consistent with the flipR-fitted log-normal dissociation profiles
- Skyline's spectral library matching (dot-product) yields confident fragment ion traces for the targeted lipid mediator assay
- On Linux, if the precompiled libSQLite.Interop.so fails, a locally rebuilt interop library (from sqlite-netFx-source-1.0.111.0) resolves export failures without changing CE values

## Limitations

- CE models in LipidCreator are currently exclusive to lipid mediators; expanding to other lipid classes requires structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior
- LipidCreator intentionally does not provide a dedicated scoring algorithm for false discovery rates and does not address coeluting/isobaric lipids or peak area integration
- Retention time, elution order of lipids, and isotope-labeled standards must be considered alongside CE values to avoid mis-annotations
- On non-Debian/Ubuntu Linux distributions the precompiled native libSQLite.Interop.so may fail, requiring manual rebuild from the sqlite-netFx source zip
- Mono-based Linux builds may exhibit repainting issues with scrollable UI areas due to incomplete Mono compatibility

## Evidence

- [discussion] the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values.: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [discussion] flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles: "flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm) and nls.multstart ... to perform a bounded grid search optimization: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/)"
- [methods] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [discussion] This requires the availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior: "This requires the availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior"
