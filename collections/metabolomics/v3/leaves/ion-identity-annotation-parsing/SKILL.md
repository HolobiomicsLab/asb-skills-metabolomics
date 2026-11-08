---
name: ion-identity-annotation-parsing
description: Use when a feature quantification table, IIN edges file, or GNPS library
  (e.g., GNPS-IIMN-PROPOGATED) contains ion-identity annotation strings with inconsistent
  spellings or ordering that must be normalized before grouping, counting, or comparing
  adduct abundances.
when_to_use_negative:
- The annotation column already contains canonical bracketed strings from a single
  harmonized source — re-parsing adds no information
- The data are raw MS/MS spectra or MGF peak lists without MS1-based ion-identity
  grouping (IIN) results, so no adduct annotations exist to parse
- The goal is MS2 spectral similarity networking or library matching, which operates
  on fragmentation spectra, not ion-identity strings
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the IIMN workflow and the GNPS-IIMN-PROPOGATED library whose ion-identity
    annotation strings are parsed and harmonized
  repo: CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Performs MS1-based feature grouping and ion identity annotation, exporting
    the IIN CSVs whose adduct annotation strings are normalized by this skill
  repo: mzmine/mzmine
- name: xcms-gnps-tools
  role: getFeatureAnnotations() extracts adduct annotations from CAMERA results in
    the IIN format required for FBMN/GNPS, providing parser input
  repo: jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:23522f9ac084bf41c4227eeb2876840f6f5229d4cd9d64b039fa28ec33c04e38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MZmine
  - xcms-gnps-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ion-identity-annotation-parsing

## Summary

Parses and harmonizes ion-identity/adduct annotation strings (e.g., 'M + H', '[M+Na]+') that appear in GNPS-IIMN and IIN feature tables, collapsing spelling variants into one canonical bracketed identity so downstream adduct-level statistics are computed on true ion identities rather than on redundant string variants.

## When to use

Use when a feature quantification table, IIN edges file, or GNPS library (e.g., GNPS-IIMN-PROPOGATED) contains ion-identity annotation strings with inconsistent spellings or ordering that must be normalized before grouping, counting, or comparing adduct abundances.

## When NOT to use

- The annotation column already contains canonical bracketed strings from a single harmonized source — re-parsing adds no information
- The data are raw MS/MS spectra or MGF peak lists without MS1-based ion-identity grouping (IIN) results, so no adduct annotations exist to parse
- The goal is MS2 spectral similarity networking or library matching, which operates on fragmentation spectra, not ion-identity strings

## Inputs

- GNPS-IIMN-PROPOGATED spectral library ion-identity annotation field (downloaded from https://gnps.ucsd.edu/ProteoSAFe/gnpslibrary.jsp?library=GNPS-IIMN-PROPOGATED)
- IIN supplementary edges/annotation CSV files exported by MZmine, MS-DIAL, or XCMS (xcms-gnps-tools getFeatureAnnotations format)
- Feature quantification table (.csv) with per-sample abundances tagged by ion-identity annotation

## Outputs

- harmonized_annotations.csv with one canonical '[M-removed+added]charge' string per row
- Normalized per-adduct abundance table (e.g., [M+Na]+, [M+NH4]+ counts/intensities per condition)
- Collapsed annotation-class counts showing merged spelling variants

## How to apply

Load the ion-identity annotation column from the IIMN-propagated library or IIN-exported CSV, strip surrounding whitespace, outer brackets, and trailing charge tokens, then split the string into added (+) and removed (-) adduct parts around 'M'. Sort the added and removed parts alphabetically to eliminate ordering ambiguity, and resolve the integer charge from an adduct-to-charge lookup table (e.g., H→+1, Na→+1, NH4→+1). Rebuild every annotation in the canonical bracketed form '[M-removed+added]charge' and write all rows to harmonized_annotations.csv. This harmonization matters because public libraries are not harmonized: multiple string variants point to the same ion identity, so without canonicalization such entries are aggregated as separate identities rather than recognized as identical. The canonical strings then support dataset-level checks such as Welch t-tests of [M+Na]+ and [M+NH4]+ abundance under salt-infusion versus H2O control conditions.

## Related tools

- **GNPS** (Hosts the IIMN workflow and the GNPS-IIMN-PROPOGATED library whose ion-identity annotation strings are parsed and harmonized) — CCMS-UCSD/GNPS_Workflows
- **MZmine** (Performs MS1-based feature grouping and ion identity annotation, exporting the IIN CSVs whose adduct annotation strings are normalized by this skill) — mzmine/mzmine
- **xcms-gnps-tools** (getFeatureAnnotations() extracts adduct annotations from CAMERA results in the IIN format required for FBMN/GNPS, providing parser input) — jorainer/xcms-gnps-tools

## Examples

```
python harmonize_iimn_annotations.py --input GNPS-IIMN-PROPOGATED.csv --output harmonized_annotations.csv
```

## Evaluation signals

- The three protonation spellings 'M + H', 'M + H +', and '[M + H]+' all resolve to the single canonical string '[M+H]+' in the saved harmonized table
- Annotation classes strictly decrease (or stay equal) in count after harmonization relative to the raw strings, confirming variant merging without identity loss
- Every output string matches the canonical bracketed pattern '[M-removed+added]charge' with adduct parts in alphabetical order
- Resolved integer charge equals the adduct-to-charge lookup value for each canonical string
- Re-running the parser on harmonized_annotations.csv is idempotent (canonical form maps to itself)

## Limitations

- Ion identity naming in public spectral libraries is not harmonized; the lookup table must cover the adduct variants actually present or entries will fail to resolve
- The IIN edges text format is a generic comma-separated style with columns ID1/ID2 and no formal schema or validator tool, so malformed rows pass silently
- The MZmine IIMN module is described as a 'special IIMN build'; whether it ships in a released MZmine version is unclear, affecting reproducibility of the exported annotation strings

## Evidence

- [abstract] Parser collapses redundant adduct spellings into canonical ion identities: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [discussion] IIN annotation and edge files are exported by feature tools for GNPS: [evidence span withheld — anchor status: not_found]
- [readme] xcms-gnps-tools provides the IIN-format adduct annotation extractor: [evidence span withheld — anchor status: not_found]
- [discussion] MS1-based ion identity annotation precedes parsing: "Perform MS1-based feature grouping and ion identity annotation"
