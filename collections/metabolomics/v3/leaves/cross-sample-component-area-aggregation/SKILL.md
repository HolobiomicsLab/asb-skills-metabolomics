---
name: cross-sample-component-area-aggregation
description: Use when a GC-MS hit table exported from vendor peak-detection software
  (e.g. Agilent Unknowns Analysis) contains multiple occurrences of the same tentative
  compound across samples and per-compound component areas must be collected and standardized.
when_to_use_negative:
- The data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend
  on published fragmentation data and do not yet apply
- The data are from LC-MS or untested instrument types (Q-TOF, Orbitrap, TimsTOF HT,
  Astral) — uafR has not been validated in those contexts
- The hit table lacks the required six columns (Component.RT, Component.Area, Base.Peak.MZ,
  File.Name, Compound.Name, Match.Factor) and cannot be reshaped to vendor-export
  format
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: Provides spreadOut() to validate and spread the hit table, mzExacto() to aggregate
    candidate peak areas across samples, and standardifyIt() to standardize areas
    to standards
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment; the package is installed from GitHub via devtools::install_github('castratton/uafR')
    and run from R or RStudio
- name: Agilent Unknowns Analysis
  role: Upstream peak-detection/deconvolution software whose exported tentative identifications
    form the input hit table consumed by spreadOut() and mzExacto()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:1b3b7258650b56315ba794c620d27e7314189131a04ef35dfdc6f4f73c455de9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - Agilent Unknowns Analysis
  - LOTUS
  - R
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-sample-component-area-aggregation

## Summary

Aggregate component peak areas for tentative compound identifications across every sample in a GC-MS experiment using the uafR R package (spreadOut + mzExacto), replacing weeks-to-months of manual peak-area selection and integration with a reproducible R workflow.

## When to use

Use when a GC-MS hit table exported from vendor peak-detection software (e.g. Agilent Unknowns Analysis) contains multiple occurrences of the same tentative compound across samples and per-compound component areas must be collected and standardized.

## When NOT to use

- The data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published fragmentation data and do not yet apply
- The data are from LC-MS or untested instrument types (Q-TOF, Orbitrap, TimsTOF HT, Astral) — uafR has not been validated in those contexts
- The hit table lacks the required six columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and cannot be reshaped to vendor-export format

## Inputs

- Aggregated GC-MS hit table (.csv exported from vendor peak-detection software such as Agilent Unknowns Analysis) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Vector of target compound names (user-defined, Match.Factor-thresholded, or exactoThese()-selected from a categorate result)
- Optional known internal/external standard values for area standardization

## Outputs

- Per-compound table of aggregated component areas across all samples, with optimal retention time, exact mass, and best match factor
- Decontaminated compound set (chemicals lacking strong cross-sample match, PubChem entry, or canonical SMILES dropped)
- Optionally, a standardized-area table normalized to internal or external standards, suitable for downstream PCA, NMDS, ANOSIM, or machine learning

## How to apply

Load the raw vendor hit table into R with read.csv and run spreadOut() to 'spread out' the table, which validates the mandatory columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor and rejects duplicate peak keys. Define the target compound list either explicitly (e.g. c('Linalool','Methyl Salicylate','Limonene','alpha-Thujene')) or by filtering the raw output with a Match.Factor threshold (e.g. > 80 or > 70), optionally derived via exactoThese() on a categorate result subset by 'LOTUS'/'KEGG'. Run mzExacto(spread, query_compounds, decontaminate = TRUE) to aggregate every occurrence of each user-specified chemical across every GC-MS sample; decontaminate removes chemicals lacking a strong cross-sample match (Tanimoto > 0.95 is strong, > 0.85 moderate), missing from PubChem, or without a canonical SMILES. Optionally call standardifyIt() to standardize component areas relative to known internal or external standards. The rationale is that manual area selection for the same molecule across samples is subjective and error-prone, whereas uafR stores critical information (retention time, exact mass, best match factor) for every tentative molecule while collecting areas reproducibly.

## Related tools

- **uafR** (Provides spreadOut() to validate and spread the hit table, mzExacto() to aggregate candidate peak areas across samples, and standardifyIt() to standardize areas to standards) — https://github.com/castratton/uafR
- **R** (Runtime environment; the package is installed from GitHub via devtools::install_github('castratton/uafR') and run from R or RStudio)
- **Agilent Unknowns Analysis** (Upstream peak-detection/deconvolution software whose exported tentative identifications form the input hit table consumed by spreadOut() and mzExacto())

## Examples

```
input_spread = spreadOut(input_dat); input_exacto = mzExacto(input_spread, query_chemicals, decontaminate = TRUE)
```

## Evaluation signals

- spreadOut() accepts the input only when all six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) are present and rejects duplicate peak keys
- With decontaminate = TRUE, every retained compound has a strong cross-sample match, a PubChem entry, and a canonical SMILES; dropped chemicals are traceable to one of those three failures
- For a known-standards dataset, peak areas correlate with injected volume with R2 in the range reported for uafR (0.8273–0.9998 over 1e-5 to 1e3 ng/μl)
- On a re-analysis (e.g. the Ponce et al. 2022 grain dataset), the number and types of identified compounds are comparable (or identical) to the published manual peak annotations, with accurate retention-time matches
- Downstream statistics (e.g. NMDS or ANOSIM) reproduce the same pattern of significance as the original manual analysis

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral mass spectrometers or on LC-MS data
- MS2/MS3 and ion mobility + MS2 support is deferred until such spectra are published in PubChem's database
- The input .csv has strict column-name requirements; the column names MUST include the six expected fields and duplicates are rejected

## Evidence

- [abstract] automates a grueling retrieval process for gas -chromatography coupled mass spectrometry (GC -MS) data: "We have developed an R package, uafR, that automates a grueling retrieval process for gas -chromatography coupled mass spectrometry (GC -MS) data"
- [intro] aggregate every occurrence of the user-specified chemicals across every GC -MS sample: "aggregate every occurrence of the user-specified chemicals across every GC -MS sample"
- [methods] selected = mzExacto(spread, query_compounds, decontaminate = TRUE): "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [intro] removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable"
- [readme] mzExacto() returns compound identities, optimal retention time, exact mass, best match factor, and aggregated component area across samples.: [evidence span withheld — anchor status: not_found]
- [readme] The column names MUST include: 'Component.RT', 'Component.Area', 'Base.Peak.MZ', 'File.Name', 'Compound.Name', and 'Match.Factor': [evidence span withheld — anchor status: external_doc]
