---
name: spectral-library-data-processing
description: Use when a spectral library's ion-identity annotation strings are inconsistent
  across entries (e.g., 'M + H', 'M + H +', '[M + H]+' all meaning protonation) and
  entries need to be aggregated by true ion identity rather than counted as distinct.
when_to_use_negative:
- When the library's ion-identity annotations are already in a single harmonized bracketed
  format and require no normalization — applying the parser would be redundant.
- When the input is a raw MS/MS peak list (.mgf or mzML spectrum) rather than a tabular
  library of ion-identity annotation strings.
- When ion species were already grouped upstream by an IIMN workflow (e.g., MZmine
  IIN edges with peak shape correlation ≥0.85); the redundancy issue this skill fixes
  is a downstream string-level problem.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Source of the public spectral library (GNPS-IIMN-PROPOGATED) whose ion-identity
    annotation strings are parsed and harmonized by this skill.
  repo: http://gnps.ucsd.edu
- name: GNPS
  role: Hosts the IIMN-based spectral libraries with broad coverage of ion species
    that are the target data for this harmonization skill.
  repo: http://gnps.ucsd.edu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:f002d94cba8e0c185b3d50b9a0d66d4ac451c1683de9aff4e9d92013793446be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# spectral-library-data-processing

## Summary

Processes MS2 spectral libraries (e.g., GNPS-IIMN-PROPOGATED) by parsing and harmonizing ion-identity annotation strings — stripping whitespace, brackets, charge tokens, and normalizing adduct ordering — so that multiple string spellings of the same ion identity collapse into one canonical form.

## When to use

Use when a spectral library's ion-identity annotation strings are inconsistent across entries (e.g., 'M + H', 'M + H +', '[M + H]+' all meaning protonation) and entries need to be aggregated by true ion identity rather than counted as distinct.

## When NOT to use

- When the library's ion-identity annotations are already in a single harmonized bracketed format and require no normalization — applying the parser would be redundant.
- When the input is a raw MS/MS peak list (.mgf or mzML spectrum) rather than a tabular library of ion-identity annotation strings.
- When ion species were already grouped upstream by an IIMN workflow (e.g., MZmine IIN edges with peak shape correlation ≥0.85); the redundancy issue this skill fixes is a downstream string-level problem.

## Inputs

- Public GNPS-IIMN-PROPOGATED spectral library downloaded from https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED
- Ion-identity annotation field from the library records (free-text ion-species strings such as 'M + H', '[M + H]+', 'M + H +')

## Outputs

- harmonized_annotations.csv containing every original row with the canonical bracketed ion-identity string '[M-removed+added]charge'
- Count of distinct raw annotation strings mapped to each canonical identity (aggregation table)

## How to apply

Download the public GNPS-IIMN-PROPOGATED library from GNPS and load its ion-identity annotation field. Implement a parser that strips surrounding whitespace, removes outer brackets and trailing charge tokens, then splits the string into added (+) and removed (-) adduct parts around the letter 'M'. Sort the added and removed parts alphabetically to eliminate ordering ambiguity, and resolve the integer charge from an adduct-to-charge lookup table. Rebuild each annotation in the canonical bracketed form '[M-removed+added]charge' and write all rows to a harmonized CSV file. Validate by confirming that the three protonated-ion variants 'M + H', 'M + H +' and '[M + H]+' all resolve to the identical canonical string '[M + H]+', collapsing what would otherwise be redundant library entries into a single ion identity.

## Related tools

- **GNPS** (Source of the public spectral library (GNPS-IIMN-PROPOGATED) whose ion-identity annotation strings are parsed and harmonized by this skill.) — http://gnps.ucsd.edu
- **GNPS** (Hosts the IIMN-based spectral libraries with broad coverage of ion species that are the target data for this harmonization skill.) — http://gnps.ucsd.edu

## Examples

```
python parse_ion_identities.py --library GNPS-IIMN-PROPOGATED.mgf --output harmonized_annotations.csv
```

## Evaluation signals

- The three protonation spellings 'M + H', 'M + H +', and '[M + H]+' all resolve to the identical canonical string '[M + H]+' in the harmonized output.
- Every row in harmonized_annotations.csv is a valid bracketed string of the form '[M-removed+added]charge' with sorted adduct parts.
- The count of distinct canonical identities is strictly less than the count of distinct raw annotation strings when duplicates exist, demonstrating that aggregation collapses redundant entries.
- Unit checks on a set of annotation strings pass without errors, confirming that whitespace, outer brackets, trailing charge tokens, and adduct ordering are all normalized.
- Charges resolved via the adduct-to-charge lookup table match the integer charge explicitly written in the original annotation strings.

## Limitations

- The parser depends on an adduct-to-charge lookup table; adducts not present in the table will fail to resolve a charge.
- Only string-level ordering and formatting differences are resolved; entries differing in chemical identity (e.g., different adduct combinations) are not merged.
- IIMN-based libraries themselves may still contain multiple string variants pointing to the same ion identity; harmonization must be reapplied to each downloaded library snapshot.

## Evidence

- [results] Ion identity naming in public spectral libraries is not harmonized: multiple string variants point to the same ion identity, so such entries were aggregated into a total count rather than being recognized as identical.: [evidence span withheld — anchor status: not_found]
- [abstract] IIMN-based spectral libraries with a broad coverage of ion species are publicly available.: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [discussion] the GNPS-IIMN-PROPOGATED library URL: "https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED"
- [abstract] IIMN integrates chromatographic peak shape correlation into molecular networks to connect and collapse different ion species of the same molecule.: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
