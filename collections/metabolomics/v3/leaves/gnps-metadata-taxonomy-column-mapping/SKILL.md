---
name: gnps-metadata-taxonomy-column-mapping
description: Use when building the Inventa priority rank from a GNPS FBMN job and
  the metadata table must carry cleaned species/genus/family columns so LC can be
  computed from LOTUS taxonomic compound counts.
when_to_use_negative:
- The analysis does not involve any Literature Component (LC) scoring — e.g. running
  only the Feature Component (FC) from the GNPS job and quantification table, where
  taxonomic columns are not consulted
- Input is already a timaR-weighted annotation table, CANOPUS summary, or MEMO vectorized
  matrix — those consume the cleaned metadata but do not require re-mapping taxonomy
  columns
- The metadata table is not the GNPS standard format used for FBMN (e.g. ad-hoc Excel
  tables with different headers) — Inventa expects the GNPS format template linked
  in the Inventa README
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: Inventa
  role: Consumes the cleaned GNPS metadata table as `metadata_filename` and uses the
    mapped taxonomic columns to compute the Literature Component score against LOTUS
  repo: https://github.com/luigiquiros/inventa.git
- name: Open Tree of Life
  role: Taxonomy name-matching service used to clean species names to currently recognized
    taxa before mapping into the metadata table
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: GNPS
  role: Source of the standard metadata table format (the same one used for FBMN)
    whose mandatory headers (ATTRIBUTE_Species, ATTRIBUTE_Organe) must be populated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:37ebe393b954a48f8067d9d3fcf6c6a5c18a75c7f4ed10d2761e802fb2b76dd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa.git
  tools:
  - GNPS
  - Inventa
  - Open Tree of Life
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# GNPS metadata taxonomy column mapping

## Summary

Prepare the GNPS-format FBMN metadata table so that taxonomic columns (species/genus/family) are present, recognized, and correctly mapped before Inventa computes the Literature Component (LC) score from LOTUS compound-report counts.

## When to use

Use when building the Inventa priority rank from a GNPS FBMN job and the metadata table must carry cleaned species/genus/family columns so LC can be computed from LOTUS taxonomic compound counts.

## When NOT to use

- The analysis does not involve any Literature Component (LC) scoring — e.g. running only the Feature Component (FC) from the GNPS job and quantification table, where taxonomic columns are not consulted
- Input is already a timaR-weighted annotation table, CANOPUS summary, or MEMO vectorized matrix — those consume the cleaned metadata but do not require re-mapping taxonomy columns
- The metadata table is not the GNPS standard format used for FBMN (e.g. ad-hoc Excel tables with different headers) — Inventa expects the GNPS format template linked in the Inventa README

## Inputs

- GNPS-format FBMN metadata table (with sample rows and file mapping)
- Open Tree of Life taxonomy service / ot-tnrs output for name cleaning
- Source metadata columns (raw species/genus/family names, organ/part)

## Outputs

- Cleaned metadata table with mandatory ATTRIBUTE_Species and ATTRIBUTE_Organe headers (GNPS format)
- Per-sample taxonomy (species/genus/family) columns ready for LOTUS compound-count queries

## How to apply

Start from the GNPS metadata table used for the Feature-Based Molecular Networking job and add the mandatory headers ATTRIBUTE_Species and ATTRIBUTE_Organe (organ/part of the organism). Clean the species names to currently recognized taxa — the Inventa workflow prescribes using the Open Tree of Life taxonomy service for this step — because LC is computed by querying LOTUS for the number of compounds reported at species, genus, and family levels of each extract's taxon. These cleaned columns are what the LOTUS queries consume, so the LC score (e.g. 0.87 for Pristimera indica roots, which had 2 species-, 8 genus-, and 6,064 family-level reported compounds) is only as correct as the mapped taxon names. Drop or manually clean any other unneeded columns (identities, 'Unknown: number' headers at the end of the quantification table are handled automatically by quand_table(), but the metadata table itself must be minimal and correct). The metadata table with mapped taxonomy columns is then passed to Inventa as `metadata_filename`, alongside the MZmine quantification table and, optionally, the timaR-weighted annotations, CANOPUS summary, and MEMO vectorized dissimilarity matrix.

## Related tools

- **Inventa** (Consumes the cleaned GNPS metadata table as `metadata_filename` and uses the mapped taxonomic columns to compute the Literature Component score against LOTUS) — https://github.com/luigiquiros/inventa.git
- **Open Tree of Life** (Taxonomy name-matching service used to clean species names to currently recognized taxa before mapping into the metadata table) — https://opentree.readthedocs.io/en/latest/readme.html
- **GNPS** (Source of the standard metadata table format (the same one used for FBMN) whose mandatory headers (ATTRIBUTE_Species, ATTRIBUTE_Organe) must be populated)

## Examples

```
conda activate inventa; python src/inventa.py --metadata_filename data/metadata.tsv --quantitative_data_filename data/quant_table.csv --parameters_filename data/parameters.json  # metadata.tsv must carry cleaned ATTRIBUTE_Species / ATTRIBUTE_Organe
```

## Evaluation signals

- The cleaned metadata table retains the GNPS template columns and includes the mandatory `ATTRIBUTE_Species` and `ATTRIBUTE_Organe` headers
- Every species value maps to a currently accepted taxon in the Open Tree of Life TNRS output (no unresolved/ambiguous names left in the LC-relevant columns)
- The genus and family columns for each sample are consistent with the cleaned species (e.g. Pristimera indica resolves to Celastraceae family-level reporting in LOTUS)
- Running Inventa with this metadata yields per-sample LC values in the 0–1 range; for Pristimera indica roots the LC is 0.87 given 2 species-, 8 genus-, and 6,064 family-level reported compounds
- The metadata table is identical to the one used for the GNPS FBMN job (sample/file mapping unchanged), so row order and file names align with the MZmine quantification table

## Limitations

- Name cleaning relies on the Open Tree of Life service; taxa absent or ambiguous in that backbone may not map cleanly and will propagate into the LC score
- LOTUS compound counts depend on the user-defined maxima (max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500 in the article's proof of concept); changing these thresholds changes LC even if the metadata mapping is unchanged
- The LC score is independent of the spectral data — a perfectly mapped metadata table does not correct an extract set that is poorly studied at the species/genus/family level

## Evidence

- [readme] mandatory headers ATTRIBUTE_Species / ATTRIBUTE_Organe: "While creating the 'metadata' there are some MANDATORY headers:

- `ATTRIBUTE_Species` : The species should be cleaned to up-to-day recognized names, you can use the Open Tree of Life to do so"
- [readme] Open Tree of Life cleaning instruction: [evidence span withheld — anchor status: external_doc]
- [readme] Metadata table = GNPS format / same as FBMN: "Metadata table (the same used for the FBMN)

... `metadata_filename`: GNPS format"
- [readme] LC uses literature reports for the taxon of a given extract: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract."
- [other] LC = 0.87 for Pristimera indica with 2/8/6064 reports: [evidence span withheld — anchor status: not_found]
