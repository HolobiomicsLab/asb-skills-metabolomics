---
name: r-package-function-export-and-example-data-execution
description: Use when a study plan requires invoking a specific exported R function
  from an installed package on the package's own example data to reproduce or validate
  a reported annotation outcome before applying the function to new data.
when_to_use_negative:
- The input is already a curated peak/fragment table produced by the package itself,
  so re-executing function export and example execution adds no value
- The task requires annotating every detected feature in an untargeted manner; MetaboAnnotatoR
  explicitly focuses on a statistically pre-selected subset of features, so this skill
  is the wrong choice for full-dataset annotation sweeps
- The data were acquired on a non-AIF DIA scheme (e.g. SWATH) or on an Orbitrap with
  higher mass accuracy that the article notes was not validated; the skill's example
  data are AIF-type LC-MS centroid chromatograms
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnnotatoR
  role: R package whose exported functions (e.g. checkIsotope) and bundled example
    data (xset/RC, targetTable.csv, LipidPos library) are used to reproduce isotopologue-type
    assignment
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime language in which the package is installed, functions exported from
    the namespace, help pages read, and example objects executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:f2d4cf5c47005b124951d226e3e7e9afcf246c642608ba9e6a10520b9bbd3c9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# R package function export and example data execution

## Summary

A workflow skill for untargeted LC-MS feature annotation in R: load an open-source R package (e.g. MetaboAnnotatoR), export functions from its namespace, read their documented arguments, and execute them against the package's bundled example data (xset/RC object, targetTable.csv, and the LipidPos fragment library) to reproduce a published result such as isotopologue-type assignment.

## When to use

Use when a study plan requires invoking a specific exported R function from an installed package on the package's own example data to reproduce or validate a reported annotation outcome before applying the function to new data.

## When NOT to use

- The input is already a curated peak/fragment table produced by the package itself, so re-executing function export and example execution adds no value
- The task requires annotating every detected feature in an untargeted manner; MetaboAnnotatoR explicitly focuses on a statistically pre-selected subset of features, so this skill is the wrong choice for full-dataset annotation sweeps
- The data were acquired on a non-AIF DIA scheme (e.g. SWATH) or on an Orbitrap with higher mass accuracy that the article notes was not validated; the skill's example data are AIF-type LC-MS centroid chromatograms

## Inputs

- Installed R package and its exported function namespace (e.g. checkIsotope from MetaboAnnotatoR)
- Bundled example MS1 extracted-ion-chromatogram / processed chromatogram objects (xset/RC from xcms/RAMClustR)
- Example target feature table (targetTable.csv) with feature m/z and RT
- Fragment library shipped with the package (e.g. LipidPos .txt/.msp fragment entries)

## Outputs

- Assigned isotopologue label per example feature (M+0, M+1, M+2, M+3 or M+0 when no heavier isotopologue is found)
- Documented function-argument mapping (e.g. checkIsotope(fmz, frt, spec, mztol)) ready to be reused on new data
- Manually flagged list of example features with ambiguous EIC correlation or intensity ordering

## How to apply

Install the package from GitHub using devtools::install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE), load it, and enumerate its exported functions (e.g. checkIsotope for isotopologue identification). Read each function's help page to confirm expected arguments — for checkIsotope these are the target feature's m/z and RT, the MS1 extracted-ion-chromatogram (EIC) table, and an m/z tolerance — then invoke the function on the bundled example objects (the xset/RC processed chromatograms, targetTable.csv targets, and the LipidPos library) rather than on user data first. The rationale is that step 3 of the MetaboAnnotatoR workflow identifies the monoisotopic peak by comparing the target's intensity with peaks at target m/z −1 Da, −2 Da, and −3 Da within the same RT window, iterating to the highest-intensity correlated EIC apex, so exercising the function on example data confirms both the argument mapping and the intensity-ordering logic before real runs. Verify execution by inspecting the returned isotopologue label (M+0 through M+3) for each example feature and confirming that the monoisotopic candidate has the highest intensity at the earliest eluting apex among correlated EICs; flag ambiguous correlation patterns for manual review instead of forcing a label.

## Related tools

- **MetaboAnnotatoR** (R package whose exported functions (e.g. checkIsotope) and bundled example data (xset/RC, targetTable.csv, LipidPos library) are used to reproduce isotopologue-type assignment) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime language in which the package is installed, functions exported from the namespace, help pages read, and example objects executed)

## Examples

```
install.packages("devtools"); library(devtools); install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); library(MetaboAnnotatoR); checkIsotope(fmz, frt, spec, mztol)
```

## Evaluation signals

- checkIsotope returns an isotopologue label in the expected range M+0 to M+3 for each example feature, consistent with the published workflow step 3
- For each assigned feature the monoisotopic candidate peak has higher intensity than the target peak at target m/z −1 Da (or the iteration moved to −2/−3 Da and the same monotonic intensity condition holds)
- The EICs used for assignment fall within the same RT window as the target feature and show correlated peak shape with the target EIC
- Ambiguous features (weak correlation or non-monotonic intensity ordering across −1 Da steps) are flagged rather than forcibly labelled
- The function was invoked with arguments matching its exported signature (fmz, frt, spec, mztol) and help-page documentation, not by guessed positional arguments

## Limitations

- Installation can fail due to dependency conflicts (e.g. 'mzR has been built against a different Rcpp version'); mitigate with Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS="true") or INSTALL_opts c('--no-multiarch','--no-lock')
- The article states that annotation of AIF datasets acquired on higher-mass-accuracy instruments such as Orbitraps was not tested, so example-data conclusions may not generalise to those acquisitions
- Isotopologue assignment depends on complete pseudo-MS/MS reconstruction; co-eluting compounds with similar chemical groups can produce nonspecific fragment ions or poorly detected fragments, making EIC correlation ambiguous
- Only AIF is currently supported; other DIA schemes such as SWATH are future developments, so example-data execution is not valid for SWATH inputs

## Evidence

- [results] Step 3 of the workflow identifies the monoisotopic peak by comparing the target feature's intensity with the peak at target m/z minus 1 Da: [evidence span withheld — anchor status: not_found]
- [results] this mechanism is exposed via the exported checkIsotope function (checkIsotope(fmz, frt, spec, mztol)): "checkIsotope(fmz, frt, spec, mztol)"
- [methods] Run checkIsotope on the example dataset features and record the assigned isotopologue type for each tested feature: [evidence span withheld — anchor status: not_found]
- [readme] Package installation instructions from the README: [evidence span withheld — anchor status: external_doc]
- [readme] README note on installation issues caused by dependency conflicts: [evidence span withheld — anchor status: external_doc]
