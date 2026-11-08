---
name: comparative-benchmark-evaluation-against-sirius
description: Use when a new MS/MS molecular structure annotation method needs a fair
  baseline comparison on identical query spectra, and SIRIUS is an appropriate reference
  tool with a defined structural database.
when_to_use_negative:
- When the candidate method uses a different structural database or query spectrum
  set — the comparison is only fair if SIRIUS runs on identical queries and the same
  database.
- When the goal is de novo annotation itself rather than benchmarking an annotation
  method against an existing tool.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 5.6.3
  role: Baseline tool for molecular formula and molecular structure annotation on
    the identical query spectra set, executed via its command line interface
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:6dc8b3d721acbe8fe2b2324b0a3723a799ee413a57540f9053fdae93c8597402
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SIRIUS
  - SIRIUS 5.6.3
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# comparative-benchmark-evaluation-against-sirius

## Summary

Benchmarks a candidate annotation method against SIRIUS 5.6.3 by running SIRIUS formula-then-structure annotation on the same query spectra (38,368 anchor–suspect queries from the global molecular network against bioDB) and comparing correct formula (4,756) and structure (4,560) counts.

## When to use

Use when a new MS/MS molecular structure annotation method needs a fair baseline comparison on identical query spectra, and SIRIUS is an appropriate reference tool with a defined structural database.

## When NOT to use

- When the candidate method uses a different structural database or query spectrum set — the comparison is only fair if SIRIUS runs on identical queries and the same database.
- When the goal is de novo annotation itself rather than benchmarking an annotation method against an existing tool.

## Inputs

- 38,368 unique query spectra from the anchor–suspect data set (filtered for compounds present in bioDB), with GNPS/MassIVE metadata (adduct, ionization mode, instrument type)
- bioDB structural database for SIRIUS formula and structure annotation
- Prediction table from the method under evaluation on the same query spectra

## Outputs

- Per-query prediction table for SIRIUS with row count equal to the number of query spectra (38,368)
- Counts of correctly annotated molecular formulas (4,756) and molecular structures (4,560) for the baseline comparison

## How to apply

Run SIRIUS 5.6.3 via its command line interface on the exact same query spectra set used by the method under evaluation — here 38,368 unique query spectra filtered to compounds present in bioDB, derived from 60,368 anchor–suspect pairs in the global molecular network. Set adduct, ionization mode, and instrument type from the GNPS/MassIVE metadata, keep default elements (C, H, N, O, P, S, Br, Cl, B, Se) and 10 ppm mass accuracy. Mirror the target method's two-stage evaluation protocol: first perform molecular formula annotation, and only if the correct formula was generated, perform molecular structure annotation against bioDB. Aggregate per-query results into a prediction table (row count must equal 38,368), count correct formulas and correct structures, then compare those counts (4,756 formulas / 4,560 structures) with the candidate method on the same queries and database.

## Related tools

- **SIRIUS 5.6.3** (Baseline tool for molecular formula and molecular structure annotation on the identical query spectra set, executed via its command line interface)

## Examples

```
sirius --formula --structure --adduct [M+H]+ --ionization-mode positive --instrument-type qtof --ppm-max 10 --database bioDB query_spectra.mgf
```

## Evaluation signals

- Prediction table row count equals 38,368 query spectra.
- Reported correct molecular formula count matches 4,756.
- Reported correct molecular structure count matches 4,560.
- SIRIUS was applied with the same two-stage protocol as the target method: formula annotation first, structure annotation only when the correct formula was generated.
- SIRIUS configuration (adduct, ionization mode, instrument type, default elements C/H/N/O/P/S/Br/Cl/B/Se, 10 ppm) matches the candidate method's settings derived from GNPS/MassIVE metadata.

## Limitations

- The SIRIUS baseline only reports molecular structure annotation when the correct molecular formula was generated, so structure counts underestimate cases where SIRIUS could have ranked a correct structure under an incorrect formula.
- SIRIUS is configured with default elements and 10 ppm accuracy; alternative element sets or tolerances may change the baseline counts.
- Benchmark validity depends on using the same query spectra (38,368) and the same database (bioDB) as the method being compared.

## Evidence

- [methods] query spectra count and filter: [evidence span withheld — anchor status: not_found]
- [methods] two-stage annotation protocol: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [results] baseline results: [evidence span withheld — anchor status: not_found]
- [results] validation of counts: [evidence span withheld — anchor status: not_found]
