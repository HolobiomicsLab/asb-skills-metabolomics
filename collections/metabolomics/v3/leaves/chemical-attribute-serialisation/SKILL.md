---
name: chemical-attribute-serialisation
description: Use when a ViMMS 2.0 workflow has just sampled a synthetic chemical mixture
  (e.g. via ChemicalMixtureCreator with HMDB-backed or uniform m/z formula samplers)
  and the sampled chemical attributes need to be persisted for simulation, replay
  or ground-truth comparison.
when_to_use_negative:
- When chemicals are extracted from existing mzML files (the empirical route) rather
  than generated synthetically — the article states chemicals are 'produced synthetically
  or extracted from existing mzML files', and the empirical path has its own feature-extraction
  tooling.
- When the downstream step needs raw instrument scans (mzML output from an Environment
  run) rather than a ground-truth chemical list.
- When chemical identities are unknown and must remain simulated anonymous m/z features
  — serialising KnownChemical assumes formula-level ground truth.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: Provides vimms.Chemicals.ChemicalMixtureCreator, the sampler classes (DatabaseFormulaSampler,
    UniformMZFormulaSampler) and the KnownChemical objects whose attributes are serialised
  repo: https://github.com/glasgowcompbio/vimms
- name: vimms-data
  role: Hosts the pickled HMDB compound database (hmdb_compounds.p) used by DatabaseFormulaSampler
    as the source of formulae for sampling
  repo: https://github.com/glasgowcompbio/vimms-data
- name: numpy
  role: Common dependency of ViMMS used for sampling chemical attributes (m/z, RT,
    intensity)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:5131266ae8c1fbea8472a4e6d892a4fcd6e41e9a37f8c0b9337d385a0d7e100b
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
  - vimms-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chemical-attribute-serialisation

## Summary

Serialises ViMMS 2.0-simulated chemical objects (KnownChemical) to a machine-readable file capturing each chemical's m/z, retention time, intensity and formula after sampling via ChemicalMixtureCreator, so downstream simulation runs have a ground-truth chemical list.

## When to use

Use when a ViMMS 2.0 workflow has just sampled a synthetic chemical mixture (e.g. via ChemicalMixtureCreator with HMDB-backed or uniform m/z formula samplers) and the sampled chemical attributes need to be persisted for simulation, replay or ground-truth comparison.

## When NOT to use

- When chemicals are extracted from existing mzML files (the empirical route) rather than generated synthetically — the article states chemicals are 'produced synthetically or extracted from existing mzML files', and the empirical path has its own feature-extraction tooling.
- When the downstream step needs raw instrument scans (mzML output from an Environment run) rather than a ground-truth chemical list.
- When chemical identities are unknown and must remain simulated anonymous m/z features — serialising KnownChemical assumes formula-level ground truth.

## Inputs

- Pickled HMDB compound database (hmdb_compounds.p) for DatabaseFormulaSampler
- Sampler configuration: DatabaseFormulaSampler or UniformMZFormulaSampler (min_mz, max_mz), RT sampler, intensity sampler, chromatogram sampler
- Requested chemical-mixture parameters (e.g. n_chems=100, n_ms2=1)

## Outputs

- Machine-readable chemical list file with one record per sampled KnownChemical, each containing m/z, RT, intensity and formula fields

## How to apply

First choose a formula sampler: DatabaseFormulaSampler reading the pickled HMDB compound database (hmdb_compounds.p from the vimms-data repository), optionally restricting m/z to a range such as 100–1000, or UniformMZFormulaSampler over a user-defined range such as min_mz=100, max_mz=500. Configure complementary RT, intensity and chromatogram samplers to control each chemical's retention-time and chromatographic behaviour. Pass all samplers into vimms.Chemicals.ChemicalMixtureCreator and call its sample() method at the requested size (e.g. 100 chemicals, up to 2 MS levels). Finally serialise the resulting KnownChemical objects to a machine-readable file so each record carries m/z, RT, intensity and formula; this modularity exists because ViMMS 2.0 allows aspects of the framework to be swapped out with alternative implementations.

## Related tools

- **ViMMS 2.0** (Provides vimms.Chemicals.ChemicalMixtureCreator, the sampler classes (DatabaseFormulaSampler, UniformMZFormulaSampler) and the KnownChemical objects whose attributes are serialised) — https://github.com/glasgowcompbio/vimms
- **vimms-data** (Hosts the pickled HMDB compound database (hmdb_compounds.p) used by DatabaseFormulaSampler as the source of formulae for sampling) — https://github.com/glasgowcompbio/vimms-data
- **numpy** (Common dependency of ViMMS used for sampling chemical attributes (m/z, RT, intensity))

## Examples

```
from vimms.Chemicals import ChemicalMixtureCreator, DatabaseFormulaSampler; formula_sampler = DatabaseFormulaSampler(hmdb_compounds, min_mz=100, max_mz=1000); chemicals = ChemicalMixtureCreator(formula_sampler, rt_sampler, intensity_sampler).sample(n_chems=100)
```

## Evaluation signals

- The output file parses without errors using standard Python tooling (numpy/scipy/pandas stack).
- The file contains exactly one record per sampled chemical (e.g. 100 records for a 100-chemical sample).
- Every record has m/z, RT, intensity and formula fields populated (non-null).
- m/z values fall inside the configured sampler range (e.g. 100–1000 for the HMDB-backed sampler, 100–500 for UniformMZFormulaSampler with min_mz=100, max_mz=500).
- Formulae correspond to entries sampled from the HMDB compound database when DatabaseFormulaSampler was used.

## Limitations

- The ViMMS 2.0 paper itself presents no validation results or quantitative performance data; utility validation via bridging to real instruments is deferred to Davies et al. (2021).
- No explicit software version number is stated for ViMMS 2.0, so reproducibility of sampler behaviour across releases is not guaranteed.
- Serialisation only captures the four core attributes (m/z, RT, intensity, formula); richer chromatographic behaviour is defined by the chromatogram samplers and may not be fully represented in the serialised file.

## Evidence

- [methods] ChemicalMixtureCreator produces chemical datasets: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [methods] Formulae sampled from HMDB: "They can be sampled from databases such as HMDB"
- [results] hmdb_compounds.p dataset: "url = 'https://github.com/glasgowcompbio/vimms-data/raw/main/hmdb_compounds.p'"
- [abstract] Modularity allows sampler swap-out: "The improved modularity in ViMMS 2.0 allows many aspects
of the framework to be swapped out with alternative implementations"
- [abstract] Chemical sources: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
