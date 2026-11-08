---
name: biosynthetic-diversity-global-map-visualization
description: Use when you have a completed BiG-SLiCE clustering run (SQLite3 output,
  e.g. the pre-processed full_run_result.zip of ~1.2M microbial BGCs) and want to
  chart BGC class distribution and GCF novelty across GTDB taxonomy.
when_to_use_negative:
- You need to cluster a small BGC set with network-based pairwise comparisons — use
  BiG-SCAPE instead; BiG-SLiCE's non-pairwise Euclidean approach targets massive datasets
- You only want to place a newly sequenced BGC into existing GCFs — use BiG-SLiCE's
  --query mode rather than re-running the global map analysis
- You have raw genomes but no antiSMASH-predicted BGCs or BiG-SLiCE output database
  yet — the map is a post-processing step, not a BGC prediction pipeline
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Produces the SQLite3 output database of BGCs, GCF models, and centroid features
    that is queried with SQL and post-processed into the diversity map; also ships
    a flask web-app visualization engine
  repo: github.com/medema-group/bigslice
- name: antiSMASH v5.1.1
  role: Supplies the predicted BGCs and their class definitions that are aggregated
    per taxon in the map
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:14b990d8388fade97947bc7bae8c662e82b561bc164de9ead364de27e508c63b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SLiCE
  - antiSMASH v5.1.1
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# biosynthetic-diversity-global-map-visualization

## Summary

Builds a global map of secondary metabolic diversity across taxonomy by post-processing a BiG-SLiCE output SQLite3 database: aggregating BGC counts, GCF assignments, and BGC class proportions per taxon, then rendering the result as a phylogram-style figure that exposes uncharted biosynthetic potential.

## When to use

Use when you have a completed BiG-SLiCE clustering run (SQLite3 output, e.g. the pre-processed full_run_result.zip of ~1.2M microbial BGCs) and want to chart BGC class distribution and GCF novelty across GTDB taxonomy.

## When NOT to use

- You need to cluster a small BGC set with network-based pairwise comparisons — use BiG-SCAPE instead; BiG-SLiCE's non-pairwise Euclidean approach targets massive datasets
- You only want to place a newly sequenced BGC into existing GCFs — use BiG-SLiCE's --query mode rather than re-running the global map analysis
- You have raw genomes but no antiSMASH-predicted BGCs or BiG-SLiCE output database yet — the map is a post-processing step, not a BGC prediction pipeline

## Inputs

- BiG-SLiCE output folder containing the SQLite3 database (e.g. full_run_result.zip covering ~1.2M microbial BGCs)
- GTDB taxonomy annotations (version 04-RS89) joined to genome accessions
- MIBiG v2.0 reference BGC feature vectors (1,910 entries)
- Optionally the authors' jupyter notebook scripts from the GigaScience GigaDB repository

## Outputs

- Global map of secondary metabolic diversity across taxonomy (bgc_class_diversity_map.png)
- Per-taxon tables of aggregated BGC counts, GCF assignments, and BGC class proportions
- Histogram/distance metrics of GCF models vs. closest MIBiG v2.0 features flagging uncharted biosynthetic potential
- Agglomerative-clustering phylogram of GCF centroid features

## How to apply

Load the BiG-SLiCE output SQLite3 database with Python's sqlite3 module, then run SQL joins between the BGC/class tables and the GTDB-derived (version 04-RS89) genome taxonomy tables to aggregate BGC counts and GCF assignments per taxonomic group. Compute per-taxon BGC class proportions and take the GCF centroid features (e.g. the 29,955 GCF models at T=900), partition them into 500 bins with K-Means, and cluster the centroids with average-linkage agglomerative clustering (scikit-learn, Euclidean distance) to build the phylogram. For each taxon, measure the distance of every GCF model to its closest MIBiG v2.0 reference BGC features; taxa whose GCFs lie far from MIBiG indicate uncharted biosynthetic potential. Render the aggregated distribution as the global diversity map, mirroring the authors' released jupyter notebook scripts from GigaDB.

## Related tools

- **BiG-SLiCE** (Produces the SQLite3 output database of BGCs, GCF models, and centroid features that is queried with SQL and post-processed into the diversity map; also ships a flask web-app visualization engine) — github.com/medema-group/bigslice
- **antiSMASH v5.1.1** (Supplies the predicted BGCs and their class definitions that are aggregated per taxon in the map)

## Examples

```
import sqlite3; con = sqlite3.connect('full_run_result/data.sqlite'); rows = con.execute('SELECT ... FROM bgc JOIN taxonomy ...').fetchall()
```

## Evaluation signals

- Reconcile aggregated BGC counts against the reported totals: 1,225,071 BGCs from 209,206 genomes and MAGs, with 29,955 GCF models at T=900
- Check that the reconstructed phylogram shows fungal BGC clades (a1–a4) distinctly separated from the mostly bacterial datasets
- Verify centroid features were partitioned into 500 K-Means bins before average-linkage agglomerative clustering with Euclidean distance
- Confirm taxa flagged as 'uncharted' are those whose GCF models lie far from the closest MIBiG v2.0 reference BGC features
- Confirm GTDB taxonomy (version 04-RS89) joins resolved for bacterial and archaeal genomes before aggregation

## Limitations

- GCF models were built only from non-fragmented BGCs (--complete-only): 802,287 (65%) of the 1,225,071 input BGCs; fragmented BGCs are only mapped afterwards via membership assignment (n_ranks=1)
- The pre-processed full_run_result.zip (~17 GB) exists only for BiG-SLiCE v1; no pre-processed result is yet available for v2.0
- Accessing the output requires SQL(ite) queries, which the authors note has a steeper learning curve than tabular output files

## Evidence

- [results] K-Means 500 bins + agglomerative clustering of centroids: "Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance)"
- [results] MIBiG distance measurement to flag uncharted potential: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [readme] SQLite output is the intended post-processing entry point: "To access BiG-SLiCE's preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]"
- [results] Dataset scale to reconcile against: "In total, a final count of 1,225,071 BGCs were predicted from 209,206 genomes and MAGs, as reported in Table 1"
