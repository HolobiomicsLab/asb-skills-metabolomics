---
name: lc-ms-vs-afadesi-msi-adduct-difference-evaluation
description: 'Use when building or extending a metabolite annotation database for
  AFADESI-MSI data: metabolites identified by LC-MS/MS in tissue extracts must be
  re-represented as the adduct ions actually formed during ambient MSI desorption
  before m/z matching can be done.'
when_to_use_negative:
- The MSI dataset has already been annotated against a database supplied in adduct-ion
  form — re-evaluating LC-MS vs AFADESI-MSI adduct differences would duplicate Database
  preparation
- Input is raw MSI data rather than metabolite lists; the adduct-difference evaluation
  operates on identified metabolites and their adducts, not on unprocessed mass spectra
- Analyzing a tissue or biofluid whose organ-specific metabolite sheet is absent from
  'Organ specific metabolite database.xlsx' — adduct forms must first be curated for
  that organ set before the difference evaluation can be run
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MSIannotator
  role: Consumes the adduct-ion database produced by this evaluation to annotate metabolites
    and adducts in preprocessed MSI data by m/z matching
  repo: Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: MATLAB script that performs the annotation step the adduct-difference-evaluated
    database enables; theoretical m/z values from the database are matched against
    measured m/z at a default 10 ppm error
- name: MATLAB
  role: Runtime environment in which the database sheets are imported as string arrays
    (MSIdata and Database) and the annotation is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:7f87fc44792a56a4d9c90b408d1d677a44a898039ff912dce45c81f0cb6c2827
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lc-ms-vs-afadesi-msi-adduct-difference-evaluation

## Summary

Evaluates the adduct-form differences of metabolite ions between LC-MS and AFADESI-MSI to expand an organ-specific metabolite database into an adduct-ion database for MSI annotation (27,407 adduct ions from 1,298 positive-mode and 515 negative-mode metabolites).

## When to use

Use when building or extending a metabolite annotation database for AFADESI-MSI data: metabolites identified by LC-MS/MS in tissue extracts must be re-represented as the adduct ions actually formed during ambient MSI desorption before m/z matching can be done.

## When NOT to use

- The MSI dataset has already been annotated against a database supplied in adduct-ion form — re-evaluating LC-MS vs AFADESI-MSI adduct differences would duplicate Database preparation
- Input is raw MSI data rather than metabolite lists; the adduct-difference evaluation operates on identified metabolites and their adducts, not on unprocessed mass spectra
- Analyzing a tissue or biofluid whose organ-specific metabolite sheet is absent from 'Organ specific metabolite database.xlsx' — adduct forms must first be curated for that organ set before the difference evaluation can be run

## Inputs

- Organ specific metabolite database.xlsx (1,298 positive-mode and 515 negative-mode metabolites, organized into brain, liver, kidney, heart, spleen, lung, muscle, pancreas and serum sheets)
- LC-MS metabolite identification results with adduct assignments
- AFADESI-MSI peak lists with measured m/z values for adduct-form comparison

## Outputs

- Expanded organ-specific adduct-ion database (27,407 adduct ions) with metabolite name, adduct form, and theoretical m/z per organ and ionization mode
- Database sheets (e.g., Database.xlsx format: metabolite names in first column, theoretical m/z in adduct-form columns) consumable by MSIannotator.m

## How to apply

Start from the organ-specific metabolite lists in 'Organ specific metabolite database.xlsx' (1,298 metabolites in positive mode Sheet1, 515 metabolites in negative mode Sheet2, partitioned into brain, liver, kidney, heart, spleen, lung, muscle, pancreas and serum sheets). For each organ/metabolite, compare the adduct forms observed in LC-MS against those observed for AFADESI-MSI, retaining the adduct species that form under ambient airflow-assisted desorption electrospray ionization conditions rather than the LC-ESI forms. Compute the theoretical m/z for every metabolite-adduct pair using standard adduct mass calculations, matching the theoretical-m/z columns used in Database.xlsx, then aggregate the enumerated adduct ions per organ sheet and ionization mode. The resulting adduct-ion database feeds MSIannotator.m, which matches measured m/z in preprocessed MSIdata against these theoretical m/z values within a 10 ppm error window. The total of 27,407 adduct ions serves as the reference-scale validation that the LC-MS-to-AFADESI-MSI adduct translation was applied across the full 1,813-metabolite space.

## Related tools

- **MSIannotator** (Consumes the adduct-ion database produced by this evaluation to annotate metabolites and adducts in preprocessed MSI data by m/z matching) — Yingzhu96/MSIannotator
- **MSIannotator.m** (MATLAB script that performs the annotation step the adduct-difference-evaluated database enables; theoretical m/z values from the database are matched against measured m/z at a default 10 ppm error)
- **MATLAB** (Runtime environment in which the database sheets are imported as string arrays (MSIdata and Database) and the annotation is executed)

## Examples

```
run MSIannotator.m  % MSIdata and Database (adduct-expanded organ sheets) imported as string arrays; default 10 ppm m/z error
```

## Evaluation signals

- The aggregated adduct-ion count across all organ sheets and both ionization modes must equal 27,407 adduct ions
- Each database row must carry a theoretical m/z computed for the assigned adduct form, consistent with the theoretical-m/z column layout of Database.xlsx
- Annotation results (Annotator file) must list both the metabolite name and its adduct form, showing the LC-MS metabolite was successfully re-represented as an MSI adduct ion
- m/z matching in downstream annotation must satisfy abs(theor-mea)/theor < 10*10^-6 (default 10 ppm) for the adduct ions emitted from this database
- Positive-mode sheets must be matched with positive-mode MSIdata and negative-mode sheets with negative-mode MSIdata (organ and detection mode must be consistent)

## Limitations

- The evaluation is grounded in organ-specific metabolite sets (eight mouse organs plus serum); tissues outside these nine categories are not covered by the established 27,407-ion database
- The adduct forms retained are those observed for AFADESI-MSI, so adduct species unique to other ionization sources or other MSI geometries may not be represented
- The database is provided in Excel format with fixed sheet organization; users altering the sheet structure must maintain the metabolite-name/theoretical-m/z column conventions expected by MSIannotator.m

## Evidence
