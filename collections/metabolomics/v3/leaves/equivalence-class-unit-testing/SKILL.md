---
name: equivalence-class-unit-testing
description: Use when building or validating a parser that normalizes ion-identity
  annotation strings from public LC-MS2 spectral libraries (e.g. GNPS-IIMN-PROPOGATED)
  and you need a concrete check that distinct spellings ('M + H', 'M + H +', '[M +
  H]+') resolve to one canonical identity.
when_to_use_negative:
- Input annotation strings are already in one canonical harmonized form and no aggregation-by-variant
  risk exists
- Applying to non-ion-identity metadata fields (e.g. retention times, m/z columns)
  where the bracket/charge-token grammar does not apply
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Source of the public spectral library (GNPS-IIMN-PROPOGATED) whose ion-identity
    annotation strings supply the equivalence-class inputs
  repo: https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:a029702ae6462ebda0eac4ab00987a90dc781eb33fd2148397badb99b19e7013
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# equivalence-class-unit-testing

## Summary

A unit-testing approach for harmonizing ion-identity annotation strings (e.g. GNPS IIMN ion species labels) in which multiple written variants of the same ion species are verified to collapse into one identical canonical output, exposing inconsistent annotation syntaxes in public spectral libraries.

## When to use

Use when building or validating a parser that normalizes ion-identity annotation strings from public LC-MS2 spectral libraries (e.g. GNPS-IIMN-PROPOGATED) and you need a concrete check that distinct spellings ('M + H', 'M + H +', '[M + H]+') resolve to one canonical identity.

## When NOT to use

- Input annotation strings are already in one canonical harmonized form and no aggregation-by-variant risk exists
- Applying to non-ion-identity metadata fields (e.g. retention times, m/z columns) where the bracket/charge-token grammar does not apply

## Inputs

- Public spectral library table with ion-identity annotation field (e.g. GNPS-IIMN-PROPOGATED from https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED)
- Set of expected-equivalent annotation spellings per ion identity (e.g. protonation variants)
- Adduct-to-charge lookup table

## Outputs

- harmonized_annotations.csv with canonical bracketed ion-identity strings '[M-removed+added]charge' per row
- Unit-test result: pass/fail evidence that an equivalence class of spellings collapses to a single canonical string (e.g. three protonated variants → one '[M + H]+' identity)

## How to apply

Load the ion-identity annotation field from a public spectral library such as the GNPS-IIMN-PROPOGATED library and write a parser that strips whitespace, outer brackets and trailing charge tokens, splits the string around 'M' into added (+) and removed (−) adduct parts, sorts those parts alphabetically to remove ordering ambiguity, and resolves integer charge from an adduct-to-charge lookup table. Rebuild each annotation in the canonical bracketed form '[M-removed+added]charge' and write all rows to harmonized_annotations.csv. Define equivalence classes of input spellings that should map to the same ion identity, then assert that all members of each class produce one identical canonical string. Because libraries are not harmonized — multiple string variants point to the same ion identity, and such entries were aggregated into a total count rather than recognized as identical — unit tests must collapse the classes before any downstream counting or merging.

## Related tools

- **GNPS** (Source of the public spectral library (GNPS-IIMN-PROPOGATED) whose ion-identity annotation strings supply the equivalence-class inputs) — https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED

## Examples

```
python harmonize_iin.py --library GNPS-IIMN-PROPOGATED --test "'M + H','M + H +','[M + H]+'->'[M + H]+'" --output harmonized_annotations.csv
```

## Evaluation signals

- The three protonated-ion variants 'M + H', 'M + H +' and '[M + H]+' all resolve to the identical canonical string '[M + H]+' in the harmonized output
- Harmonized annotations match the bracketed pattern '[M-removed+added]charge' with added and removed adduct parts sorted alphabetically
- No two distinct input spellings of the same ion identity are aggregated as separate entries in harmonized_annotations.csv
- Integer charge is correctly resolved from the adduct-to-charge lookup for every parsed row

## Limitations

- Public spectral libraries are not harmonized, so variant aggregation must be handled before any counting; the parser assumes a bracketed '[M±adduct]charge' grammar and will fail on free-form or non-standard labels
- Charge resolution depends on a complete adduct-to-charge lookup table; adducts absent from the table cannot be canonicalized

## Evidence

- [other] Ion identity naming in public spectral libraries is not harmonized: multiple string variants point to the same ion identity, so such entries were aggregated into a total count rather than being recognized as identical.: [evidence span withheld — anchor status: not_found]
- [other] Parser steps: strip whitespace, outer brackets, trailing charge tokens; split added/removed adduct parts around 'M'; sort parts alphabetically; resolve charge via lookup; rebuild canonical form '[M-removed+added]charge'.: [evidence span withheld — anchor status: not_found]
- [other] Canonical output form.: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion: three protonation spellings produce one identical canonical output.: [evidence span withheld — anchor status: not_found]
- [readme] README confirming the GNPS spectral library environment referenced for the IIMN library.: [evidence span withheld — anchor status: external_doc]
