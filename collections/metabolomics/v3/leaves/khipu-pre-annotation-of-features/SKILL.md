---
name: khipu-pre-annotation-of-features
description: Use when you have an Asari-derived LC-MS feature table and need to group
  isotopologues and adducts into empirical compounds (empCpds) as pre-annotation prior
  to MS1 or MS2 annotation.
when_to_use_negative:
- Input is still raw .mzML/.raw data — feature extraction must first be done with
  Asari before grouping into empCpds.
- The goal is final compound identification — khipu performs pre-annotation only;
  annotation assigns compound identity to empirical compounds via MS1 libraries, MS2
  (matchms), or authentic standards.
- Stable isotope tracing experiments requiring isotopologue patterns beyond the default
  m+13C3 set without adjusting the configurable isotope/adduct lists.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: khipu
  role: Grouping engine that grids co-eluting features (isotopologues and adducts)
    into empirical compounds
- name: pcpfm
  role: Pipeline exposing the build_empCpds operation with --khipu_mz_tolerance and
    --khipu_rt_tolerance flags and empCpd moniker output
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: MetDataModel
  role: Defines the JSON-compatible empirical compound (empCpd) data structure produced
    by the grouping
  repo: https://github.com/shuzhao-li-lab/metDataModel
- name: Asari
  role: Upstream preprocessor producing the feature table that is grouped into empirical
    compounds
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:f43199f5cb412ec5babf06982eab52c77b3ff504e34f2d26c8d565328a57306a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - MetDataModel
  - Python
  - khipu
  - pcpfm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# khipu pre-annotation of features into empirical compounds

## Summary

Groups co-eluting Asari features (adducts and isotopologues) into empirical compounds using the khipu package via pcpfm's build_empCpds operation, serving as the pre-annotation step before MS1/MS2 annotation.

## When to use

Use when you have an Asari-derived LC-MS feature table and need to group isotopologues and adducts into empirical compounds (empCpds) as pre-annotation prior to MS1 or MS2 annotation.

## When NOT to use

- Input is still raw .mzML/.raw data — feature extraction must first be done with Asari before grouping into empCpds.
- The goal is final compound identification — khipu performs pre-annotation only; annotation assigns compound identity to empirical compounds via MS1 libraries, MS2 (matchms), or authentic standards.
- Stable isotope tracing experiments requiring isotopologue patterns beyond the default m+13C3 set without adjusting the configurable isotope/adduct lists.

## Inputs

- Asari-derived feature table (full_feature_table.tsv from the asari_results export directory recommended)
- pcpfm experiment directory with annotations/ subdirectory
- User-configurable adduct and isotope lists (defaults: charges up to z=3, m+13C3 isotopologues, mode-based common adducts)

## Outputs

- empCpd.json in the experiment's annotations directory, listing each empirical compound's grouped features and their adduct/isotope assignments, with singletons retained as ungrouped entries

## How to apply

Load the Asari feature table (the full feature table is recommended) from the experiment's asari_results export directory and invoke pcpfm's build_empCpds operation, which calls khipu to grid features into empirical compounds using adduct lists appropriate to the experiment's inferred ionization mode, charges up to z=3, and m+13C3 isotopologues. Apply the default grouping tolerances for Orbitrap LC data: 5 ppm m/z tolerance (--khipu_mz_tolerance) and 2 seconds retention time tolerance (--khipu_rt_tolerance), so that co-eluting isotopologues and adducts are grouped together. Adduct and isotope sets are user-configurable if the defaults do not match the chemistry of the experiment. Save the result under a user-specified empCpd moniker as empCpd.json in the experiment's annotations directory, recording each compound's constituent features and their adduct/isotope assignments; singleton features without inferred adducts are retained as ungrouped entries. This pre-annotation also prevents degenerate features from being queried separately in later MS1 annotation.

## Related tools

- **khipu** (Grouping engine that grids co-eluting features (isotopologues and adducts) into empirical compounds)
- **pcpfm** (Pipeline exposing the build_empCpds operation with --khipu_mz_tolerance and --khipu_rt_tolerance flags and empCpd moniker output) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **MetDataModel** (Defines the JSON-compatible empirical compound (empCpd) data structure produced by the grouping) — https://github.com/shuzhao-li-lab/metDataModel
- **Asari** (Upstream preprocessor producing the feature table that is grouped into empirical compounds) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics

## Examples

```
pcpfm build_empCpds --empCpd_name my_empCpds --khipu_mz_tolerance 5 --khipu_rt_tolerance 2
```

## Evaluation signals

- empCpd.json exists in the experiment's annotations directory and parses as valid JSON.
- Each empirical compound contains its grouped features with assigned adducts/isotopologues; singleton features without inferred adducts appear as ungrouped entries.
- Grouped members of an empCpd co-elute within the 2 s RT tolerance and fall within the 5 ppm m/z tolerance.
- Downstream MS1 annotation no longer queries degenerate features (adduct/isotope relatives) separately.
- empCpd.json is chainable and compatible with the metDataModel empirical compound structure for later MS1/MS2/authentic-standard annotation.

## Limitations

- If unresolved isomers are not chromatographically separated, the empirical compound can be any or a mixture of the isomers — pre-annotation does not resolve isomeric identity.
- Default isotopologue coverage is m+13C3; heavier or non-13C labeling requires user reconfiguration of adduct/isotope lists.
- Tolerances (5 ppm m/z, 2 s RT) are tuned for Orbitrap LC data; other instruments may need adjustment.

## Evidence

- [readme] An empirical compound (empCpd) is a computational unit for a tentative metabolite, based on the experimental measurement.: [evidence span withheld — anchor status: external_doc]
- [readme] The step of assigning features to empirical compound is called pre-annotation: [evidence span withheld — anchor status: external_doc]
- [readme] pre-annotation to group featues to empirical compounds (khipu): "pre-annotation to group featues to empirical compounds (khipu)"
- [intro] We have introduced the concept of empirical compound to describe the measurement of a tentative metabolite: "We have introduced the concept of empirical compound to describe the measurement of a tentative metabolite"
