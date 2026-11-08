---
name: chemical-dropout-probability-sampling
description: Use when you need synthetic chemical datasets for multiple samples (e.g.
  control vs case groups) where each chemical is present in only a subset of samples
  according to a configurable dropout rate.
when_to_use_negative:
- You need a single shared chemical list with no case-control/dropout design — use
  ChemicalMixtureCreator for a single chemical dataset instead.
- You want to simulate the MS/MS acquisition itself (Top-N DDA, SmartROI, WeightedDEW
  fragmentation schedules) — this skill only produces the per-sample input chemical
  sets, not scans.
- You want to extract chemical objects empirically from existing mzML files rather
  than generate synthetic per-sample mixtures with dropout.
edam_topics:
- http://edamontology.org/topic_2250
tools:
- name: ViMMS 2.0
  role: Provides MultipleMixtureCreator and ChemicalMixtureCreator to generate per-sample
    chemical lists with dropout and intensity-change probabilities
  repo: https://github.com/glasgowcompbio/vimms
- name: pandas
  role: Dependency used to assemble and write the per-sample chemical sets as a table
    with a sample identifier column
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:ac2f410af2cd5540271c88d64ae65dc4db9465b5e1566ed441560f3b7dae1638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chemical-dropout-probability-sampling

## Summary

Generates per-sample chemical lists for case-control style experiments by applying probabilistic dropout (missing_probability) and intensity-changing (changing_probability) settings to a shared master chemical list, using ViMMS's MultipleMixtureCreator. This lets researchers simulate the biological variability expected across multiple samples before any MS/MS acquisition simulation.

## When to use

Use when you need synthetic chemical datasets for multiple samples (e.g. control vs case groups) where each chemical is present in only a subset of samples according to a configurable dropout rate.

## When NOT to use

- You need a single shared chemical list with no case-control/dropout design — use ChemicalMixtureCreator for a single chemical dataset instead.
- You want to simulate the MS/MS acquisition itself (Top-N DDA, SmartROI, WeightedDEW fragmentation schedules) — this skill only produces the per-sample input chemical sets, not scans.
- You want to extract chemical objects empirically from existing mzML files rather than generate synthetic per-sample mixtures with dropout.

## Inputs

- Master list of base Chemical objects shared across all samples (e.g. sampled from HMDB via hmdb_compounds.p or created with ChemicalMixtureCreator)
- Sample group descriptions (group_list, e.g. ['control', 'case'])
- Per-group settings in group_dict, including missing_probability (dropout proportion) and changing_probability (intensity variation probability)

## Outputs

- One chemical list per sample, each containing a subset of the master chemicals determined by the configured dropout rates
- A table of per-sample chemical sets with a sample identifier column (exportable with pandas)

## How to apply

Load the base chemical list (the master set of chemicals shared across all samples, e.g. sampled from a database such as HMDB or produced via ChemicalMixtureCreator). Pass the master list to ViMMS's MultipleMixtureCreator together with the sample group descriptions (group_list, e.g. ['control', 'case']) and a group_dict holding per-group settings such as missing_probability and changing_probability. Call generate_chemical_lists() to create one chemical set per sample, where each chemical is dropped from a sample according to the specified missing (dropout) proportion. Finally write the resulting per-sample chemical sets to a table with a sample identifier column. The per-group probabilities are the decision points: missing_probability controls chemical absence per sample, while changing_probability controls how chemical intensities vary across the case-control experiment.

## Related tools

- **ViMMS 2.0** (Provides MultipleMixtureCreator and ChemicalMixtureCreator to generate per-sample chemical lists with dropout and intensity-change probabilities) — https://github.com/glasgowcompbio/vimms
- **pandas** (Dependency used to assemble and write the per-sample chemical sets as a table with a sample identifier column)

## Examples

```
from vimms.Chemicals import MultipleMixtureCreator; mmc = MultipleMixtureCreator(chemical_list, group_list=['control','case'], group_dict=group_dict); chem_lists = mmc.generate_chemical_lists()
```

## Evaluation signals

- Output contains exactly one chemical set per sample named in group_list (e.g. one per 'control' and per 'case' sample).
- For each chemical, the proportion of samples in which it appears reflects the configured missing_probability (dropout rate).
- Every chemical in a per-sample set is drawn from the supplied master chemical list — no novel chemical entries appear.
- Chemical counts differ across sample groups when changing_probability / missing_probability settings differ between groups.
- The exported table includes a sample identifier column and non-empty chemical sets for samples with low dropout settings.

## Limitations

- Dropout is probabilistic, so per-sample chemical counts only reflect the configured missing_probability on average, not exactly.
- ViMMS 2.0's validation is deferred to a separate publication (Davies et al., 2021); no validation of the framework is presented in this paper.
- No explicit software version number is stated for ViMMS 2.0 in the paper; behaviour should be checked against the glasgowcompbio/vimms repository.
- This skill covers chemical-list generation only; downstream MS/MS simulation requires a separate fragmentation-strategy step.

## Evidence

- [methods] MultipleMixtureCreator generates per-sample lists with dropout: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [abstract] Chemical objects may be synthetic or from mzML: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
- [readme] ViMMS generates new LC-MS/MS data from empirical data or replay: [evidence span withheld — anchor status: external_doc]
- [methods] HMDB as chemical source: "They can be sampled from databases such as HMDB"
