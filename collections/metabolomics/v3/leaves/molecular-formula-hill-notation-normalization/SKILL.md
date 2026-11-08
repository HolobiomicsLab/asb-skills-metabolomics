---
name: molecular-formula-hill-notation-normalization
description: Use when a list of de novo molecular formula annotations (e.g. from ZODIAC
  on LC-MS/MS data) must be checked against a structure database and formula strings
  may come in inconsistent element orderings.
when_to_use_negative:
- Input formulas are already guaranteed to be in Hill notation and only a database
  lookup is needed — normalization adds no value and risks transcription errors.
- The goal is element-count arithmetic (e.g. adduct or fragment formula derivation
  like +H2O, +CO2, -NH3 checks at specific m/z offsets) rather than string-level identity
  comparison across databases.
- Formulas carry structural/stereochemical annotations (SMILES, InChI) that should
  be matched at the structure level instead of via formula strings.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ZODIAC
  role: Produces the molecular formula candidates and novel-formula annotations whose
    strings are normalized to Hill notation before database verification
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: SIRIUS
  role: Source of molecular formula annotations (top-50 candidates reranked by ZODIAC)
    and of the intensity-explained metric used in the strict novelty filters feeding
    this normalization step
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: PubChem
  role: Target structure database queried (e.g. via PUG REST fastformula endpoint)
    with the Hill-notation formulas to count hits
- name: ChemSpider
  role: Alternative target structure database for the same novel-formula absence check;
    access requires account credentials
- name: pandas
  role: Loads the deposited novel-formula CSV (Supplementary Table 3-style list) and
    builds the verification table with per-formula hit counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:87b5dd94c21821aaf8c52ae5efb035f8f169d9b374cba8b0824a3892078cebb4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - ChemSpider
  - PubChem
  - SIRIUS
  - ZODIAC
  - pandas
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# molecular-formula-hill-notation-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize molecular formula strings to Hill notation before cross-checking annotated formulas (e.g. ZODIAC/SIRIUS novel formulas) against structure databases such as PubChem or ChemSpider, so that identical formulas string-match reliably and database hit counts are reproducible.

## When to use

Use when a list of de novo molecular formula annotations (e.g. from ZODIAC on LC-MS/MS data) must be checked against a structure database and formula strings may come in inconsistent element orderings.

## When NOT to use

- Input formulas are already guaranteed to be in Hill notation and only a database lookup is needed — normalization adds no value and risks transcription errors.
- The goal is element-count arithmetic (e.g. adduct or fragment formula derivation like +H2O, +CO2, -NH3 checks at specific m/z offsets) rather than string-level identity comparison across databases.
- Formulas carry structural/stereochemical annotations (SMILES, InChI) that should be matched at the structure level instead of via formula strings.

## Inputs

- List of molecular formula strings (e.g. novel formulas from ZODIAC/SIRIUS annotations of LC-MS/MS spectra, as in Supplementary Table 3/4 style CSV files)
- Formula strings possibly in non-Hill element orderings from tool output or supplementary tables

## Outputs

- Normalized Hill-notation formula strings, one per input row
- Verification table pairing each Hill-notation formula with its structure-database hit count (e.g. pubchem_novel_formula_verification.csv with 62 rows, all counts 0 for genuinely novel formulas)

## How to apply

Collect the molecular formula strings to verify, for example the novel formulas reported for the tomato and diatoms datasets after applying the strict filters (ZODIAC score ≥ 0.999, ≥ 95 % of MS/MS spectrum intensity explained by SIRIUS, and a formula connected to ≥ 20 compounds in the similarity network). Parse each formula into its element/count tokens and re-serialize in Hill notation: carbon first, hydrogen second, all other elements in alphabetical order, omitting counts of 1. Normalizing to Hill notation is the rationale for making 'is this formula already in PubChem or ChemSpider' a deterministic string-level comparison rather than a parser-dependent one; it underpins the paper's claim that 62 novel formulas are absent from the largest public structure databases. After normalization, feed each formula into a database formula search (e.g. PubChem PUG REST fastformula endpoint) and record hit counts; a count of 0 for every normalized formula is the expected result for genuinely novel formulas. Verify the output table has one row per formula with the Hill-notation string and the hit count, and that no formula was silently dropped or re-spelled inconsistently.

## Related tools

- **ZODIAC** (Produces the molecular formula candidates and novel-formula annotations whose strings are normalized to Hill notation before database verification) — https://bio.informatik.uni-jena.de/software/sirius/
- **SIRIUS** (Source of molecular formula annotations (top-50 candidates reranked by ZODIAC) and of the intensity-explained metric used in the strict novelty filters feeding this normalization step) — https://bio.informatik.uni-jena.de/software/sirius/
- **PubChem** (Target structure database queried (e.g. via PUG REST fastformula endpoint) with the Hill-notation formulas to count hits)
- **ChemSpider** (Alternative target structure database for the same novel-formula absence check; access requires account credentials)
- **pandas** (Loads the deposited novel-formula CSV (Supplementary Table 3-style list) and builds the verification table with per-formula hit counts)

## Examples

```
import pandas as pd; df = pd.read_csv('compounds_with_novel_molecular_formulas.csv'); df['formula_hill'] = df['formula'].apply(normalize_hill); counts = [pubchem_fastformula_hit_count(f) for f in df['formula_hill']]; pd.DataFrame({'formula': df['formula_hill'], 'pubchem_hits': counts}).to_csv('pubchem_novel_formula_verification.csv', index=False)
```

## Evaluation signals

- Every formula in the output table parses to identical element counts as its input string (round-trip parse equality).
- All output formula strings follow Hill ordering: C first, H second, remaining elements alphabetically, no explicit count of 1.
- Row count of the verification table equals the number of input formulas (e.g. exactly 62 for the 59 diatom + 3 tomato novel formulas).
- Hit-count field present for every row; for genuinely novel formulas every count equals 0 (e.g. the PubChem fastformula check returned 0 compound IDs for all 62 formulas).
- No duplicate normalized formula strings appear unless duplicates exist in the input.

## Limitations

- Hill-notation normalization is a string-level identity check; formulas differing only by isotopic or adduct annotations (e.g. ±H2O, ±CO2, ±NH3 equivalents) must be handled separately — the article explicitly checked MS1 peaks at m/z +18.01, +43.99 and −17.03 for such artifacts.
- Structure-database access may require credentials (ChemSpider API) or network access (PubChem REST), so the absence check cannot always be fully automated; the article deferred the ChemSpider check to expert review for this reason.
- Normalization does not guarantee the formula is chemically novel in any deeper sense — a zero-hit count only means absence from the queried databases (PubChem, ChemSpider), and searching an unknown compound with a novel formula will always return an incorrect database hit that may go unnoticed.

## Evidence

- [supplementary] Normalize each formula string to Hill notation: [evidence span withheld — anchor status: not_found]
- [results] Isotope/adduct-related peaks checked at specific m/z offsets: [evidence span withheld — anchor status: not_found]
