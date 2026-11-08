---
name: adduct-notation-normalization
description: Use when merging spectral-library or feature-table records whose ion-identity
  annotations appear in inconsistent spellings (e.g. 'M + H', 'M + H +', '[M + H]+')
  and you need them aggregated as one ion identity before networking or counting.
when_to_use_negative:
- Input records already carry the canonical bracketed ion-species string, e.g. '[M
  + H]+', so parsing would be a no-op
- The task is raw-peak feature grouping or adduct *assignment* from m/z differences
  — that is the IIMN MS1-based ion identity networking step (retention time tolerance
  0.1 min, m/z tolerance 0.001 or 10 ppm), not string normalization
- Annotations contain free-text compound names or InChIKeys rather than bracketed
  ion-species strings; no adduct parts can be split around 'M'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: GNPS
  role: Source of the public GNPS-IIMN-PROPOGATED spectral library whose ion-identity
    annotation strings are parsed and harmonized
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Feature-finding tool that exports ion-identity/adduct annotations feeding
    the IIMN workflow; its annotations can carry the non-canonical spellings normalized
    here
  repo: https://github.com/mzmine/mzmine
- name: xcms-gnps-tools
  role: getFeatureAnnotations() extracts CAMERA adduct annotations for features in
    the format required for Ion Identity Networking; these strings are the input to
    normalization
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:d5fa8590341e8e6dc474804b5147fc6051266fee67f22212cf1794158fc6a33e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MZmine
  - xcms-gnps-tools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Adduct-notation-normalization

## Summary

Parses and harmonizes heterogeneous ion-identity annotation strings from spectral libraries (e.g. the GNPS-IIMN-PROPOGATED library) into a single canonical bracketed ion-species form so that multiple spellings of the same adduct collapse into one identity.

## When to use

Use when merging spectral-library or feature-table records whose ion-identity annotations appear in inconsistent spellings (e.g. 'M + H', 'M + H +', '[M + H]+') and you need them aggregated as one ion identity before networking or counting.

## When NOT to use

- Input records already carry the canonical bracketed ion-species string, e.g. '[M + H]+', so parsing would be a no-op
- The task is raw-peak feature grouping or adduct *assignment* from m/z differences — that is the IIMN MS1-based ion identity networking step (retention time tolerance 0.1 min, m/z tolerance 0.001 or 10 ppm), not string normalization
- Annotations contain free-text compound names or InChIKeys rather than bracketed ion-species strings; no adduct parts can be split around 'M'

## Inputs

- Ion-identity annotation string column from a GNPS spectral library table (e.g. GNPS-IIMN-PROPOGATED .tsv)
- Adduct annotations exported by feature-finding tools (MZmine, MS-DIAL, XCMS) in IIN supplementary edges (.csv) format
- Adduct-to-charge lookup table

## Outputs

- harmonized_annotations.csv — one canonical bracketed ion-species string per input record (e.g. '[M + H]+')
- Aggregated ion-identity counts where previously distinct string variants collapse to a single identity

## How to apply

Load the ion-identity annotation field from the spectral library (e.g. the GNPS-IIMN-PROPOGATED library at https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED) or from MZmine/xcms-gnps-tools IIN feature annotations. Strip surrounding whitespace, remove outer brackets and trailing charge tokens, split the remaining string into added (+) and removed (−) adduct parts around 'M', sort each side alphabetically to remove ordering ambiguity, and resolve integer charge from an adduct-to-charge lookup table. Rebuild each entry in the canonical bracketed form '[M-removed+added]charge' and write all rows to harmonized_annotations.csv. Validate by unit checks confirming that the three protonation spellings 'M + H', 'M + H +' and '[M + H]+' resolve to the identical canonical string '[M + H]+'; harmonization is accepted only when all annotation classes collapse to a single '[M + H]+' identity, since public libraries store multiple string variants that point to the same ion identity and must be aggregated as one total count rather than being recognized as identical.

## Related tools

- **GNPS** (Source of the public GNPS-IIMN-PROPOGATED spectral library whose ion-identity annotation strings are parsed and harmonized) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MZmine** (Feature-finding tool that exports ion-identity/adduct annotations feeding the IIMN workflow; its annotations can carry the non-canonical spellings normalized here) — https://github.com/mzmine/mzmine
- **xcms-gnps-tools** (getFeatureAnnotations() extracts CAMERA adduct annotations for features in the format required for Ion Identity Networking; these strings are the input to normalization) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
python harmonize_iin_annotations.py --input GNPS-IIMN-PROPOGATED.tsv --column 'IonSpecies' --output harmonized_annotations.csv
```

## Evaluation signals

- Unit check: 'M + H', 'M + H +' and '[M + H]+' all resolve to the identical canonical string '[M + H]+' in harmonized_annotations.csv
- Schema check: every output row matches the canonical bracketed pattern '[M-removed+added]charge' with alphabetical ordering of adduct parts on each side
- Count check: aggregating by canonical string yields fewer distinct ion identities than the raw annotation column (multiple spellings collapse to one identity)
- Charge check: parsed integer charge equals the value given by the adduct-to-charge lookup table for the resolved adduct parts

## Limitations

- Multiple string variants point to the same ion identity in public spectral libraries, so naive aggregation counts them separately; the harmonization parser is required to resolve this
- The MZmine IIMN build is described as a 'special IIMN build' — unclear whether its annotation export is part of a released MZmine version or a custom build
- Charge states that cannot be resolved by the lookup table remain unharmonized; annotations with unrecognized adduct tokens will fail the canonical rebuild

## Evidence

- [full_text] Ion identity naming in public spectral libraries is not harmonized: multiple string variants point to the same ion identity, so such entries were aggregated into a total count rather than being recognized as identical.: [evidence span withheld — anchor status: not_found]
- [methods] Three protonated-ion spellings collapse to one canonical identity: [evidence span withheld — anchor status: not_found]
- [discussion] Public IIMN spectral library is the concrete input for normalization: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [readme] xcms-gnps-tools supplies CAMERA adduct annotations in the IIN format: [evidence span withheld — anchor status: not_found]
- [discussion] Source library location for the ion-identity annotation column: "https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED"
