---
name: spectral-metadata-configuration
description: Use when running SIRIUS (or similar formula/structure annotation tools)
  on query spectra from public repositories and per-spectrum adduct, ionization mode,
  and instrument type must be set from the original GNPS/MassIVE metadata rather than
  left at defaults.
when_to_use_negative:
- Input spectra lack reliable adduct or instrument metadata (e.g. no GNPS/MassIVE
  provenance), so per-spectrum configuration would be guesswork
- The analysis goal is BAM's biotransformation-based candidate generation and ranking
  (PROXIMAL2/GNN-SOM), not SIRIUS formula/structure annotation
- The query set was not filtered for compounds present in the structural database,
  so formula/structure annotation against bioDB would be meaningless for most queries
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS 5.6.3
  role: Annotation engine whose CLI is configured with adduct, ionization mode, and
    instrument type from GNPS/MassIVE metadata, default elements, and 10 ppm mass
    accuracy; performs molecular formula and then structure annotation against bioDB
- name: GNPS/MassIVE
  role: Public data repository supplying the query spectra and the experimental metadata
    (adduct, ionization mode, instrument type) used to configure SIRIUS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:707e64fd019630977dfe1028a7a65eb0606a794f6a1c3b4d280ae8d47b41dfe7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS/MassIVE
  - SIRIUS 5.6.3
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-metadata-configuration

## Summary

Configure SIRIUS molecular formula/structure annotation using per-spectrum experimental metadata (adduct, ionization mode, instrument type) harvested from GNPS/MassIVE records, with the default element set (C, H, N, O, P, S, Br, Cl, B, Se) and 10 ppm mass accuracy. Correct metadata configuration is a prerequisite for reproducing the SIRIUS baseline on the 38,368-spectrum anchor–suspect query set.

## When to use

Use when running SIRIUS (or similar formula/structure annotation tools) on query spectra from public repositories and per-spectrum adduct, ionization mode, and instrument type must be set from the original GNPS/MassIVE metadata rather than left at defaults.

## When NOT to use

- Input spectra lack reliable adduct or instrument metadata (e.g. no GNPS/MassIVE provenance), so per-spectrum configuration would be guesswork
- The analysis goal is BAM's biotransformation-based candidate generation and ranking (PROXIMAL2/GNN-SOM), not SIRIUS formula/structure annotation
- The query set was not filtered for compounds present in the structural database, so formula/structure annotation against bioDB would be meaningless for most queries

## Inputs

- 38,368 unique query spectra from the anchor–suspect data set derived from the global molecular network (filtered for compounds present in bioDB)
- GNPS/MassIVE metadata records providing adduct, ionization mode, and instrument type per spectrum
- SIRIUS 5.6.3 command line interface with the bioDB structural database

## Outputs

- SIRIUS molecular formula annotations per query spectrum
- SIRIUS molecular structure annotations (only for queries with a generated formula)
- Prediction table with one row per query spectrum (38,368 rows) and correctness flags for formula and structure

## How to apply

For each query spectrum in the 38,368-spectrum anchor–suspect set (compounds present in bioDB, derived from the global molecular network), look up the adduct, ionization mode, and instrument type recorded in the corresponding GNPS/MassIVE metadata and pass them to the SIRIUS 5.6.3 command line interface. Keep the default element set (C, H, N, O, P, S, Br, Cl, B, Se) and the 10 ppm mass accuracy setting rather than widening the search space. Run molecular formula annotation first against the biomolecule structure database (bioDB), and only for spectra where the correct molecular formula was generated proceed to molecular structure annotation. Aggregate per-query results into a prediction table covering all 38,368 queries so downstream counts (4756 correct formulas, 4560 correct structures) can be validated against the reference values.

## Related tools

- **SIRIUS 5.6.3** (Annotation engine whose CLI is configured with adduct, ionization mode, and instrument type from GNPS/MassIVE metadata, default elements, and 10 ppm mass accuracy; performs molecular formula and then structure annotation against bioDB)
- **GNPS/MassIVE** (Public data repository supplying the query spectra and the experimental metadata (adduct, ionization mode, instrument type) used to configure SIRIUS)

## Examples

```
sirius --formula --structure --db bioDB --elements CHNOPSBBrClSe --mass-accuracy 10ppm --instrument <gnps_metadata_instrument> --ion-mode <ionization_mode> --adduct <adduct> <query.mgf>
```

## Evaluation signals

- Prediction table row count equals exactly 38,368 (one row per query spectrum)
- Reported count of correctly annotated molecular formulas equals 4756
- Reported count of correctly annotated molecular structures equals 4560
- Formula annotation is attempted for every query, while structure annotation is run only where a formula was generated (hierarchical gating)
- SIRIUS configuration log shows per-spectrum adduct/ionization/instrument values matching GNPS/MassIVE metadata and 10 ppm mass accuracy with default element set

## Limitations

- Structure annotation is gated on formula annotation: if SIRIUS fails to generate the correct molecular formula, no structure annotation is attempted for that query, capping the achievable structure count
- Configuration depends on the quality of GNPS/MassIVE metadata; incorrect adduct or instrument type entries propagate directly into SIRIUS settings
- Baseline is specific to SIRIUS 5.6.3 with the bioDB database and the default element set; results do not transfer to other databases or element configurations

## Evidence

- [methods] Configure SIRIUS 5.6.3 via the command line interface, setting adduct, ionization mode, and instrument type from the corresponding GNPS/MassIVE metadata, with default elements (C, H, N, O, P, S, Br, Cl, B, Se) and 10 ppm mass accuracy.: [evidence span withheld — anchor status: not_found]
- [methods] we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation.: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [methods] verify the prediction table row count equals 38,368 and that the reported true formula count (4756) and true structure count (4560) match the reference values: [evidence span withheld — anchor status: not_found]
