---
name: chemical-mixture-configuration
description: Use when you need to produce synthetic chemical datasets (m/z, RT, intensity,
  formula) for ViMMS 2.0 simulations, either sampled from a database like HMDB or
  over a uniform m/z range.
when_to_use_negative:
- When the goal is to simulate from empirical data rather than synthetic sampling
  — extract chemicals from an existing mzML file instead of generating them via ChemicalMixtureCreator
- When the task is fragmentation-strategy simulation or evaluation (controller classes,
  Environment, write_mzML) rather than generating the input chemical list
- When no pickled HMDB compound database or sampler configuration is available and
  chemical properties cannot be defined
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Framework whose vimms.Chemicals.ChemicalMixtureCreator class and sampler classes
    generate the synthetic chemical dataset
  repo: https://github.com/glasgowcompbio/vimms
- name: HMDB
  role: External compound database sampled by DatabaseFormulaSampler to provide formulae
    and m/z values
- name: Python
  role: Language in which the samplers and ChemicalMixtureCreator are instantiated
    and called
- name: numpy
  role: Common dependency used by ViMMS for the sampling logic underlying chemical
    generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:a64dc8d8cd9a3293d984dd544f6f268388720ed82b229f78030f5d36656defb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - HMDB
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chemical-mixture-configuration

## Summary

Configure pluggable formula, RT, intensity and chromatogram samplers and feed them into vimms.Chemicals.ChemicalMixtureCreator to synthetically generate the KnownChemical list that seeds ViMMS 2.0's simulated LC-MS/MS runs.

## When to use

Use when you need to produce synthetic chemical datasets (m/z, RT, intensity, formula) for ViMMS 2.0 simulations, either sampled from a database like HMDB or over a uniform m/z range.

## When NOT to use

- When the goal is to simulate from empirical data rather than synthetic sampling — extract chemicals from an existing mzML file instead of generating them via ChemicalMixtureCreator
- When the task is fragmentation-strategy simulation or evaluation (controller classes, Environment, write_mzML) rather than generating the input chemical list
- When no pickled HMDB compound database or sampler configuration is available and chemical properties cannot be defined

## Inputs

- Pickled HMDB compound database (hmdb_compounds.p) for database-backed formula sampling
- Sampler configuration parameters (min_mz/max_mz range, number of chemicals, MS levels, RT/intensity/chromatogram sampler settings)
- Optionally an existing mzML file, since chemicals can also be extracted from existing mzML files

## Outputs

- List of KnownChemical objects produced by ChemicalMixtureCreator.sample()
- Serialised machine-readable file with one record per chemical containing m/z, RT, intensity, and formula fields

## How to apply

First instantiate a formula sampler: DatabaseFormulaSampler reading a pickled HMDB compound database (hmdb_compounds.p) with m/z constrained to e.g. 100–1000, or UniformMZFormulaSampler over a user-defined range such as min_mz=100, max_mz=500. Next configure RT, intensity, and chromatogram samplers to control each chemical's retention-time and chromatographic behaviour. Pass all samplers into vimms.Chemicals.ChemicalMixtureCreator and call its sample() method requesting the desired number of chemicals (e.g. 100 chemicals up to 2 MS levels). Serialise the resulting KnownChemical objects to a machine-readable file capturing each chemical's m/z, RT, intensity, and formula, then validate that the file parses with one populated record per sampled chemical.

## Related tools

- **ViMMS 2.0** (Framework whose vimms.Chemicals.ChemicalMixtureCreator class and sampler classes generate the synthetic chemical dataset) — https://github.com/glasgowcompbio/vimms
- **HMDB** (External compound database sampled by DatabaseFormulaSampler to provide formulae and m/z values)
- **Python** (Language in which the samplers and ChemicalMixtureCreator are instantiated and called)
- **numpy** (Common dependency used by ViMMS for the sampling logic underlying chemical generation)

## Examples

```
from vimms.Chemicals import ChemicalMixtureCreator; from vimms.Common import UniformMZFormulaSampler; chems = ChemicalMixtureCreator(UniformMZFormulaSampler(min_mz=100, max_mz=500)).sample(100, 2)
```

## Evaluation signals

- The serialised chemical file parses and contains exactly one record per requested chemical (e.g. 100 records)
- Every record has m/z, RT, intensity, and formula fields populated
- All sampled m/z values fall within the configured range (e.g. 100–1000 for DatabaseFormulaSampler from HMDB, or min_mz=100, max_mz=500 for UniformMZFormulaSampler)
- Each chemical carries the requested number of MS levels (e.g. up to 2 MS levels)
- Formulas for database-backed sampling are traceable to entries in the HMDB compound database

## Limitations

- ViMMS chemicals can be generated synthetically or extracted from existing mzML files; this skill covers only the synthetic sampling path, and the min_intensity threshold used when extracting from mzML strongly affects simulation speed (threshold 1.75E5 for fast runs, 1E4 for more accuracy, 0 includes all noise ROIs)
- Chemical generation must precede any fragmentation simulation; it does not by itself evaluate or benchmark acquisition strategies
- No changelog or explicit software version number is stated for ViMMS 2.0 in the paper, so sampler behaviour should be checked against the repository documentation

## Evidence

- [methods] Chemicals are generated by sampling: a DatabaseFormulaSampler is configured to sample formulae from the HMDB database with m/z values between 100 and 1000: "They can be sampled from databases such as HMDB"
- [methods] ChemicalMixtureCreator is the entry point for chemical generation: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [abstract] Chemical sources include synthetic sampling or extraction from mzML: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
- [results] The HMDB pickled compound file is a concrete dataset input: "url = 'https://github.com/glasgowcompbio/vimms-data/raw/main/hmdb_compounds.p'"
- [abstract] Modularity of ViMMS 2.0 allows samplers to be swapped: "The improved modularity in ViMMS 2.0 allows many aspects
of the framework to be swapped out with alternative implementations"
