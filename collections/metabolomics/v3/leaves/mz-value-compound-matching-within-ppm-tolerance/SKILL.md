---
name: mz-value-compound-matching-within-ppm-tolerance
description: Use when you have already produced positive/negative-mode m/z peak tables
  (e.g. XCMS or MetaboAnalyst export) and need putative MSI level 2 identities for
  individual m/z values within a set ppm tolerance.
when_to_use_negative:
- Input is raw spectra — MetaboShiny does not accept raw peak data and requires pre-processed
  peak lists
- You need definitive structural confirmation (MSI level 1) rather than MSI level
  2 putative annotation; matching only provides putative identities
- The m/z values of interest are already significant hits downstream of statistical
  analysis and only the enriched compound table needs updating — use the search panel
  on those hits instead of pre-matching the full peak list
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetaboShiny
  role: Interactive R/Shiny application that executes the ppm-window m/z search, filters
    matches by isotope/adduct, and exports the annotation table
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that builds the compound database storing m/z values for
    common isotopes and adducts (e.g. M + H) of every compound from over 30 supported
    sources
  repo: https://github.com/joannawolthuis/MetaDBparse
- name: HMDB
  role: Compound database searched to return putative identities such as creatine
    riboside for 264 m/z
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:7be45ff151a06ae60e4601a5ade2dfcca41b228880d8ef7fedb357434292557c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - HMDB
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mz-value-compound-matching-within-ppm-tolerance

## Summary

Match pre-processed peak-list m/z values against one or more compound databases within a user-defined ppm error margin, returning matched compound identities with their adduct assignments and isotope status (e.g. creatine riboside as an M + H hit at 264 m/z via HMDB).

## When to use

Use when you have already produced positive/negative-mode m/z peak tables (e.g. XCMS or MetaboAnalyst export) and need putative MSI level 2 identities for individual m/z values within a set ppm tolerance.

## When NOT to use

- Input is raw spectra — MetaboShiny does not accept raw peak data and requires pre-processed peak lists
- You need definitive structural confirmation (MSI level 1) rather than MSI level 2 putative annotation; matching only provides putative identities
- The m/z values of interest are already significant hits downstream of statistical analysis and only the enriched compound table needs updating — use the search panel on those hits instead of pre-matching the full peak list

## Inputs

- positive- and negative-mode m/z peak tables (MetaboAnalyst-like, MetaboShiny native, or MetaboLights format; XCMS/MSnbase output accepted)
- prebuilt compound database (SQLite) generated locally via MetaDBparse, e.g. from HMDB
- user-defined ppm error margin for the mass spectrometer
- adduct definitions and adduct rules tables (importable in the Settings/Adducts tab)

## Outputs

- annotation table with matched compound name, molecular formula, SMILES, database ID, adduct assignment (e.g. M + H), isotope status and dppm error column (exported as mz_annotation_matches.csv)
- compound-match list where hits with identical SMILES are collapsed to a single row while isomers (same formula, different SMILES) remain separate

## How to apply

First build the compound database locally with MetaDBparse, which stores adduct/isotope-derived m/z values (e.g. M + H) and SMILES for each compound from one of 35 supported databases such as HMDB. For each query m/z, compute the accepted mass window using the ppm_range helper at the error margin set when importing the peak files (step 2 of file import: 'Set the error margin of your mass spectrometer in parts per million (ppm)'). Query the SQLite-backed database for entries whose derived m/z falls inside that window, then filter matches to main-peak isotopes, sort by ppm error, and collapse hits sharing identical SMILES into single rows. Export the annotation table with adduct and dppm columns (e.g. mz_annotation_matches.csv) and inspect matched compound name, formula, SMILES and adduct assignment; validate by confirming that a known marker (e.g. 264 m/z) returns its expected identity (creatine riboside, M + H adduct).

## Related tools

- **MetaboShiny** (Interactive R/Shiny application that executes the ppm-window m/z search, filters matches by isotope/adduct, and exports the annotation table) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that builds the compound database storing m/z values for common isotopes and adducts (e.g. M + H) of every compound from over 30 supported sources) — https://github.com/joannawolthuis/MetaDBparse
- **HMDB** (Compound database searched to return putative identities such as creatine riboside for 264 m/z)

## Examples

```
docker run -p 8080:8080 -v ~/MetaboShiny/:/root/MetaboShiny/:cached --rm -it jcwolthuis/metaboshiny:latest Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"
```

## Evaluation signals

- Exported annotation table contains both an adduct column and a dppm (ppm error) column sorted by dppm after isotope filtering
- Searching a benchmark m/z against HMDB returns the known identity with its adduct: 264 m/z → creatine riboside (M + H)
- Hits with identical SMILES are collapsed into one row; compounds with the same formula but different SMILES remain as separate rows
- Every returned hit's derived m/z lies within the ppm window defined at file-import time (ppm_range helper)
- The database version number and download date are listed after building, confirming the database was freshly constructed via MetaDBparse

## Limitations

- Databases are not automatically updated — a database must be rebuilt (once per version) before changes take effect
- Custom databases require a specific schema (logo, base CSV with defined columns, database name + description) and an app restart before they become buildable
- MetaboShiny performs annotation only on m/z ratios and putative MSI level 2 identities; matches provide putative, not confirmed, identities

## Evidence

- [methods] a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit: "a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit"
- [intro] supporting over 30 databases and millions of m/z values with adducts and isotopes: "MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1) and is capable of searching through millions of m/z values, including metabolite variants with"
- [readme] Set the error margin of your mass spectrometer in parts per million (ppm).: "2. Set the error margin of your mass spectrometer in parts per million (ppm).
3. Set the m/z missing values percentage threshold."
- [readme] MetaboShiny does not accept raw peak data: "MetaboShiny does not accept raw peak data. We suggest using either XCMS (with the MetaboAnalyst export option) or another method of choice such as MSnbase."
- [supplementary] MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds: "MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds"
