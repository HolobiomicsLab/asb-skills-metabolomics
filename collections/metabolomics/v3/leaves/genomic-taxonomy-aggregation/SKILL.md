---
name: genomic-taxonomy-aggregation
description: Use when you have a BiG-SLiCE output SQLite database plus genome taxonomy
  annotations (e.g., GTDB 04-RS89) and need per-taxon BGC class proportions or GCF
  diversity summaries.
when_to_use_negative:
- Input is raw genome FASTA or antiSMASH output not yet clustered — run BiG-SLiCE
  clustering first to obtain GCF assignments
- Taxonomy is missing or unreliable for the genomes — GTDB-based joins require valid
  GTDB assignments; the paper used the GTDB toolkit only when GTDB API lookup failed
- You need pairwise BGC similarity networks rather than taxon-level aggregation —
  BiG-SLiCE's Euclidean-space clustering output is not a substitute for BiG-SCAPE-style
  network analyses
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Produces the SQLite3 output database whose BGC/class and GCF tables are aggregated
    per taxonomic group; also supplies pre-processed full-run results
  repo: https://github.com/medema-group/bigslice
- name: GTDB (Genome Taxonomy Database)
  role: Source of taxonomic assignments (version 04-RS89) joined against BGC/GCF tables;
    GTDB toolkit used for genomes not covered by the API
- name: pyhmmer
  role: HMM scanning dependency used within BiG-SLiCE v2 for domain detection underlying
    the clustered BGC features
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:98d0a2970a8f9c7ac4af24475ef6a82fd9c7af64ff3c9c48a8a5cb9e2e5c17e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - GTDB (Genome Taxonomy Database)
  - pyhmmer
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genomic-taxonomy-aggregation

## Summary

Aggregate BiG-SLiCE clustering results (BGC counts and GCF assignments) per taxonomic group by joining the output SQLite3 database's BGC/class tables with GTDB-derived genome taxonomy tables, enabling global maps of secondary metabolic diversity across taxa.

## When to use

Use when you have a BiG-SLiCE output SQLite database plus genome taxonomy annotations (e.g., GTDB 04-RS89) and need per-taxon BGC class proportions or GCF diversity summaries.

## When NOT to use

- Input is raw genome FASTA or antiSMASH output not yet clustered — run BiG-SLiCE clustering first to obtain GCF assignments
- Taxonomy is missing or unreliable for the genomes — GTDB-based joins require valid GTDB assignments; the paper used the GTDB toolkit only when GTDB API lookup failed
- You need pairwise BGC similarity networks rather than taxon-level aggregation — BiG-SLiCE's Euclidean-space clustering output is not a substitute for BiG-SCAPE-style network analyses

## Inputs

- BiG-SLiCE output SQLite3 database file (e.g., full_run_result.zip covering ~1.2M microbial BGCs)
- Genome taxonomy table (GTDB API version 04-RS89 assignments for bacterial and archaeal genomes)
- MIBiG v2.0 reference BGC features (for novelty/distance flagging)

## Outputs

- Per-taxon aggregated BGC counts and GCF assignment table
- Per-taxon BGC class proportion summary
- Global map/phylogram of secondary metabolic diversity across taxonomy (e.g., bgc_class_diversity_map.png)

## How to apply

Load the BiG-SLiCE output SQLite3 database (e.g., from the pre-processed full_run_result.zip covering ~1.2M microbial BGCs) with the sqlite3 Python module. Run SQL joins between the BGC/class tables and the genome taxonomy tables (GTDB-derived, version 04-RS89) to aggregate BGC counts and GCF assignments per taxonomic group. Compute per-taxon class proportions and, optionally, flag taxa whose GCF models lie far from the closest MIBiG v2.0 reference BGC features to indicate uncharted biosynthetic potential. Validate the aggregation by reconciling counts against the reported totals of 1,225,071 BGCs from 209,206 genomes/MAGs with 29,955 GCF models at T=900, then render the result (e.g., a phylogram or class-diversity map) using the authors' released jupyter notebook scripts as a style reference.

## Related tools

- **BiG-SLiCE** (Produces the SQLite3 output database whose BGC/class and GCF tables are aggregated per taxonomic group; also supplies pre-processed full-run results) — https://github.com/medema-group/bigslice
- **GTDB (Genome Taxonomy Database)** (Source of taxonomic assignments (version 04-RS89) joined against BGC/GCF tables; GTDB toolkit used for genomes not covered by the API)
- **pyhmmer** (HMM scanning dependency used within BiG-SLiCE v2 for domain detection underlying the clustered BGC features) — https://github.com/althonos/pyhmmer

## Examples

```
import sqlite3; con = sqlite3.connect('full_run_result/data.db'); rows = con.execute("SELECT t.rank, b.class, COUNT(*) FROM bgcs b JOIN taxa t ON b.genome_id = t.genome_id GROUP BY t.rank, b.class").fetchall()
```

## Evaluation signals

- Reconciled BGC count total matches the reported 1,225,071 BGCs from 209,206 genomes and MAGs
- GCF model count at T=900 matches the reported 29,955 GCF models
- SQL joins return non-empty per-taxon rows for all taxonomic groups with BGCs, with no orphan genome IDs after the GTDB join
- Per-taxon class proportions sum to 1 (or 100%) within each taxon
- Rendered figure (e.g., bgc_class_diversity_map.png) reproduces the expected separation of fungal BGC clades (a1–a4) from bacterial datasets

## Limitations

- The pre-processed full_run_result.zip corresponds to BiG-SLiCE v1 results; there is currently no pre-processed result for BiG-SLiCE v2 (whose clustering uses cosine-like l2-normalized distances and PFAM 35.0), so aggregations from v2 runs are not directly comparable to the published global map
- Taxonomy for bacterial/archaeal genomes came from the GTDB API version 04-RS89, and remaining genomes were inferred from marker composition with the GTDB toolkit, so taxon-level aggregates inherit those assignment uncertainties
- The SQLite database requires SQL fluency ('the learning curve might be steeper compared to the conventional tabular-formatted output files')

## Evidence

- [results] Load the BiG-SLiCE output SQLite3 database and join taxonomy: "To access BiG-SLiCE's preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]"
- [results] GTDB taxonomy source: "We queried all bacterial and archaeal NCBI genome accessions through the GTDB API (version 04-RS89) to fetch their taxonomy information"
- [results] Totals to reconcile against: "In total, a final count of 1,225,071 BGCs were predicted from 209,206 genomes and MAGs, as reported in Table 1"
- [results] Clustering thresholds used: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [results] Novelty flagging against MIBiG: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [readme] Availability of pre-processed full-run result: "there is currently no pre-processed result for BiG-SLiCE v2, we will work to make it available soon."
