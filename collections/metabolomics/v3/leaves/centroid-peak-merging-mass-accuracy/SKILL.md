---
name: centroid-peak-merging-mass-accuracy
description: Use when building an isotopic profile database (IPDB) from a molecular
  formula list so that only centroid-level theoretical peaks are queried against experimental
  MS1 LC/HRMS peaklists.
when_to_use_negative:
- The input is already experimental MS1 peaklist data (use IDSL.IPA peak detection
  rather than theoretical profile centroiding)
- The instrument cannot resolve the isotopologue spacing and no mass-accuracy window
  is specified, so centroid merging criteria are undefined
- Only MS2 spectra are available and no MS1 isotopic-profile matching is planned
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.UFA
  role: Computes theoretical isotopic profiles for molecular formulas and generates
    the centroid-level IPDB queried for MS1 formula annotation
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Produces the experimental peaklists that the centroided IPDB profiles are
    matched against
  repo: https://github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:0f4ce23d6704b5abd94b8bc93b12bcf3dd54f0fe8bc83051f3f0308228dc8924
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Centroid peak merging within mass-accuracy windows

## Summary

Generates centroid-level theoretical isotopic profiles for the IDSL.UFA IPDB by merging adjacent isotopologue m/z entries that fall within the instrument mass-accuracy window (e.g. 5 ppm for HRMS) into single representative m/z values with summed relative intensities.

## When to use

Use when building an isotopic profile database (IPDB) from a molecular formula list so that only centroid-level theoretical peaks are queried against experimental MS1 LC/HRMS peaklists.

## When NOT to use

- The input is already experimental MS1 peaklist data (use IDSL.IPA peak detection rather than theoretical profile centroiding)
- The instrument cannot resolve the isotopologue spacing and no mass-accuracy window is specified, so centroid merging criteria are undefined
- Only MS2 spectra are available and no MS1 isotopic-profile matching is planned

## Inputs

- Unique molecular formula list extracted from a compound source (e.g. RefMet, SDF, or formula list)
- Elemental isotope masses and natural abundances (IUPAC atomic mass tables / PubChem element data)
- Instrument mass-accuracy specification (e.g. ppm tolerance for the HRMS instrument)

## Outputs

- IPDB: per-formula centroid isotopic profile table with m/z, relative intensity, and molecular formula (stored as .Rdata, e.g. IPDB_v1.8)
- Normalized centroid profiles where the monoisotopic peak is the most intense entry and relative intensities sum to 1 within each formula

## How to apply

Enumerate all isotopologue combinations for each unique molecular formula using isotope masses and natural abundances (IDSL.UFA draws these from the IUPAC atomic mass tables / PubChem element references), computing both the monoisotopic m/z and all shifted isotopologue m/z values with their relative probabilities. Then merge adjacent theoretical isotopologue m/z entries that fall inside the instrument's mass-accuracy window — for Orbitrap HRMS data this is on the order of 5 ppm — into one centroid m/z value, summing their relative intensities so each merged centroid represents the combined abundance of the co-eluting isotopologues. Retain the monoisotopic peak as the most intense centroid and normalize the merged relative intensities within each formula's profile so they sum to 1. Assemble the per-formula centroid profiles into the IPDB keyed by molecular formula, which IDSL.UFA then queries against experimental MS1 peaks. The rationale is that unmerged isotopologue lists are finer-grained than the instrument can resolve, so centroiding reduces spurious candidate peaks while preserving the m/z and intensity pattern used for isotopic-profile matching and formula ranking.

## Related tools

- **IDSL.UFA** (Computes theoretical isotopic profiles for molecular formulas and generates the centroid-level IPDB queried for MS1 formula annotation) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Produces the experimental peaklists that the centroided IPDB profiles are matched against) — https://github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Each formula's centroid profile must contain the monoisotopic peak as the most intense entry
- Relative intensities must be normalized (sum to 1) within each formula's profile
- Number of formulas represented in the IPDB must match the count of unique formulas in the input compound list
- No two centroid m/z values within a formula's profile should fall inside the same mass-accuracy window (no unmerged adjacent centroids)

## Limitations

- Centroid granularity depends entirely on the chosen mass-accuracy window; too wide a window over-merges distinct isotopologues and blurs the theoretical pattern
- IPDB construction is database-dependent — formula sources such as RefMet determine which chemical spaces are covered, and compounds absent from those sources are missed
- Only elements/isotopes represented in the underlying isotope tables (e.g. IUPAC atomic mass tables) can be profiled

## Evidence

- [readme] IPDB generation via isotope masses and abundances: [evidence span withheld — anchor status: not_found]
- [readme] Theoretical isotope pattern prediction uses IUPAC atomic mass tables: [evidence span withheld — anchor status: not_found]
- [intro] Formula assignment by matching observed vs theoretical isotopic profiles: "molecular formula can be assigned to a LC/HRMS peak using the observed and theoretical isotopic profiles for a chemical compound"
- [results] IDSL.UFA computes theoretical isotopic profiles and matches against experimental MS1 data: "IDSL.UFA computes theoretical isotopic profiles for molecular formulas, matches theoretical isotopic profiles against experimental LC/HRMS data in individual data file"
- [readme] IDSL.UFA requires only MS1 for molecular formula annotation: [evidence span withheld — anchor status: not_found]
