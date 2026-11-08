---
name: sqlite-relational-querying
description: Use when the analysis input is a BiG-SLiCE output SQLite3 database (e.g.,
  from full_run_result.zip covering ~1.2M BGCs) and you need to aggregate BGC counts,
  GCF memberships, or class proportions per taxon.
when_to_use_negative:
- The data are already aggregated into a flat table/TSV — BiG-SLiCE v2 supports --export-csv
  to export pre-calculated BGCs and GCFs tables into TSVs, avoiding manual SQL joins.
- The analysis goal is interactive GCF querying of new BGCs — use BiG-SLiCE's --query
  mode instead of hand-written SQL against the clustering database.
- Only the web-based visualization is needed — run the bundled flask mini web-app
  rather than querying the database programmatically.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Produces the SQLite3 output database whose BGC, class, GCF, and taxonomy tables
    are queried; pre-processed ~1.2M BGC result is downloadable as full_run_result.zip
  repo: https://github.com/medema-group/bigslice
- name: sqlite3 (Python module)
  role: Used to open the BiG-SLiCE output database and run SQL joins between BGC/class
    tables and genome taxonomy tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:3743efe8a8b19d344c68c6f91278bd8c3364f59c9a21e358a368e195777dae0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - sqlite3 (Python module)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# SQLite relational querying

## Summary

Load BiG-SLiCE's output SQLite3 database and run SQL joins between the BGC/class tables and genome taxonomy tables (GTDB-derived) to aggregate BGC counts and GCF assignments per taxonomic group for downstream visualization.

## When to use

Use when the analysis input is a BiG-SLiCE output SQLite3 database (e.g., from full_run_result.zip covering ~1.2M BGCs) and you need to aggregate BGC counts, GCF memberships, or class proportions per taxon.

## When NOT to use

- The data are already aggregated into a flat table/TSV — BiG-SLiCE v2 supports --export-csv to export pre-calculated BGCs and GCFs tables into TSVs, avoiding manual SQL joins.
- The analysis goal is interactive GCF querying of new BGCs — use BiG-SLiCE's --query mode instead of hand-written SQL against the clustering database.
- Only the web-based visualization is needed — run the bundled flask mini web-app rather than querying the database programmatically.

## Inputs

- BiG-SLiCE output SQLite3 database file (e.g., from the pre-processed full_run_result.zip covering ~1.2M microbial BGCs)
- Genome taxonomy tables derived from GTDB version 04-RS89 (via GTDB API or GTDB toolkit inference)
- GCF centroid features at a chosen threshold (e.g., T=900 with 29,955 GCF models)
- MIBiG v2.0 reference BGC feature distances for novelty flagging

## Outputs

- Aggregated BGC counts and GCF assignments per taxonomic group
- Per-taxon BGC class proportion table
- Set of taxa with GCFs distant from closest MIBiG v2.0 features (uncharted biosynthetic potential)
- Rendered global map of secondary metabolic diversity across taxonomy (e.g., bgc_class_diversity_map.png)

## How to apply

Open the BiG-SLiCE output .sqlite3 file using the sqlite3 Python module, since BiG-SLiCE stores all processed input data in this relational form rather than tabular flat files. Run SQL joins between the BGC/class tables and the genome taxonomy tables (GTDB version 04-RS89) to aggregate BGC counts and GCF assignments per taxonomic group, then compute per-taxon class proportions in pandas from the joined rows. For the global diversity map, further flag taxa whose GCF models lie far from the closest MIBiG v2.0 reference BGC features, indicating uncharted biosynthetic potential. The rationale is that once familiar with SQL queries, the database provides a powerful data-wrangling experience for postprocessing, mirroring the jupyter notebook scripts the authors released for generating manuscript figures. Validate by reconciling aggregated BGC counts against the reported totals of 1,225,071 BGCs from 209,206 genomes and MAGs (with 29,955 GCF models at T=900) before rendering outputs such as bgc_class_diversity_map.png.

## Related tools

- **BiG-SLiCE** (Produces the SQLite3 output database whose BGC, class, GCF, and taxonomy tables are queried; pre-processed ~1.2M BGC result is downloadable as full_run_result.zip) — https://github.com/medema-group/bigslice
- **sqlite3 (Python module)** (Used to open the BiG-SLiCE output database and run SQL joins between BGC/class tables and genome taxonomy tables)

## Examples

```
import sqlite3; con = sqlite3.connect('full_run_result/result/data6.db'); rows = con.execute('SELECT t.*, c.class FROM bgcs b JOIN classes c ON b.id = c.bgc_id JOIN genomes g ON b.genome_id = g.id JOIN taxon_names t ON g.taxon_id = t.taxon_id').fetchall()
```

## Evaluation signals

- Aggregated BGC counts reconcile to the reported totals: 1,225,071 BGCs from 209,206 genomes and MAGs
- GCF model count matches the selected threshold run, e.g., 29,955 GCF models at T=900
- Joins across BGC/class and taxonomy tables return non-null taxonomy for genomes annotated via GTDB API version 04-RS89 or GTDB toolkit marker-based inference
- Class proportion vectors sum to 1 per taxon, and fungal BGCs separate into distinct clades (a1–a4) from the mostly bacterial datasets in the rendered phylogram
- Rendered figure matches the author-released jupyter notebook scripts' output style (e.g., bgc_class_diversity_map.png)

## Limitations

- The learning curve for SQL queries is steeper than conventional tabular output files, per the README's note for advanced users.
- The pre-processed ~1.2M BGC result (full_run_result.zip, 17GB) was generated with BiG-SLiCE v1; there is currently no pre-processed result for BiG-SLiCE v2.
- Taxonomy coverage depends on GTDB: bacterial and archaeal accessions were queried through the GTDB API, while remaining genomes needed GTDB toolkit marker-based inference.

## Evidence

- [results] Load the BiG-SLiCE output SQLite3 database using the sqlite3 Python module and join BGC/class tables with GTDB taxonomy: [evidence span withheld — anchor status: not_found]
- [readme] README describes programmatic access via SQL queries against the output SQLite database: "To access BiG-SLiCE's preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]"
- [results] GTDB version 04-RS89 used to fetch taxonomy for bacterial and archaeal genomes: "We queried all bacterial and archaeal NCBI genome accessions through the GTDB API (version 04-RS89) to fetch their taxonomy information"
- [results] Reported totals used for validation of aggregated counts: "In total, a final count of 1,225,071 BGCs were predicted from 209,206 genomes and MAGs, as reported in Table 1"
- [results] GCF model count at T=900 used in downstream MIBiG distance analysis: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
