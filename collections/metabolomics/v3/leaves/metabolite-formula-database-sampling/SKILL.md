---
name: metabolite-formula-database-sampling
description: Use when building synthetic chemical inputs for ViMMS 2.0 simulations
  and you need formulae sampled from a database like HMDB (e.g. m/z 100–1000) or drawn
  from a uniform m/z range rather than extracted from mzML.
when_to_use_negative:
- You want chemicals derived from real experimental data — ViMMS 2.0 can alternatively
  extract chemical objects from existing mzML files (e.g. Beer_multibeers_1_T10_POS.mzML)
  instead of sampling synthetically.
- You need the full ViMMS simulation output (mzML scans) — this skill only covers
  the chemical-generation stage, not running controllers in an Environment or writing
  mzML.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: ViMMS 2.0
  role: Framework providing ChemicalMixtureCreator and the sampler classes (DatabaseFormulaSampler,
    UniformMZFormulaSampler) used to generate chemical datasets for simulation
  repo: https://github.com/glasgowcompbio/vimms
- name: HMDB
  role: External metabolite database from which formulae (and m/z values between 100
    and 1000) are sampled
- name: Poetry
  role: Virtual environment and dependency management for installing ViMMS 2.0 (`poetry
    install`)
- name: Python
  role: Runtime language for instantiating samplers and calling ChemicalMixtureCreator.sample()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:67434ab40d8e77120b49852e5403c78449dcb52337ae80f6184c1676956fa497
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - HMDB
  - Poetry
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# metabolite-formula-database-sampling

## Summary

Generate simulated metabolite lists in ViMMS 2.0 by plugging database-backed or uniform samplers (formulae, RT, intensity, chromatograms) into ChemicalMixtureCreator, producing KnownChemical objects with m/z, RT, intensity and formula fields.

## When to use

Use when building synthetic chemical inputs for ViMMS 2.0 simulations and you need formulae sampled from a database like HMDB (e.g. m/z 100–1000) or drawn from a uniform m/z range rather than extracted from mzML.

## When NOT to use

- You want chemicals derived from real experimental data — ViMMS 2.0 can alternatively extract chemical objects from existing mzML files (e.g. Beer_multibeers_1_T10_POS.mzML) instead of sampling synthetically.
- You need the full ViMMS simulation output (mzML scans) — this skill only covers the chemical-generation stage, not running controllers in an Environment or writing mzML.

## Inputs

- Pickled HMDB compound database (hmdb_compounds.p from glasgowcompbio/vimms-data)
- Sampler configuration parameters: m/z range (e.g. min_mz=100, max_mz=500 or 100–1000 for HMDB), chemical count (e.g. 100), max MS levels (e.g. 2)
- RT, intensity and chromatogram sampler objects

## Outputs

- List of KnownChemical objects
- Serialised machine-readable chemical file with one record per chemical containing m/z, RT, intensity and formula fields

## How to apply

Install ViMMS 2.0 in a Poetry virtual environment (`poetry install`). Choose a formula sampler: DatabaseFormulaSampler reading a pickled HMDB compound database (hmdb_compounds.p from the vimms-data repo) restricted to an m/z range such as 100–1000, or UniformMZFormulaSampler over a user-defined range (e.g. min_mz=100, max_mz=500). Configure RT, intensity and chromatogram samplers to control each chemical's retention-time and chromatographic behaviour. Pass the samplers into vimms.Chemicals.ChemicalMixtureCreator and call sample() to generate the requested number of chemicals (e.g. 100 chemicals up to 2 MS levels). Serialise the resulting KnownChemical objects to a machine-readable file capturing m/z, RT, intensity and formula. This pluggable-sampler design reflects ViMMS 2.0's modularity, where many framework aspects can be swapped with alternative implementations.

## Related tools

- **ViMMS 2.0** (Framework providing ChemicalMixtureCreator and the sampler classes (DatabaseFormulaSampler, UniformMZFormulaSampler) used to generate chemical datasets for simulation) — https://github.com/glasgowcompbio/vimms
- **HMDB** (External metabolite database from which formulae (and m/z values between 100 and 1000) are sampled)
- **Poetry** (Virtual environment and dependency management for installing ViMMS 2.0 (`poetry install`))
- **Python** (Runtime language for instantiating samplers and calling ChemicalMixtureCreator.sample())

## Examples

```
from vimms.Chemicals import ChemicalMixtureCreator; from vimms.Database import DatabaseFormulaSampler; dm = DatabaseFormulaSampler(min_mz=100, max_mz=1000); c = ChemicalMixtureCreator(dm); chemicals = c.sample(100, 2)
```

## Evaluation signals

- The serialised output file parses and contains exactly one record per sampled chemical (e.g. 100 records for a 100-chemical request).
- Each record has all four fields populated: m/z, RT, intensity, and formula.
- When DatabaseFormulaSampler is used, all formulae originate from the HMDB compound database and sampled m/z values fall within the configured range (e.g. 100–1000).
- When UniformMZFormulaSampler is used, sampled m/z values lie within the user-defined range (e.g. 100–500).
- Chemicals carry at most the requested number of MS levels (e.g. 2).

## Limitations

- Database-backed sampling requires the pickled HMDB compound file (hmdb_compounds.p), which must be fetched from the vimms-data repository beforehand.
- This paper provides no validation of the simulation output itself; utility validation via bridging to real instruments is reported separately in Davies et al. (2021).
- Chemicals produced this way are synthetic; for realistic peak-shape behaviour from empirical data, extracting chemicals from existing mzML files may be preferable.

## Evidence

- [results] DatabaseFormulaSampler reading a pickled HMDB compound database for database-backed sampling, or UniformMZFormulaSampler over a user-defined m/z range: "They can be sampled from databases such as HMDB"
- [methods] Chemicals are generated by sampling formulae from HMDB with m/z values between 100 and 1000: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [results] hmdb_compounds.p is the pickled HMDB compound database available from vimms-data: "url = 'https://github.com/glasgowcompbio/vimms-data/raw/main/hmdb_compounds.p'"
- [abstract] Modularity allows samplers to be swapped with alternative implementations: "The improved modularity in ViMMS 2.0 allows many aspects of the framework to be swapped out with alternative implementations"
- [abstract] Chemicals can be produced synthetically or extracted from existing mzML files: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
