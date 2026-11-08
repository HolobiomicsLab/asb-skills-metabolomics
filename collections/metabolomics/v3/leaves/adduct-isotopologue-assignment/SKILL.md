---
name: adduct-isotopologue-assignment
description: Use when an Asari-derived feature table must be grouped into empirical
  compounds (empCpds) prior to MS1 or MS2 annotation, so that isotopologues and adducts
  of the same metabolite are annotated as one unit.
when_to_use_negative:
- Input is raw LC-MS data (.raw or .mzML) that has not yet been preprocessed by Asari
  into a feature table — feature detection must come first.
- The goal is MS2 spectral matching (matchms against a MoNA-derived database) rather
  than grouping MS1 features into empirical compounds.
- The workflow requires actual compound identification levels/scores — annotation
  (assigning compound identities to empCpds) is a separate downstream step, not pre-annotation.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: khipu
  role: Grids features into empirical compounds by grouping co-eluting adducts and
    isotopologues under the configured m/z and RT tolerances
- name: pcpfm
  role: Exposes the build_empCpds operation that invokes khipu on the Asari feature
    table and writes empCpd.json
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: MetDataModel
  role: Defines the empirical compound (empCpd) data model, a JSON-compatible, chainable
    computational unit for a tentative metabolite
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:196ce61417c84dbde735be47b942f24cb8541b6e2d9838ca9116239bf1fa6a61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - MetDataModel
  - Python
  - khipu
  - pcpfm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# adduct-isotopologue-assignment

## Summary

Groups co-eluting features (adducts and m+13C3 isotopologues) from an Asari feature table into empirical compounds using khipu via the pcpfm build_empCpds operation, producing empCpd.json that serves as pre-annotation before MS1/MS2 annotation.

## When to use

Use when an Asari-derived feature table must be grouped into empirical compounds (empCpds) prior to MS1 or MS2 annotation, so that isotopologues and adducts of the same metabolite are annotated as one unit.

## When NOT to use

- Input is raw LC-MS data (.raw or .mzML) that has not yet been preprocessed by Asari into a feature table — feature detection must come first.
- The goal is MS2 spectral matching (matchms against a MoNA-derived database) rather than grouping MS1 features into empirical compounds.
- The workflow requires actual compound identification levels/scores — annotation (assigning compound identities to empCpds) is a separate downstream step, not pre-annotation.

## Inputs

- Asari-derived feature table (preferred_Feature_table.tsv or full_feature_table.tsv from the experiment's asari_results export directory)
- Experiment's inferred ionization mode (positive/negative), used to select mode-based common adduct lists
- Optional user overrides for adduct list, isotopologue list, m/z tolerance and RT tolerance

## Outputs

- empCpd.json saved under a user-specified moniker in the experiment's annotations directory, containing empirical compounds with grouped constituent features and their adduct/isotope assignments
- Ungrouped singleton entries for features without inferred adducts

## How to apply

Load the Asari feature table (the full feature table is recommended, from the experiment's asari_results export directory) into the pcpfm pipeline and invoke the build_empCpds operation, which calls khipu to grid features into empirical compounds. Apply the default grouping tolerances for Orbitrap LC data: 5 ppm m/z tolerance (--khipu_mz_tolerance) and 2 seconds retention time tolerance (--khipu_rt_tolerance), with adduct lists appropriate to the experiment's inferred ionization mode, charges up to z=3, and m+13C3 isotopologues (all user-configurable). Save the result under a user-specified empCpd moniker as empCpd.json in the experiment's annotations directory, recording each compound's constituent features and their adduct/isotope assignments. Singleton features without inferred adducts are retained as ungrouped entries, and the pre-annotation prevents degenerate features from being queried separately in later MS1 annotation.

## Related tools

- **khipu** (Grids features into empirical compounds by grouping co-eluting adducts and isotopologues under the configured m/z and RT tolerances)
- **pcpfm** (Exposes the build_empCpds operation that invokes khipu on the Asari feature table and writes empCpd.json) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **MetDataModel** (Defines the empirical compound (empCpd) data model, a JSON-compatible, chainable computational unit for a tentative metabolite) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm build_empCpds --empCpd my_empCpds --khipu_mz_tolerance 5 --khipu_rt_tolerance 2
```

## Evaluation signals

- empCpd.json exists in the experiment's annotations directory and parses as valid JSON.
- Each empirical compound lists its grouped constituent features with explicit adduct and isotopologue assignments.
- Features that could not be assigned an adduct are retained as singleton/un grouped entries rather than discarded.
- Grouping respected the intended tolerances (default 5 ppm m/z and 2 s RT for Orbitrap LC data).
- Downstream MS1 annotation queries each metabolite once per empCpd, not separately per degenerate isotope/adduct feature.

## Limitations

- If the experiment does not resolve isomers, the empirical compound can be any or a mixture of the isomers, so assignment is tentative.
- Default adduct/isotopologue lists and tolerances are geared to Orbitrap LC-MS data; other data types may need user-configured adducts, isotopologues, and tolerances.
- Pre-annotation only assigns adduct/isotope structure — it does not provide compound identity; annotation to authentic compound libraries or spectral databases is a separate step.

## Evidence

- [readme] An empirical compound (empCpd) is a computational unit for a tentative metabolite, based on the experimental measurement.: [evidence span withheld — anchor status: external_doc]
- [readme] The step of assigning features to empirical compound is called pre-annotation: [evidence span withheld — anchor status: external_doc]
- [readme] pre-annotation to group featues to empirical compounds (khipu): "pre-annotation to group featues to empirical compounds (khipu)"
- [intro] We have introduced the concept of empirical compound to describe the measurement of a tentative metabolite: "We have introduced the concept of empirical compound to describe the measurement of a tentative metabolite"
