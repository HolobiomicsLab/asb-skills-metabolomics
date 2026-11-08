---
name: lipid-transition-list-generation
description: Use when designing targeted lipidomics assays in Skyline or stand-alone
  LipidCreator that need user-defined target lists and fragment libraries for PRM
  and MRM experiments across multiple lipid classes.
when_to_use_negative:
- The workflow requires handling coeluting or isobaric lipids, peak area integration,
  or false discovery rate scoring — LipidCreator intentionally does not address these.
- The target involves stereoisomer resolution at large scale, since no isotope-coded
  standards or large-scale methods are available for this.
- The lipid classes of interest fall outside LipidCreator's coverage (e.g. rare archaeal
  lipids, acyl-PG or acyl-PE in E. coli, di- and tetra-acylated cardiolipins) or belong
  to prokaryotic systems beyond the tested scope.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0121
tools:
- name: LipidCreator
  role: Primary tool that computes MS/MS fragment masses, manages stable isotope labeling,
    optimizes collision energy, and exports transition lists and blib spectral libraries
    for targeted lipidomics.
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Host platform into which LipidCreator exports transition lists and blib libraries
    for PRM/MRM assay development and data analysis.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: flipR
  role: Nonlinear regression used to fit fragment-specific dissociation profiles for
    collision energy optimization.
- name: minpack.lm
  role: R package used for Levenberg-Marquardt nonlinear least squares fitting within
    collision energy optimization.
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: R package used for bounded grid search optimization in collision energy model
    fitting.
  repo: https://rdrr.io/cran/nls.multstart/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:b532e57234704e602ab400ea7839b3bfc13726772727e8dbf55db2af5f58bec4
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lipid-transition-list-generation

## Summary

Generates precursor-product transition lists for targeted lipidomics PRM/MRM assays by computing MS/MS fragment masses from LipidCreator's lipid knowledge-base covering over 60 lipid classes, with stable isotope labeling management and collision energy optimization.

## When to use

Use when designing targeted lipidomics assays in Skyline or stand-alone LipidCreator that need user-defined target lists and fragment libraries for PRM and MRM experiments across multiple lipid classes.

## When NOT to use

- The workflow requires handling coeluting or isobaric lipids, peak area integration, or false discovery rate scoring — LipidCreator intentionally does not address these.
- The target involves stereoisomer resolution at large scale, since no isotope-coded standards or large-scale methods are available for this.
- The lipid classes of interest fall outside LipidCreator's coverage (e.g. rare archaeal lipids, acyl-PG or acyl-PE in E. coli, di- and tetra-acylated cardiolipins) or belong to prokaryotic systems beyond the tested scope.

## Inputs

- Lipid species names and lipid classes from the LipidCreator knowledge-base (over 60 lipid classes supported)
- User-defined precursor and fragment ion definitions, including stable isotope labels
- Collision energy model parameters (instrument-specific, with configurable minimum CE threshold)
- Optional calibration fragment measurements for flipR-based CE model fitting

## Outputs

- Transition list of precursor-product ion pairs for PRM/MRM (Skyline small-molecule targets)
- Spectral library in blib format (requires sqlite3; export from the lipid review dialog after activating the collision energy module)
- Collision energy values for each transition, including model-based predicted values where above the minimum threshold

## How to apply

Launch LipidCreator either as a Skyline plugin (Tools > LipidCreator) or in stand-alone mode, then use its lipid knowledge-base to compose targets covering the supported lipid classes and define precursor-product ion pairs. Manage stable isotope labeling within the tool, activate the collision energy module to obtain model-based predicted collision energies above the instrument-specific configurable minimum, and optionally run flipR-based nonlinear regression (using minpack.lm and nls.multstart) with a bounded grid search to fit fragment-specific dissociation profiles. Export the transition list and, after adding some transitions, export a blib spectral library from the lipid review dialog; the stand-alone batch mode computes pairs at roughly 60,000 pairs per second on a standard i5-4310M notebook with 8 GB RAM. Validate that the resulting assay supports unambiguous lipid identification — for unvalidated assays, include multiple fragments and both polarity modes.

## Related tools

- **LipidCreator** (Primary tool that computes MS/MS fragment masses, manages stable isotope labeling, optimizes collision energy, and exports transition lists and blib spectral libraries for targeted lipidomics.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Host platform into which LipidCreator exports transition lists and blib libraries for PRM/MRM assay development and data analysis.) — https://skyline.ms/project/home/software/Skyline/begin.view
- **flipR** (Nonlinear regression used to fit fragment-specific dissociation profiles for collision energy optimization.)
- **minpack.lm** (R package used for Levenberg-Marquardt nonlinear least squares fitting within collision energy optimization.) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (R package used for bounded grid search optimization in collision energy model fitting.) — https://rdrr.io/cran/nls.multstart/

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- The exported transition list contains precursor-product pairs for lipids drawn from the >60 supported lipid classes.
- Pair-generation throughput in stand-alone batch mode falls in the ~10^4 pairs/s range (~60,000 pairs/s on an i5-4310M notebook with 8 GB RAM).
- A blib spectral library export succeeds after adding some transitions with the collision energy module activated (on Linux this validates the SQLite interop build).
- Collision energy values reported match the model-based predictions when above the instrument-specific configurable minimum; below it the software must not substitute predicted values.
- For unvalidated assays, the list contains multiple fragments per lipid and includes both polarity modes to allow unequivocal lipid species identification.

## Limitations

- CE models are currently exclusive to lipid mediators; extending them to other classes requires more structurally diverse lipid standards and CE-dependent fragmentation measurements.
- Coverage has exceptions even within eukaryotes; bacteria such as E. coli and archaea lack standards (e.g. acyl-PG, acyl-PE in E. coli and di- and tetra-acylated cardiolipins).
- Retention time, elution order, and isotope-labeled standards must be considered alongside the transition list to avoid mis-annotations.
- No large-scale method exists to handle stereoisomers due to the lack of isotope-coded standards.
- LipidCreator has been tested with Thermo QExactive HF and Agilent QTOF instruments; other platforms are untested.
- On Linux/Ubuntu, scrollable-area repainting issues may occur under Mono; the bundled native sqlite.Interop.so may fail on non-Debian/Ubuntu distributions, requiring a custom build.
- There is no dedicated FDR scoring algorithm; downstream quantification quality depends on transition-list curation rather than built-in scoring.

## Evidence

- [abstract] computation of transition lists: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [results] batch generation rate: [evidence span withheld — anchor status: not_found]
- [discussion] collision energy model: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [readme] blib export step: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [discussion] scope limitation on FDR: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
