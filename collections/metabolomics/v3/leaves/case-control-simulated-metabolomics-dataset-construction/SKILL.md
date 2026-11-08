---
name: case-control-simulated-metabolomics-dataset-construction
description: Use when a simulation experiment requires multiple chemical datasets
  (e.g. 'control' vs 'case' samples) derived from one shared base chemical list, with
  configurable per-chemical dropout rates or cross-sample intensity variation.
when_to_use_negative:
- When a single simulated chemical mixture for one sample suffices — use ChemicalMixtureCreator
  directly rather than the multi-sample dropout machinery
- When the input is already a real acquired dataset (e.g. mzML files) to replay rather
  than a synthetic case-control design
- When chemicals must appear in every sample without dropout — a missing_probability
  of dropout being unwanted makes MultipleMixtureCreator's per-sample dropping inappropriate
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ViMMS 2.0
  role: Provides MultipleMixtureCreator (and ChemicalMixtureCreator for the base list)
    to generate one chemical set per sample with configured dropout and intensity-change
    probabilities
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the chemical-list generation code is written and invoked
    (requires Python 3.0 or higher)
- name: pandas
  role: Used to write the resulting per-sample chemical sets to a table with a sample
    identifier column
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:c5994c815da0ee0cb7b761c105037d3e2e1c5dc52fc47c84138bd3f475c1ffb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - pandas
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Case-control simulated metabolomics dataset construction

## Summary

Constructs multi-sample, case-control style simulated metabolomics chemical lists in ViMMS 2.0 by sharing a master base-chemical list across samples and applying per-group dropout (missing_probability) and intensity-change (changing_probability) settings via MultipleMixtureCreator.

## When to use

Use when a simulation experiment requires multiple chemical datasets (e.g. 'control' vs 'case' samples) derived from one shared base chemical list, with configurable per-chemical dropout rates or cross-sample intensity variation.

## When NOT to use

- When a single simulated chemical mixture for one sample suffices — use ChemicalMixtureCreator directly rather than the multi-sample dropout machinery
- When the input is already a real acquired dataset (e.g. mzML files) to replay rather than a synthetic case-control design
- When chemicals must appear in every sample without dropout — a missing_probability of dropout being unwanted makes MultipleMixtureCreator's per-sample dropping inappropriate

## Inputs

- Master list of base chemicals shared across all samples (synthetic chemical objects or compounds sampled from a database such as HMDB, e.g. hmdb_compounds.p)
- Sample group descriptions (group_list), e.g. ['control', 'case']
- Per-group settings dictionary (group_dict) with missing_probability and changing_probability

## Outputs

- One chemical set (chemical mixture) per sample, with chemicals dropped or intensity-varied per the configured rates
- A table of per-sample chemical sets with a sample identifier column (e.g. pandas DataFrame/CSV)

## How to apply

First load or generate the master list of base chemicals shared across all samples (e.g. via ChemicalMixtureCreator or sampling compounds from HMDB). Then instantiate ViMMS 2.0's MultipleMixtureCreator, passing the master chemical list, the sample group descriptions (group_list, e.g. ['control', 'case']), and per-group settings in group_dict such as missing_probability (the proportion of extracts in which each chemical is dropped) and changing_probability (how intensities vary across the case-control experiment). Call generate_chemical_lists() to produce one chemical set per sample, where each chemical is dropped from a sample according to the configured missing proportion. Finally write the resulting per-sample chemical sets to a table with a sample identifier column (e.g. using pandas). Validate that the output contains one chemical set per sample group and that chemical counts per sample reflect the configured dropout rate.

## Related tools

- **ViMMS 2.0** (Provides MultipleMixtureCreator (and ChemicalMixtureCreator for the base list) to generate one chemical set per sample with configured dropout and intensity-change probabilities) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the chemical-list generation code is written and invoked (requires Python 3.0 or higher))
- **pandas** (Used to write the resulting per-sample chemical sets to a table with a sample identifier column)

## Examples

```
from vimms.Chemicals import MultipleMixtureCreator; mmc = MultipleMixtureCreator(chem_list, group_list=['control','case'], group_dict={...}); chem_lists = mmc.generate_chemical_lists()
```

## Evaluation signals

- Output contains exactly one chemical set per sample group defined in group_list (e.g. one set per 'control' and per 'case' sample)
- Number of chemicals present in each sample reflects the configured missing_probability dropout rate (chemicals absent from a sample appear with the expected frequency across extracts)
- All per-sample sets derive from the same master base-chemical list (no chemicals outside the supplied list)
- Chemicals flagged by changing_probability show the expected intensity variation across the case-control groups
- Resulting table has a sample identifier column linking each chemical set to its source sample

## Limitations

- The article does not report validation of ViMMS 2.0 itself — utility validation via bridging on actual mass spectrometers is deferred to a separate publication (Davies et al., 2021)
- No evaluation results, benchmarks, or quantitative performance data are reported for this simulation workflow in the paper; only a statement of hope for future use
- No explicit software version number is stated for ViMMS 2.0 in the paper

## Evidence

- [full_text] users supply a list of base chemicals shared across samples and can specify the dropout rate (the proportion of extracts in which each chemical appears): [evidence span withheld — anchor status: not_found]
- [full_text] MultipleMixtureCreator invocation with group_list and group_dict: [evidence span withheld — anchor status: not_found]
- [methods] use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [methods] They can be sampled from databases such as HMDB: "They can be sampled from databases such as HMDB"
