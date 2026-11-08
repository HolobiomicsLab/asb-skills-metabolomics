---
name: gnps-metadata-sample-mapping
description: Use when running Inventa or GNPS Feature-Based Molecular Networking on
  an extract set and you must supply a metadata table with cleaned, up-to-date species
  names and organ information per sample.
when_to_use_negative:
- Input is already the MZmine quantitative table (Peak area / row m/z / row retention
  time) — that is a different Inventa input, not the metadata mapping.
- The analysis does not need taxon-based scores (e.g. only the Feature Component or
  Similarity Component is of interest) and no sample-to-taxon mapping is required.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: Inventa
  role: Consumes the GNPS-format metadata table via metadata_filename to map extracts
    to species/organ for taxon-based scoring
  repo: https://github.com/luigiquiros/inventa
- name: Open Tree of Life
  role: Used to clean species names to up-to-date recognized names before populating
    ATTRIBUTE_Species
- name: GNPS
  role: Defines the preferred metadata table standard format that the mapping must
    follow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:1fb0973cecd4f7a567b5360181ae150e64296e4e912feab9fa6e20fde0cbc9dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - Open Tree of Life
  - scikit-bio
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# GNPS metadata sample mapping

## Summary

Prepares and validates the mandatory GNPS-format metadata table that maps each extract to cleaned species (ATTRIBUTE_Species) and organ (ATTRIBUTE_Organe), enabling Inventa to compute taxon-based components such as the Literature Component.

## When to use

Use when running Inventa or GNPS Feature-Based Molecular Networking on an extract set and you must supply a metadata table with cleaned, up-to-date species names and organ information per sample.

## When NOT to use

- Input is already the MZmine quantitative table (Peak area / row m/z / row retention time) — that is a different Inventa input, not the metadata mapping.
- The analysis does not need taxon-based scores (e.g. only the Feature Component or Similarity Component is of interest) and no sample-to-taxon mapping is required.

## Inputs

- Metadata table in GNPS standard format with ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- Open Tree of Life taxonomy reference (https://opentree.readthedocs.io/en/latest/readme.html) for name cleaning
- Sample/extract list matching the FBMN job

## Outputs

- Validated metadata_filename consumable by Inventa and FBMN
- Consistent sample identifiers across metadata, quantitative table, and MEMO matrix

## How to apply

Build the metadata table in the GNPS standard format (the Google-Sheets template referenced by the Inventa README), including the mandatory headers ATTRIBUTE_Species and ATTRIBUTE_Organe for every sample. Use the Open Tree of Life to clean species names to currently recognized taxonomy before importing, since LC and CC scores depend on species/genus/family taxon assignment. Use the same metadata table for FBMN and for Inventa so sample identifiers remain consistent across the quantification table, annotations, and MEMO matrix. Drop the table into the /data folder and point the metadata_filename parameter at it; Inventa then matches extract rows to the quantitative table for the Literature Component calculation.

## Related tools

- **Inventa** (Consumes the GNPS-format metadata table via metadata_filename to map extracts to species/organ for taxon-based scoring) — https://github.com/luigiquiros/inventa
- **Open Tree of Life** (Used to clean species names to up-to-date recognized names before populating ATTRIBUTE_Species)
- **GNPS** (Defines the preferred metadata table standard format that the mapping must follow)

## Examples

```
git clone https://github.com/luigiquiros/inventa.git; conda env create -n inventa -f inventa/environment.yml; conda activate inventa; then set metadata_filename in the notebook to your GNPS-format table dropped in /data
```

## Evaluation signals

- The metadata table contains the mandatory headers ATTRIBUTE_Species and ATTRIBUTE_Organe for every sample
- Species names in ATTRIBUTE_Species are recognized taxonomy names (cleaned via Open Tree of Life), not synonyms or misspellings
- Sample identifiers in the metadata match those in the quantitative table, annotations files, and MEMO matrix so rows join correctly
- The table follows the GNPS standard format template so FBMN and Inventa both accept it without reformatting
- Taxon labels (species, genus, family) derived from the metadata are consistent with the LC/CC scoring output (e.g. LC value of 1 indicates no reported compounds for the considered taxon)

## Limitations

- Species names must be cleaned to up-to-date recognized names manually or via Open Tree of Life; stale synonyms will misdirect the Literature Component
- Only the specified mandatory headers are validated by Inventa — additional columns must be formatted per GNPS conventions or may be ignored
- The same metadata table is used for FBMN and Inventa, so changes must propagate to both to keep sample mapping consistent

## Evidence

- [readme] The standard format from GNPS is preferred: "`metadata_filename`: GNPS format ([https://docs.google.com/spreadsheets/d/1pSrqOdmMVBhVGpxIZeglToxihymTuaR4_sqTbLBlgOA/edit#gid=0]"
- [readme] Mandatory headers for species and organ: [evidence span withheld — anchor status: external_doc]
- [readme] Open Tree of Life for name cleaning: [evidence span withheld — anchor status: external_doc]
- [readme] Organe column definition: [evidence span withheld — anchor status: external_doc]
- [readme] Metadata used for both FBMN and Inventa: "- Metadata table (the same used for the FBMN)"
