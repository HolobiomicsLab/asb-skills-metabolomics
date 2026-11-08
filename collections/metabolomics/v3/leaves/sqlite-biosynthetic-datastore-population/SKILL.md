---
name: sqlite-biosynthetic-datastore-population
description: Use when a BiG-SLiCE run (full or --query) must persist processed BGCs,
  GCF models, and cluster assignments into the SQLite3 database in the output folder
  for later SQL retrieval, visualization, or GCF query mode.
when_to_use_negative:
- The dataset is small enough that pairwise comparison tools like BiG-SCAPE (quadratic
  O(n2) complexity) are tractable and network-based GCF analysis is preferred
- Input is already a tabular feature table or precomputed distance matrix rather than
  antiSMASH GenBank output — the datastore is populated from BiG-SLiCE's own preprocessing,
  not from arbitrary feature tables
- You need the exact v1.0.0 pHMM models and thresholds (T = 300–1,500) from the paper's
  run but are using BiG-SLiCE 2.0, which switched to cosine-like distances and PFAM
  35.0, making direct numeric comparison invalid
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Performs feature extraction, GCF clustering (--complete-only), membership
    assignment (--query --n_ranks), and writes all results into the SQLite3 datastore
    in the output folder
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Supplies the antiSMASH-predicted BGC GenBank files that populate the input
    folder and, in --query mode, the BGCs being placed into existing GCFs
  repo: https://antismash.secondarymetabolites.org/
- name: PyHMMER
  role: Python-native HMM scanning backend (v2.0) used during feature extraction whose
    results feed the datastore
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:4287e68f8423da62090d1951ddf7d6e9fb393a4c501bd975d0af8c787e5008ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - PyHMMER
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sqlite-biosynthetic-datastore-population

## Summary

Populate and manage the SQLite3 datastore that BiG-SLiCE writes into its output folder, holding all processed input BGCs (GenBank-derived features), GCF centroid models, and per-BGC GCF membership assignments from a large-scale run.

## When to use

Use when a BiG-SLiCE run (full or --query) must persist processed BGCs, GCF models, and cluster assignments into the SQLite3 database in the output folder for later SQL retrieval, visualization, or GCF query mode.

## When NOT to use

- The dataset is small enough that pairwise comparison tools like BiG-SCAPE (quadratic O(n2) complexity) are tractable and network-based GCF analysis is preferred
- Input is already a tabular feature table or precomputed distance matrix rather than antiSMASH GenBank output — the datastore is populated from BiG-SLiCE's own preprocessing, not from arbitrary feature tables
- You need the exact v1.0.0 pHMM models and thresholds (T = 300–1,500) from the paper's run but are using BiG-SLiCE 2.0, which switched to cosine-like distances and PFAM 35.0, making direct numeric comparison invalid

## Inputs

- BiG-SLiCE input folder with antiSMASH-predicted BGC GenBank files (prepared per the wiki Input-folder instructions)
- Optional custom GenBank inputs converted with generate_antismash_gbk.py (e.g. ClusterFinder/DeepBGC or manually refined cluster borders)
- antiSMASH output folder for --query mode placement of new BGCs into existing GCF models

## Outputs

- SQLite3 database file inside the BiG-SLiCE output folder containing processed BGC features, GCF centroid models, and cluster/membership assignments
- Ranked GCF/BGC query results (--query mode, top-n_ranks matches) recorded against the latest clustering result
- Flask web-app assets (requirements.txt, start_server.sh) for interactive visualization of the datastore

## How to apply

Run BiG-SLiCE with `bigslice -i <input_folder> <output_folder>` after preparing the input folder per the wiki (antiSMASH-predicted BGC GenBank files); the tool writes processed input data, GCF models, and cluster assignments as an SQLite3 database file inside the output folder. During the clustering phase use --complete-only so only non-fragmented BGCs build GCF centroid models (in the paper's run, 802,287 of 1,225,071 BGCs, i.e. 65%), then run `bigslice --query <antismash_output_folder> --n_ranks 1 <output_folder>` to map fragmented BGCs onto GCFs by membership values d, which is also stored in the datastore. For retrieval, write SQL(ite) queries against the database (e.g. counting BGCs per genome, GCFs per species at T = 300 vs T = 900 for alpha-diversity), or launch the bundled Flask web-app (`bash <output_folder>/start_server.sh`) which visualizes the same database. The datastore must be versioned against the BiG-SLiCE version and HMM database banner (e.g. v2.0.0, bigslice-models-2022-11-30) since clustering distances and pHMM versions changed between v1 and v2.

## Related tools

- **BiG-SLiCE** (Performs feature extraction, GCF clustering (--complete-only), membership assignment (--query --n_ranks), and writes all results into the SQLite3 datastore in the output folder) — https://github.com/medema-group/bigslice
- **antiSMASH** (Supplies the antiSMASH-predicted BGC GenBank files that populate the input folder and, in --query mode, the BGCs being placed into existing GCFs) — https://antismash.secondarymetabolites.org/
- **PyHMMER** (Python-native HMM scanning backend (v2.0) used during feature extraction whose results feed the datastore) — https://github.com/althonos/pyhmmer

## Examples

```
bigslice -i <input_folder> <output_folder> && bigslice --query <antismash_output_folder> --n_ranks 1 <output_folder>
```

## Evaluation signals

- The populated datastore contains exactly 1,225,071 BGCs traceable to 209,206 genomes and MAGs when reproducing the paper's full run (Table 1 counts)
- The --complete-only clustering phase built GCF models from exactly 802,287 non-fragmented BGCs (65% of the 1,225,071 input)
- SQL counts of GCFs at T = 300 vs T = 900 reproduce the decreasing-GCF-models trend reported for the five tested thresholds (300, 600, 900, 1,200, 1,500)
- Query mode returns top-n_ranks matching GCFs with similarity measurements and resolves against the latest clustering result in the same output folder
- The bundled Flask visualization server (start_server.sh) runs against the datastore and serves the expected genome/GCF views

## Limitations

- Postprocessing requires learning SQL(ite) queries, which the README notes has a steeper learning curve than tabular output files
- The pre-processed full-run result (~1.2M BGCs, 17 GB zipped) exists only for v1; no pre-processed v2 datastore was available at the time of the README
- BiG-SLiCE 2.0 changed the distance metric (cosine-like via l2-normalization) and pHMM database (PFAM 35.0), so a v2 datastore is not numerically comparable to v1-published GCF memberships at the same T thresholds

## Evidence

- [readme] Datastore location and contents: "**BiG-SLiCE**'s output folder contains both the processed input data (in the form of an [SQLite3](https://www.sqlite.org/index.html) database file)"
- [readme] SQL access to datastore: "To access **BiG-SLiCE**'s preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]"
- [results] Full-run scale invariant: "In total, a final count of 1,225,071 BGCs were predicted from 209,206 genomes and MAGs, as reported in Table 1"
- [results] Fragmented BGC membership assignment: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [results] Complete-only filtering for model building: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
