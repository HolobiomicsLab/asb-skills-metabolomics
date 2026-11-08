---
name: spectral-library-compound-counting
description: 'Use when characterizing chemical coverage of integrated open mass spectral
  libraries: you need per-library compound counts, spectra-per-compound medians, or
  NPClassifier class distributions to compare library contents.'
when_to_use_negative:
- Input spectra lack resolvable InChIKeys or standardized chemical identifiers — FragHub
  itself removes spectra lacking SMILES/InChI, precursor m/z, and adduct information,
  so counting cannot be anchored on compound identity
- The goal is already-integrated post-FragHub statistics on harmonized output rather
  than characterization of raw heterogeneous libraries
- You need collision-energy or peak-intensity statistics rather than compound-coverage
  composition; collision energies were left unharmonized in FragHub (~70 nonstandardized
  formats)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: FragHub
  role: Provides the harmonized, InChIKey-standardized spectral libraries whose compound
    and spectrum counts form the baseline for grouping and comparison
  repo: https://github.com/eMetaboHUB/FragHub
- name: NPClassifier API
  role: Annotates each unique compound InChIKey with PathwayNP and superclassNP ontology
    terms used to build the per-library class distributions
- name: Python
  role: Implements the counting, median-computation, and aggregation logic for the
    per-library statistics
  repo: https://github.com/eMetaboHUB/FragHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:949b5795264cb60946ea387700497f03f92b4427331274d711576bf7df9a0ec7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - NPClassifier API
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-library-compound-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-library compound and spectrum counts in open mass spectral libraries (GNPS, MassBank, MoNA, MS-Dial-VS17) by grouping .MGF/.MSP/.JSON spectra on InChIKey identifiers, then computing medians of spectra per compound and chemical-class distributions via NPClassifier to characterize library composition.

## When to use

Use when characterizing chemical coverage of integrated open mass spectral libraries: you need per-library compound counts, spectra-per-compound medians, or NPClassifier class distributions to compare library contents.

## When NOT to use

- Input spectra lack resolvable InChIKeys or standardized chemical identifiers — FragHub itself removes spectra lacking SMILES/InChI, precursor m/z, and adduct information, so counting cannot be anchored on compound identity
- The goal is already-integrated post-FragHub statistics on harmonized output rather than characterization of raw heterogeneous libraries
- You need collision-energy or peak-intensity statistics rather than compound-coverage composition; collision energies were left unharmonized in FragHub (~70 nonstandardized formats)

## Inputs

- Raw OMSL library files: GNPS in .MGF format, MassBank, MoNA, MS-Dial-VS17 records (various formats)
- Compound identifiers (InChIKey) associated with each spectrum

## Outputs

- CSV comparison table with one row per library/chemical-class combination
- Per-library median number of spectra per compound (GNPS 2, MassBank 12)
- Per-library chemical class distributions from NPClassifier PathwayNP and superclassNP annotations
- Summary of per-library compound and spectrum totals

## How to apply

Load each raw OMSL file in its native format (GNPS as .MGF, MassBank, MoNA, and MS-Dial-VS17) and group spectra by the compound InChIKey so that the number of spectra each compound contributes can be counted. Compute the median number of spectra per compound per library (e.g. GNPS median 2, MassBank median 12) from these grouped counts. For every unique compound InChIKey, query the NPClassifier API to retrieve PathwayNP and superclassNP annotations, then tabulate compounds and spectra per pathway/superclass per library. Aggregate medians, class distributions, and totals into a comparison table saved as CSV with one row per library/chemical-class combination. Validate the recomputed totals against the published library sizes: 13,507 compounds/63,935 spectra (GNPS), 8,358 compounds/164,261 spectra (MassBank), 21,839 compounds/190,359 spectra (MoNA), and 22,282 compounds/376,430 spectra (MS-Dial-VS17), noting that MS-Dial-VS17 is the only library presplit into positive and negative ionization modes.

## Related tools

- **FragHub** (Provides the harmonized, InChIKey-standardized spectral libraries whose compound and spectrum counts form the baseline for grouping and comparison) — https://github.com/eMetaboHUB/FragHub
- **NPClassifier API** (Annotates each unique compound InChIKey with PathwayNP and superclassNP ontology terms used to build the per-library class distributions)
- **Python** (Implements the counting, median-computation, and aggregation logic for the per-library statistics) — https://github.com/eMetaboHUB/FragHub

## Examples

```
python count_compounds.py --omsl gnps.mgf massbank.msp mona.msp msdial.msp --group-by InChIKey --median spectra_per_compound --npclassifier annotate --out class_distribution.csv
```

## Evaluation signals

- Recomputed compound/spectrum totals match published values: GNPS 13,507 compounds/63,935 spectra; MassBank 8,358/164,261; MoNA 21,839/190,359; MS-Dial-VS17 22,282/376,430
- Computed medians of spectra per compound reproduce the reported values (e.g. GNPS median 2, MassBank median 12)
- Dominant chemical classes per library match the published characterization: fatty acids predominate in GNPS and MassBank, alkaloids in GNPS and MS-Dial-VS17, and MoNA is rich in carbohydrates, amino acids, and peptides
- Aggregated combined totals reproduce 794,985 spectra for 35,673 unique chemical identifiers before FragHub refinement
- CSV schema holds one row per library/chemical-class combination with no duplicated library-class pairs

## Limitations

- Counts depend on FragHub's upstream cleaning: spectra without SMILES, InChI, precursor m/z, or adduct information are removed, so raw-file counts diverge from post-integration counts (794,985 vs 602,744 spectra; 35,673 vs 32,193 chemicals)
- Compound coverage (~32,193) is low relative to the estimated several million natural products, so the class distributions reflect a sparse reference space
- Redundancy across OMSLs may introduce duplicate compounds counted multiple times unless deduplicated via SPLASH keys recorded in 'update.json'
- Chemical class disparity between OMSLs and natural product catalogues means the distributions may not generalize to unmeasured chemical space

## Evidence

- [results] Chemical class coverage is uneven across OMSLs: fatty acids predominate in GNPS and MassBank, alkaloids are prominent in GNPS and MS-Dial-VS17, and MoNA is rich in carbohydrates, amino acids, and peptides.: [evidence span withheld — anchor status: not_found]
- [results] Per-library medians of spectra per compound were computed from InChIKey-grouped spectrum counts.: [evidence span withheld — anchor status: not_found]
- [results] Compound InChIKeys were annotated with NPClassifier pathway and superclass terms.: [evidence span withheld — anchor status: not_found]
- [results] Published library sizes act as validation anchors for the counting procedure.: [evidence span withheld — anchor status: not_found]
- [results] Counts should be computed on raw OMSLs, not on FragHub-harmonized output, since integration refines totals further.: [evidence span withheld — anchor status: not_found]
