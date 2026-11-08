---
name: reactome-neo4j-pathway-query
description: Use when annotated peak features carry ChEBI (or KEGG COMPOUND) IDs and
  metabolite sets must be drawn from Reactome pathways for Homo sapiens or other supported
  species, rather than KEGG.
when_to_use_negative:
- Compound annotations are KEGG IDs and KEGG pathway sets suffice — use the PiMP_KEGG
  offline database instead of standing up a Neo4j Reactome instance
- The metabolite sets of interest are GNPS Molecular Families or MS2LDA Mass2Motifs
  rather than curated pathways — retrieve those from GNPS/MS2LDA outputs instead
- Peaks lack ChEBI/KEGG annotations entirely — unannotated peaks are excluded from
  pathway analysis, so Reactome querying cannot help
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Executes the Reactome pathway retrieval (--db COMPOUND/ChEBI, --connect_to_reactome_server,
    offline downloaded files) and builds the pathway mapping for activity-level scoring
  repo: https://github.com/glasgowcompbio/PALS
- name: Reactome
  role: Pathway database queried online via a Neo4j server instance or offline from
    downloaded pathway files
- name: ChEBI
  role: Compound identifier scheme used to match peak annotations to the Reactome
    compound database (--db ChEBI)
- name: Neo4j
  role: Graph database hosting the local Reactome instance contacted in online mode
- name: pandas
  role: Stores the annotation and intensity DataFrames passed programmatically into
    DataSource
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:6573e8fb3c4efbcab14a09e2c2cd9fe8073b66528ed9db3b065346f2daeb66a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - ChEBI
  - Neo4j
  - PALS
  - Python
  - Reactome
  - pandas
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# reactome-neo4j-pathway-query

## Summary

Retrieves Reactome pathway membership for annotated MS peaks by querying a local Neo4j-hosted Reactome database (online mode) or pre-downloaded Reactome pathway files (offline mode), building the feature-to-pathway mapping tables that PALS uses for mPLAGE/ORA/GSEA scoring.

## When to use

Use when annotated peak features carry ChEBI (or KEGG COMPOUND) IDs and metabolite sets must be drawn from Reactome pathways for Homo sapiens or other supported species, rather than KEGG.

## When NOT to use

- Compound annotations are KEGG IDs and KEGG pathway sets suffice — use the PiMP_KEGG offline database instead of standing up a Neo4j Reactome instance
- The metabolite sets of interest are GNPS Molecular Families or MS2LDA Mass2Motifs rather than curated pathways — retrieve those from GNPS/MS2LDA outputs instead
- Peaks lack ChEBI/KEGG annotations entirely — unannotated peaks are excluded from pathway analysis, so Reactome querying cannot help

## Inputs

- Annotation CSV/DataFrame mapping peak IDs (row_id) to ChEBI compound IDs (entity_id)
- Intensity CSV/DataFrame with peak features as rows, samples as columns, and optional group line
- Experimental design dictionary or --comparisons spec (e.g. Stage_1/Control Stage_2/Control)
- Local Neo4j server hosting the Reactome database (online mode) or downloaded Reactome pathway files (offline mode)

## Outputs

- Feature-to-pathway mapping tables linking each annotated peak ID to Reactome pathway IDs
- Pathway ranking table with p-value columns, unq_pw_F, tot_ds_F, and F_coverage per pathway

## How to apply

Load the annotation matrix with pandas where the row index is the peak ID and the second column is the assigned compound annotation as a ChEBI ID (Peaks without annotations are excluded from pathway analysis). Choose --db ChEBI (Reactome compound matching by ChEBI IDs; COMPOUND matches by KEGG IDs) and set --species, e.g. "Homo sapiens" (default). Decide the retrieval mode: pass --connect_to_reactome_server to query a local Neo4j server hosting the Reactome database for the most updated pathways (optionally add --use_all_reactome_pathways to include non-metabolic pathways), or omit the flag to run offline mode against the downloaded Reactome pathway files, which cover metabolic pathways only. Feed the resulting peak-to-pathway mapping, together with the intensity matrix (after min_replace imputation, default 5000, log-2 transformation and SciPy standardisation), into the chosen ranking method (PLAGE, ORA, or GSEA).

## Related tools

- **PALS** (Executes the Reactome pathway retrieval (--db COMPOUND/ChEBI, --connect_to_reactome_server, offline downloaded files) and builds the pathway mapping for activity-level scoring) — https://github.com/glasgowcompbio/PALS
- **Reactome** (Pathway database queried online via a Neo4j server instance or offline from downloaded pathway files)
- **ChEBI** (Compound identifier scheme used to match peak annotations to the Reactome compound database (--db ChEBI))
- **Neo4j** (Graph database hosting the local Reactome instance contacted in online mode)
- **pandas** (Stores the annotation and intensity DataFrames passed programmatically into DataSource)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db COMPOUND --comparisons Stage_1/Control Stage_2/Control --min_replace 5000 --species "Homo sapiens" --connect_to_reactome_server
```

## Evaluation signals

- Every peak ID appearing in the mapping tables is present in the annotation matrix; peaks without annotations are excluded
- All retrieved set IDs are valid Reactome pathway IDs for the chosen database option and species
- Offline mode yields metabolic pathways only; adding --use_all_reactome_pathways in online mode expands beyond metabolic pathways
- Output ranking table contains the expected columns: p-value, unq_pw_F (unique formulae in pathway), tot_ds_F (formula hits in dataset), and F_coverage = tot_ds_F/unq_pw_F
- Offline-mode COMPOUND run on the HAT example data reproduces the documented command output without needing a Neo4j server

## Limitations

- Offline mode only covers metabolic pathways for a pre-packaged set of common species; querying all pathways requires an online Neo4j Reactome instance
- Annotations must be KEGG or ChEBI IDs; UniProt/ENSEMBL db options serve proteomics/transcriptomics, not metabolomics
- Multiple peak IDs may map to multiple compound IDs and vice versa due to identification uncertainty; unannotated peaks are silently dropped from pathway analysis

## Evidence

- [other] PALS obtains metabolites, pathways, and their mapping by querying a local copy of the KEGG database or by querying a Reactome instance running on top of a local Neo4j graph database.: "querying a Reactome instance running on top of a local Neo4j graph database"
- [readme] README on Reactome database options and species handling.: "*ChEBI*: Reactome compound database matching by ChEBI ids."
- [readme] README on online vs offline Reactome retrieval modes.: "--connect_to_reactome_server: Whether to connect to an instance of Neo4j server hosting Reactome database (online mode)."
- [readme] README on offline mode scope.: "Note that only metabolic pathways are available in this mode."
- [methods] Article methods on excluding unannotated peaks.: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
- [methods] Article methods on updated-pathway retrieval via server connection.: "PALS can connect to a Reactome database instance to retrieve the most updated pathways"
