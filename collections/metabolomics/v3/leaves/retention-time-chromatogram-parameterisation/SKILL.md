---
name: retention-time-chromatogram-parameterisation
description: Use when building synthetic chemical datasets for ViMMS 2.0 simulations
  and the generated chemicals need controlled retention-time, intensity and chromatogram
  shapes (e.g. Gaussian peaks) alongside formula-derived m/z values, rather than chemicals
  extracted from existing mzML files.
when_to_use_negative:
- When chemical objects should be extracted from existing mzML files instead of sampled
  synthetically — ViMMS 2.0 supports both routes, and this skill covers only the synthetic/sampled
  route
- When the goal is implementing or tuning a fragmentation controller (e.g. Top-N DDA,
  SmartROIController, WeightedDEWController) rather than producing the input chemical
  mixture
- When ground-truth features from real data (e.g. multibeers_2_aligned.csv) are needed
  rather than virtual chemicals
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: Provides vimms.Chemicals.ChemicalMixtureCreator and the pluggable formula/RT/intensity/chromatogram
    sampler classes used to generate the chemical list
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the samplers are instantiated, configured and invoked
- name: numpy
  role: Common dependency of ViMMS used by the sampling/numerical machinery behind
    RT and intensity draws
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:e6cb8bac54ea8eb18f7f92dcf431da16d52f17f50050ead0f32e32312da6c4e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# retention-time-chromatogram-parameterisation

## Summary

Parameterises the retention-time, intensity and chromatographic behaviour of synthetic chemicals in ViMMS 2.0 by configuring RT, intensity and chromatogram samplers that are passed alongside a formula sampler into ChemicalMixtureCreator, yielding KnownChemical objects ready for in-silico MS/MS simulation.

## When to use

Use when building synthetic chemical datasets for ViMMS 2.0 simulations and the generated chemicals need controlled retention-time, intensity and chromatogram shapes (e.g. Gaussian peaks) alongside formula-derived m/z values, rather than chemicals extracted from existing mzML files.

## When NOT to use

- When chemical objects should be extracted from existing mzML files instead of sampled synthetically — ViMMS 2.0 supports both routes, and this skill covers only the synthetic/sampled route
- When the goal is implementing or tuning a fragmentation controller (e.g. Top-N DDA, SmartROIController, WeightedDEWController) rather than producing the input chemical mixture
- When ground-truth features from real data (e.g. multibeers_2_aligned.csv) are needed rather than virtual chemicals

## Inputs

- Pickled HMDB compound database (hmdb_compounds.p) for database-backed formula sampling
- User-defined m/z range for uniform sampling (e.g. min_mz=100, max_mz=500)
- Sampler configuration: formula sampler, RT sampler, intensity sampler, chromatogram sampler
- Requested chemical count and number of MS levels (e.g. 100 chemicals, 2 MS levels)

## Outputs

- List of KnownChemical objects with populated m/z, RT, intensity and formula fields
- Machine-readable serialised chemical list consumed by the ViMMS 2.0 simulator

## How to apply

First choose a formula sampler that determines m/z — either a DatabaseFormulaSampler reading a pickled HMDB compound database (hmdb_compounds.p) with m/z between 100 and 1000, or a UniformMZFormulaSampler over a user-defined range such as min_mz=100, max_mz=500. Then configure RT, intensity and chromatogram samplers to control the retention-time position, peak height and chromatographic shape of each chemical, since the improved modularity of ViMMS 2.0 allows many aspects of the framework to be swapped out with alternative implementations. Pass all samplers into vimms.Chemicals.ChemicalMixtureCreator and call its sample() method with the requested size (e.g. 100 chemicals up to 2 MS levels). The decision point is the source of formulae (database-backed vs uniform m/z); the resulting KnownChemical objects should be serialised capturing each chemical's m/z, RT, intensity and formula.

## Related tools

- **ViMMS 2.0** (Provides vimms.Chemicals.ChemicalMixtureCreator and the pluggable formula/RT/intensity/chromatogram sampler classes used to generate the chemical list) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the samplers are instantiated, configured and invoked)
- **numpy** (Common dependency of ViMMS used by the sampling/numerical machinery behind RT and intensity draws)

## Examples

```
from vimms.Chemicals import ChemicalMixtureCreator; cmc = ChemicalMixtureCreator(formula_sampler, rt_sampler, intensity_sampler, chromatogram_sampler); chemicals = cmc.sample(100, 2)
```

## Evaluation signals

- The serialised output file parses and contains exactly one record per sampled chemical (e.g. 100 records for a 100-chemical mixture)
- Every KnownChemical record has m/z, RT, intensity and formula fields populated (non-null, m/z within the configured range such as 100–1000 for HMDB-backed sampling)
- Retention times and chromatogram shapes differ across chemicals according to the configured RT/chromatogram samplers rather than being identical
- Feeding the chemical list into the ViMMS simulator via Environment produces a writable mzML containing scans for the sampled chemicals

## Limitations

- No validation of ViMMS 2.0 is presented in the paper itself; utility validation (including bridging simulated controllers to actual mass spectrometers) is deferred to Davies et al., 2021
- Database-backed sampling is restricted to formulae from HMDB with m/z between 100 and 1000; other ranges require alternatives like UniformMZFormulaSampler
- Downstream simulation cost depends on feature thresholds: a min_intensity threshold of 1.75E5 was set high to reduce processing time, while lowering to 1E4 (or 0 to include noise ROIs) makes runs much slower with higher memory usage

## Evidence

- [methods] use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [methods] They can be sampled from databases such as HMDB: "They can be sampled from databases such as HMDB"
- [abstract] generate chemical objects (produced synthetically or extracted from existing mzML files): "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
- [abstract] The improved modularity in ViMMS 2.0 allows many aspects of the framework to be swapped out with alternative implementations.: "The improved modularity in ViMMS 2.0 allows many aspects
of the framework to be swapped out with alternative implementations"
- [results] url = 'https://github.com/glasgowcompbio/vimms-data/raw/main/hmdb_compounds.p': "url = 'https://github.com/glasgowcompbio/vimms-data/raw/main/hmdb_compounds.p'"
