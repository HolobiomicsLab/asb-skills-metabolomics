---
name: kegg-compound-pathway-mapping
description: Use when you have an annotation matrix pairing peak IDs with KEGG (Cxxxxx)
  or ChEBI IDs and need to build peak-to-pathway membership tables for metabolite-set
  activity scoring with PALS.
when_to_use_negative:
- All peak features are unannotated — peaks that do not have annotations will not
  be used for pathway analysis, so no peak-to-pathway mapping can be built
- The dataset is proteomic or transcriptomic and maps via UniProt/ENSEMBL IDs rather
  than metabolite compounds (KEGG/ChEBI); use the protein/gene Reactome database route
  instead
- Feature-to-pathway membership tables already exist and only the activity-level decomposition
  (PLAGE scoring) step is needed
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Performs the database queries of pathways and builds the peak-to-metabolite-set
    mapping used for pathway activity level scoring
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads the annotation DataFrame pairing peak IDs with KEGG/ChEBI entity IDs
    before database lookup
- name: KEGG
  role: Compound database queried via the PiMP_KEGG export for metabolite pathway
    membership
- name: Reactome
  role: Pathway database queried online via a local Neo4j server or offline from downloaded
    pathway files (COMPOUND/ChEBI modes)
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:29444166e5b99c963a5f19dfbb38028ccdce922f18c80e226cb335308b7794ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - PALS
  - Python
  - Reactome
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kegg-compound-pathway-mapping

## Summary

Maps peak features with KEGG or ChEBI compound annotations onto metabolite sets (pathways) via the PiMP-exported KEGG compound database (PiMP_KEGG) or a Reactome database instance on Neo4j, forming the feature-to-set membership tables that PALS scores with PLAGE/ORA/GSEA.

## When to use

Use when you have an annotation matrix pairing peak IDs with KEGG (Cxxxxx) or ChEBI IDs and need to build peak-to-pathway membership tables for metabolite-set activity scoring with PALS.

## When NOT to use

- All peak features are unannotated — peaks that do not have annotations will not be used for pathway analysis, so no peak-to-pathway mapping can be built
- The dataset is proteomic or transcriptomic and maps via UniProt/ENSEMBL IDs rather than metabolite compounds (KEGG/ChEBI); use the protein/gene Reactome database route instead
- Feature-to-pathway membership tables already exist and only the activity-level decomposition (PLAGE scoring) step is needed

## Inputs

- Annotation matrix CSV/DataFrame with two columns: peak ID (row_id, e.g. 3033929) and assigned compound ID (entity_id, KEGG or ChEBI)
- Peak intensity matrix CSV/DataFrame (rows = peak features, columns = samples, optional second-line group labels)
- Experimental design dictionary with groups and case/control comparisons (programmatic use)
- Pathway database choice: PiMP_KEGG, COMPOUND, or ChEBI; species name for Reactome queries

## Outputs

- Feature-to-set mapping tables linking each annotated peak ID to pathway IDs (KEGG or Reactome) and, optionally, Molecular Family and Mass2Motif IDs
- Pathway-level candidate sets consumed downstream by PLAGE/ORA/GSEA ranking

## How to apply

Load the annotation matrix with pandas: the row index (or first column) is the peak ID and the second column (entity_id) holds the assigned KEGG or ChEBI compound ID; peaks lacking annotations are excluded from pathway analysis. Choose the database consistent with the annotation ID type using --db: PiMP_KEGG (KEGG compounds exported from PiMP) for KEGG IDs, or COMPOUND/ChEBI for Reactome queries with --species (default 'Homo sapiens'); note PiMP_KEGG, COMPOUND and ChEBI are for metabolomics, while UniProt and ENSEMBL serve proteomics/transcriptomics. For Reactome, decide between online mode (--connect_to_reactome_server against a local Neo4j server for the most updated pathways, optionally --use_all_reactome_pathways) and offline mode using downloaded Reactome pathway files (metabolic pathways only). Query the chosen database for set membership and assemble feature-to-set mapping tables linking each annotated peak ID to its pathway IDs; optionally extend the set space with GNPS Molecular Families and MS2LDA Mass2Motifs per the GNPS_analysis.ipynb workflow.

## Related tools

- **PALS** (Performs the database queries of pathways and builds the peak-to-metabolite-set mapping used for pathway activity level scoring) — github.com/glasgowcompbio/PALS
- **pandas** (Loads the annotation DataFrame pairing peak IDs with KEGG/ChEBI entity IDs before database lookup)
- **KEGG** (Compound database queried via the PiMP_KEGG export for metabolite pathway membership)
- **Reactome** (Pathway database queried online via a local Neo4j server or offline from downloaded pathway files (COMPOUND/ChEBI modes))

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control
```

## Evaluation signals

- The --db value matches the annotation ID type: PiMP_KEGG for KEGG compound IDs, ChEBI/COMPOUND for ChEBI-matched Reactome queries
- Mapping tables cover only peak IDs present in the annotation matrix; unannotated peaks are excluded from pathway sets
- All set IDs in the mapping draw from the selected database (KEGG pathway/compound IDs or Reactome pathway IDs) or from GNPS/MS2LDA groupings
- Multiple peak IDs may legitimately map to multiple compound IDs and vice versa — duplicate peak-to-entity rows (e.g. peak 3033931 mapping to C00183, C00719, C00583) are expected, not errors
- Output pathway ranking table contains p-value, unq_pw_F, tot_ds_F and F_coverage columns, confirming mapped sets reached the scoring stage

## Limitations

- In offline Reactome mode only metabolic pathways are available, and only for a pre-defined set of common species
- Peak identification uncertainty means multiple peak IDs may map to multiple compound IDs and vice versa, inflating or blurring set membership
- Peaks without compound annotations contribute nothing to pathway activity scores, so heavily unannotated feature lists under-represent pathways

## Evidence

- [readme] peaks that do not have annotations will not be used for pathway analysis: "users also provide a list of compound annotations assigned to peak features (peaks that do not have 
annotations will not be used for pathway analysis)"
- [readme] PiMP_KEGG, COMPOUND and ChEBI are for metabolomics use: "Note that *PiMP_KEGG*, *COMPOUND* and *ChEBI* are for 
metabolomics use, while *UniProt* and *ENSEMBL* are for 
proteomics and transcriptomics use respectively."
- [methods] PALS queries KEGG or Reactome for pathway mapping: [evidence span withheld — anchor status: not_found]
- [methods] PiMP_KEGG database definition: "PiMP_KEGG: KEGG compound database exported from PiMP."
- [readme] Reactome online query option: "PALS can connect to a Reactome database instance to retrieve the most updated pathways"
