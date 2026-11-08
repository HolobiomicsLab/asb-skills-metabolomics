---
name: empirical-compound-grouping
description: Use when a pcpfm/Asari feature table must be pre-annotated into empirical
  compounds (grouping adducts and isotopologues) before MS1 or MS2 annotation of an
  LC-MS metabolomics experiment.
when_to_use_negative:
- The input is raw .mzML/.raw data that has not yet been preprocessed by Asari into
  a feature table — run Asari preprocessing first.
- The data are GC-MS or other non-LC-MS types, which the pipeline does not yet support
  ('We are working to add supports of GC and other data types').
- Features are already grouped into empirical compounds or a prior empCpd.json exists
  for the moniker — regrouping risks degenerate duplicate annotations.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Provides the build_empCpds operation that orchestrates grouping and writes
    empCpd.json to the annotations directory
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: khipu
  role: Engine that grids features into empirical compounds using configurable adduct/isotopologue
    lists and m/z/RT tolerances
- name: metDataModel
  role: Defines the JSON-compatible, chainable empirical compound (empCpd) data structure
    produced by grouping
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:cf45f06ccd40e3671710080bbbd3a78df3dc2ce155b9cf2a7476335cbb340e80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - MetDataModel
  - Python
  - khipu
  - metDataModel
  - pcpfm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# empirical-compound-grouping

## Summary

Groups features from an Asari feature table into empirical compounds (empCpds) via the khipu package in pcpfm, co-clustering co-eluting adducts and isotopologues as a pre-annotation step before MS1/MS2 annotation.

## When to use

Use when a pcpfm/Asari feature table must be pre-annotated into empirical compounds (grouping adducts and isotopologues) before MS1 or MS2 annotation of an LC-MS metabolomics experiment.

## When NOT to use

- The input is raw .mzML/.raw data that has not yet been preprocessed by Asari into a feature table — run Asari preprocessing first.
- The data are GC-MS or other non-LC-MS types, which the pipeline does not yet support ('We are working to add supports of GC and other data types').
- Features are already grouped into empirical compounds or a prior empCpd.json exists for the moniker — regrouping risks degenerate duplicate annotations.

## Inputs

- Asari-derived feature table (full_feature_table.tsv from the asari_results export directory)
- pcpfm experiment directory with annotations/ output location
- Experiment ionization mode (drives the default adduct list)

## Outputs

- empCpd.json in the experiment's annotations directory, mapping each empirical compound to its grouped features with adduct/isotopologue assignments
- Ungrouped entries for singleton features without inferred adducts

## How to apply

Load the Asari-derived feature table (the full feature table from the experiment's asari_results export directory is recommended) and invoke the pcpfm build_empCpds operation, which calls khipu to grid features into empirical compounds using default adduct lists appropriate to the experiment's inferred ionization mode, charges up to z=3, and m+13C3 isotopologues. Apply the default tolerances for Orbitrap LC data: 5 ppm m/z tolerance (--khipu_mz_tolerance) and 2 second retention time tolerance (--khipu_rt_tolerance) so co-eluting isotopologues and adducts are grouped. The adduct and isotope sets are user-configurable if the defaults do not fit the chemistry. Save the result under a user-specified empCpd moniker as empCpd.json in the experiment's annotations directory, recording each compound's constituent features and their adduct/isotope assignments. This pre-annotation step also prevents degenerate features (e.g., the same metabolite's adduct variants) from being queried separately in later MS1 annotation.

## Related tools

- **pcpfm** (Provides the build_empCpds operation that orchestrates grouping and writes empCpd.json to the annotations directory) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **khipu** (Engine that grids features into empirical compounds using configurable adduct/isotopologue lists and m/z/RT tolerances)
- **metDataModel** (Defines the JSON-compatible, chainable empirical compound (empCpd) data structure produced by grouping) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm build_empCpds --empCpd my_empCpds --khipu_mz_tolerance 5 --khipu_rt_tolerance 2
```

## Evaluation signals

- empCpd.json exists in the experiment's annotations directory and parses as valid JSON.
- Each empirical compound contains its grouped constituent features with explicit adduct/isotopologue assignments.
- Singleton features without inferred adducts are retained as ungrouped entries rather than dropped.
- Grouped features are consistent with the tolerances used: co-eluting within the RT tolerance (default 2 s) and within 5 ppm m/z.
- Downstream MS1 annotation queries each metabolite once per empCpd rather than per degenerate feature.

## Limitations

- If the experiment does not resolve isomers, the empirical compound may represent any or a mixture of isomers, so empCpd identity is tentative, not a compound ID.
- The adduct and isotopologue lists are defaults (charges up to z=3, m+13C3); unusual chemistry requires user reconfiguration.
- Defaults are tuned for Orbitrap LC data (5 ppm / 2 s); other instrument types may need adjusted tolerances.

## Evidence

- [readme] empCpd is a computational unit for a tentative metabolite: [evidence span withheld — anchor status: external_doc]
- [readme] pre-annotation definition: [evidence span withheld — anchor status: external_doc]
- [readme] pipeline includes khipu pre-annotation step: "pre-annotation to group featues to empirical compounds (khipu)"
- [intro] empirical compound concept in article: "We have introduced the concept of empirical compound to describe the measurement of a tentative metabolite"
