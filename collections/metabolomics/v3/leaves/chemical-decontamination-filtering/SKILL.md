---
name: chemical-decontamination-filtering
description: Use when extracting component areas from a spreadOut()-processed GC-MS
  hit table and you must purge chemicals that lack strong cross-sample library matches,
  PubChem records, or canonical SMILES before downstream standardization.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on spectra
  published in PubChem and do not yet apply to these acquisition modes
- You need to retain novel or unannotated chemicals that legitimately lack PubChem
  entries or canonical SMILES — decontamination would silently drop them
- The vendor export has been renamed from the strict required column names (Component.RT,
  Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor), which spreadOut()
  and mzExacto() require unchanged
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: uafR
  role: R package providing spreadOut() to prepare the peak table and mzExacto(decontaminate
    = TRUE) to extract and decontaminate chemical areas across GC-MS samples
  repo: github.com/castratton/uafR
- name: R
  role: execution environment for running the uafR decontamination workflow
- name: PubChem
  role: public chemical database consulted during decontamination; chemicals without
    a PubChem entry are removed
- name: Agilent Unknowns Analysis
  role: upstream peak-deconvolution software whose exported hit table is the raw input
    prior to spreadOut() and decontamination
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:111410433a272fb3a8380c7701e426a12f6f171899dea8b9655386834b3f2b7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent Unknowns Analysis
  - PubChem
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-decontamination-filtering

## Summary

Applies uafR's decontaminate sub-argument to mzExacto() so that GC-MS compound extraction retains only chemicals with a strong cross-sample match, a PubChem entry, and a canonical SMILES, dropping ambiguous or unverifiable tentative identifications.

## When to use

Use when extracting component areas from a spreadOut()-processed GC-MS hit table and you must purge chemicals that lack strong cross-sample library matches, PubChem records, or canonical SMILES before downstream standardization.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on spectra published in PubChem and do not yet apply to these acquisition modes
- You need to retain novel or unannotated chemicals that legitimately lack PubChem entries or canonical SMILES — decontamination would silently drop them
- The vendor export has been renamed from the strict required column names (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor), which spreadOut() and mzExacto() require unchanged

## Inputs

- spreadOut()-processed GC-MS hit table (data frame with Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor columns, no duplicate peak keys)
- character vector of query chemical names (e.g. known internal standards or Match.Factor > 80 subset)
- optional decontaminate = TRUE flag

## Outputs

- data frame of extracted compounds with optimal retention time, exact mass, best match factor, and aggregated component area per chemical per sample (0 where the chemical is absent)
- decontaminated chemical list (chemicals failing cross-sample match, PubChem lookup, or canonical SMILES generation removed)

## How to apply

First prepare the raw Agilent Unknowns Analysis export (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) with spreadOut() so retention-time windows and top-2 m/z matches are established. Then call mzExacto() with decontaminate = TRUE on the query chemical list: the function orders query chemicals by exact mass, matches top-2 published m/z values and match factors per sample, aggregates Component.Area per chemical per sample (substituting 0 when a chemical is absent), and applies the decontamination filter. Rationale: tentative GC-MS identifications from library search are noisy, so chemicals without a strong match across samples, without a PubChem entry, or without a canonical SMILES cannot be reliably verified or compared via Tanimoto structural similarity (strong > 0.95, moderate > 0.85). The filter runs before standardifyIt()-style standardization so downstream abundance comparisons operate only on defensible identities. Validate by running on a set of known internal standards and checking that the expected number of extracted-compound rows is returned with non-decreasing area vs. injected volume.

## Related tools

- **uafR** (R package providing spreadOut() to prepare the peak table and mzExacto(decontaminate = TRUE) to extract and decontaminate chemical areas across GC-MS samples) — github.com/castratton/uafR
- **R** (execution environment for running the uafR decontamination workflow)
- **PubChem** (public chemical database consulted during decontamination; chemicals without a PubChem entry are removed)
- **Agilent Unknowns Analysis** (upstream peak-deconvolution software whose exported hit table is the raw input prior to spreadOut() and decontamination)

## Examples

```
selected = mzExacto(spread, query_compounds, decontaminate = TRUE)
```

## Evaluation signals

- Running the decontaminated extraction on the four known internal standards returns exactly four extracted-compound rows
- Output carries the aggregated per-sample Component.Area values, with 0 substituted wherever a query chemical is absent from a sample
- Every retained chemical has a non-null PubChem entry and a canonical SMILES; chemicals failing either are absent from the result
- On purified internal standards, extracted peak areas correlate with injected volume (R2 range 0.8273–0.9998 in the paper's validation)
- Input contract check: all six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) present and duplicate peak keys rejected

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or on LC-MS data
- MS2/MS3 and ion mobility + MS2 support is deferred until such spectra are published for most chemicals in PubChem's database
- The decontamination step depends on cross-sample match strength and public-database coverage, so chemicals absent from PubChem are removed even if present in the run

## Evidence

- [intro] Decontaminate filter removes unverifiable chemicals: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [methods] decontaminate sub-argument usage in mzExacto: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [readme] mzExacto output contract: [evidence span withheld — anchor status: not_found]
- [intro] Tanimoto structural match strength thresholds: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [results] Standards validation accuracy: "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998"
