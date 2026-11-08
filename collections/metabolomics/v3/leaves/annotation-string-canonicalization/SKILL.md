---
name: annotation-string-canonicalization
description: Use when ion-identity annotation strings from GNPS, MassBank of North
  America, or IIMN-propagated library entries exist as inconsistent variants (e.g.,
  'M + H', 'M + H +', '[M + H]+') and must be harmonized before counting or merging
  identities.
when_to_use_negative:
- The input is already a canonicalized/harmonized annotation column — re-parsing adds
  no value and risks re-introducing parsing errors
- The strings are free-text compound names or spectral library metadata other than
  ion-species/adduct identity annotations
- The task needs MS2 peak lists or spectra themselves rather than the ion-identity
  labels attached to library entries
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Hosts the GNPS-IIMN-PROPOGATED spectral library whose ion-identity annotation
    field is the parse input, and the GNPS environment where IIMN-propagated annotations
    originate
  repo: https://ccms-ucsd.github.io/GNPSDocumentation/fbmn-iin/
- name: xcms-gnps-tools
  role: Provides getFeatureAnnotations() to extract adduct annotations in the IIN
    format consumed by FBMN/GNPS, i.e., the producer side of the strings this skill
    canonicalizes
  repo: jorainer/xcms-gnps-tools
- name: mzmine
  role: Feature-finding tool whose MS1-based ion identity annotation output (IIN results)
    feeds IIMN and can produce the adduct strings that need harmonization
  repo: mzmine/mzmine
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:ae5182b5ae2a7efe12fb9d45860e00825eb944a6f47b5fa8eb5fb735c7cc34f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - mzmine
  - xcms-gnps-tools
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# annotation-string-canonicalization

## Summary

Reconstructs and normalizes ion-species annotation strings from public LC-MS2 spectral libraries (e.g., the GNPS-IIMN-PROPOGATED library) into a single canonical bracketed form, so that variant spellings of the same ion identity collapse into one class instead of being counted as separate entries.

## When to use

Use when ion-identity annotation strings from GNPS, MassBank of North America, or IIMN-propagated library entries exist as inconsistent variants (e.g., 'M + H', 'M + H +', '[M + H]+') and must be harmonized before counting or merging identities.

## When NOT to use

- The input is already a canonicalized/harmonized annotation column — re-parsing adds no value and risks re-introducing parsing errors
- The strings are free-text compound names or spectral library metadata other than ion-species/adduct identity annotations
- The task needs MS2 peak lists or spectra themselves rather than the ion-identity labels attached to library entries

## Inputs

- GNPS-IIMN-PROPOGATED spectral library ion-identity annotation field (downloaded from the GNPS library page)
- Raw annotation strings in mixed conventions, e.g. 'M + H', 'M + H +', '[M + H]+'
- Adduct-to-charge lookup table for resolving integer charge states

## Outputs

- harmonized_annotations.csv containing every library row rewritten in the canonical form '[M-removed+added]charge'
- Set of collapsed annotation classes (e.g., one '[M + H]+' identity from three protonation spellings)

## How to apply

Download the GNPS-IIMN-PROPOGATED spectral library from GNPS (https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED) and load its ion-identity annotation field. For each string, strip surrounding whitespace, outer brackets, and trailing charge tokens; split the remaining text into added (+) and removed (-) adduct parts around the 'M' core. Sort the added and removed parts alphabetically to eliminate ordering ambiguity, and resolve the integer charge via an adduct-to-charge lookup table. Rebuild each annotation in the canonical bracketed form '[M-removed+added]charge' and write every row to harmonized_annotations.csv. The rationale comes from the IIMN finding that libraries aggregate multiple spellings of the same ion identity into separate totals, so canonicalization is what makes identity-level counting correct.

## Related tools

- **GNPS** (Hosts the GNPS-IIMN-PROPOGATED spectral library whose ion-identity annotation field is the parse input, and the GNPS environment where IIMN-propagated annotations originate) — https://ccms-ucsd.github.io/GNPSDocumentation/fbmn-iin/
- **xcms-gnps-tools** (Provides getFeatureAnnotations() to extract adduct annotations in the IIN format consumed by FBMN/GNPS, i.e., the producer side of the strings this skill canonicalizes) — jorainer/xcms-gnps-tools
- **mzmine** (Feature-finding tool whose MS1-based ion identity annotation output (IIN results) feeds IIMN and can produce the adduct strings that need harmonization) — mzmine/mzmine

## Examples

```
canonical = '[M' + ''.join('-'+p for p in sorted(removed)) + ''.join('+'+p for p in sorted(added)) + ']' + charge; assert canonical == '[M + H]+'
```

## Evaluation signals

- Unit check: the three written protonation spellings 'M + H', 'M + H +' and '[M + H]+' all resolve to the identical canonical string '[M + H]+' in the saved table
- Every row of harmonized_annotations.csv matches the canonical bracketed pattern '[M-removed+added]charge' after parsing
- Input annotation string ordering is irrelevant: permuting adduct parts before parsing yields the same canonical output because added/removed parts are sorted alphabetically
- Every parsed row has a non-null integer charge resolved from the adduct-to-charge lookup table
- The number of unique annotation classes in the output is less than or equal to the number of unique raw spellings, with known variants collapsing to a single identity

## Limitations

- Parsing assumes well-formed ion-species strings with an 'M' core and +/- separated adduct parts; malformed or unusual adduct spellings outside the lookup table will not resolve to a charge
- Canonicalization depends on the adduct-to-charge lookup table; adducts absent from the table cannot be harmonized
- In the source libraries, inconsistent entries were only detected because they were aggregated into a total count rather than recognized as identical — no formal schema or validator tool exists for IIN annotation formats, so validation must be custom

## Evidence

- [results] Ion identity naming in public spectral libraries is not harmonized: [evidence span withheld — anchor status: not_found]
- [abstract] IIMN-based spectral libraries with a broad coverage of ion species are publicly available.: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [readme] getFeatureAnnotations(): extract adduct annotations for features from a *CAMERA* result in the format required for Ion Identity Networking (IIN) in FBMN/GNPS.: [evidence span withheld — anchor status: not_found]
