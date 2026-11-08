---
name: sqlite-cluster-result-querying
description: Use when a BiG-SLiCE run (e.g., a threshold sweep at T = {300, 600, 900,
  1,200, 1,500}) has completed and you need per-BGC GCF assignments or GCF counts
  from the output folder's SQLite3 database to compute metrics like delta-GCF or V-score.
when_to_use_negative:
- You only need to browse a single clustering result visually — the flask-based web-app
  in the output folder (start_server.sh) is the intended interactive interface
- You need sequence-similarity network distances between individual BGC pairs rather
  than Euclidean-space GCF assignments — pairwise comparisons are BiG-SCAPE's territory
- The clustering was produced by a different tool that does not deposit results in
  a SQLite3 database
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: Produces the output folder whose SQLite3 database holds GCF models and BGC
    assignments to be queried
  repo: https://github.com/medema-group/bigslice
- name: SQLite3
  role: Query engine used to extract GCF counts and cluster assignments from the output
    database
  repo: https://www.sqlite.org/index.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:882c0cc77a8a805d549ba15cae4e29fa01aff4b0f12d7ad3d292c38be5497870
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - SQLite3
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# sqlite-cluster-result-querying

## Summary

Extract clustering results (number of GCF models and per-BGC cluster assignments) from BiG-SLiCE output folders by querying the SQLite3 database file that the tool deposits alongside its web-app visualization scripts. This is the programmatic postprocessing route the authors recommend over the browser UI for downstream analyses such as threshold sweeps and V-score computation.

## When to use

Use when a BiG-SLiCE run (e.g., a threshold sweep at T = {300, 600, 900, 1,200, 1,500}) has completed and you need per-BGC GCF assignments or GCF counts from the output folder's SQLite3 database to compute metrics like delta-GCF or V-score.

## When NOT to use

- You only need to browse a single clustering result visually — the flask-based web-app in the output folder (start_server.sh) is the intended interactive interface
- You need sequence-similarity network distances between individual BGC pairs rather than Euclidean-space GCF assignments — pairwise comparisons are BiG-SCAPE's territory
- The clustering was produced by a different tool that does not deposit results in a SQLite3 database

## Inputs

- BiG-SLiCE output folder containing the SQLite3 database file with processed input data and GCF models
- BGC-to-GCF cluster assignment records within the database (per threshold run T = 300–1,500)
- Ground-truth labels: MIBiG v1.3 compound-group annotations for the 92 reference BGCs

## Outputs

- GCF model count per threshold run (e.g., 29,955 GCFs at T = 900 in the paper's full run)
- Per-BGC GCF assignment table for the 92 MIBiG reference BGCs
- delta-GCF values between consecutive thresholds and per-threshold V-scores (harmonic mean of homogeneity and completeness)

## How to apply

After running BiG-SLiCE (bigslice -i <input_folder> <output_folder>), locate the SQLite3 database file deposited in the output folder — the README notes the output contains 'the processed input data (in the form of an SQLite3 database file)' plus flask scripts for the mini web-app. Rather than using the interactive visualization, run direct SQL queries against the database (the authors point to the sqlite.org SELECT tutorial for syntax) to pull (a) the total count of GCF models per run and (b) the GCF assignment of every input BGC — in the MIBiG validation workflow, the 92 reference BGCs and their compound-group labels. Repeat per threshold run, then compute delta-GCF between consecutive thresholds and the V-score (harmonic mean of homogeneity and completeness) against the MIBiG compound groups externally, e.g., in a Python/Jupyter script; the authors published their notebook scripts for exactly this purpose. Select the threshold maximizing the V-score (the paper's example: T = 1,100 yields V-score 0.81).

## Related tools

- **BiG-SLiCE** (Produces the output folder whose SQLite3 database holds GCF models and BGC assignments to be queried) — https://github.com/medema-group/bigslice
- **SQLite3** (Query engine used to extract GCF counts and cluster assignments from the output database) — https://www.sqlite.org/index.html

## Examples

```
bigslice -i <input_folder> <output_folder> && sqlite3 <output_folder>/*.db ".tables"
```

## Evaluation signals

- GCF model counts extracted per threshold match a monotonically decreasing trend as T increases from 300 to 1,500, as reported in the paper
- Every one of the 92 reference BGCs has exactly one GCF assignment per run; NULL or missing assignments indicate a malformed query
- Computed V-score at T = 1,100 reproduces the reported value of 0.81 within ±0.01
- delta-GCF between consecutive thresholds is recomputable and consistent when queries are re-run against the same database
- Query results agree with what the bundled flask web-app displays for the same output folder

## Limitations

- The README warns the SQL learning curve is 'steeper compared to the conventional tabular-formatted output files', though the database enables more powerful data wrangling once mastered
- BiG-SLiCE's heuristic clustering 'cannot capture the perfect clustering of the reference groups' — V-score plateaus below 1.0 even at the optimal threshold
- BiG-SLiCE 2.0 switched to cosine-like (l2-normalized) distances and PFAM 35.0 models, so absolute GCF counts and assignments may differ from the v1-era paper results (T = 300–1,500 sweep, V-score 0.81)

## Evidence

- [readme] Output folder contains SQLite3 database with processed data: [evidence span withheld — anchor status: not_found]
- [readme] SQL access is the recommended programmatic route: "To access BiG-SLiCE's preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]"
- [results] Threshold sweep produces decreasing GCF counts: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [results] GCF counts are extracted per threshold for downstream analysis: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [readme] Authors provide notebook scripts for postprocessing: "you can also [download and reuse some jupyter notebook scripts]"
