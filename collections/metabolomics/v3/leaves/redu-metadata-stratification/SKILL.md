---
name: redu-metadata-stratification
description: Use when you have a table of matched public spectra (e.g. MASST hits
  for synthesized conjugated bile acids) with ReDU metadata attached and need to test
  whether detections are enriched in specific tissues (fecal vs blood/liver) or health
  phenotypes (IBD vs healthy).
when_to_use_negative:
- The metabolites of interest have no MASST matches in public data (e.g. they do not
  ionize well or are too low in abundance to have been selected for MS/MS), so there
  is nothing to stratify.
- The input spectra lack ReDU metadata (no biospecimen/health annotations), making
  tissue or phenotype stratification impossible.
- The question concerns structural elucidation (isomer assignment) rather than phenotype
  association — count tables cannot distinguish closely related isomers.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3337
tools:
- name: MASST
  role: Provides the repository-scale spectral matches whose hits, joined with ReDU
    metadata, are stratified by biospecimen/tissue and health phenotype
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:5199cebd8c838a97cfb7e7f10b6424f486d5d9e906c160afae53cc9c3d9465c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# redu-metadata-stratification

## Summary

Stratify matched public-data spectral counts (e.g. synthesized conjugated bile acid MASST matches) by ReDU sample metadata — biospecimen/tissue and health phenotype — and build tissue-by-conjugate and phenotype-by-conjugate count tables plus Jaccard-based ordination to reveal disease associations.

## When to use

Use when you have a table of matched public spectra (e.g. MASST hits for synthesized conjugated bile acids) with ReDU metadata attached and need to test whether detections are enriched in specific tissues (fecal vs blood/liver) or health phenotypes (IBD vs healthy).

## When NOT to use

- The metabolites of interest have no MASST matches in public data (e.g. they do not ionize well or are too low in abundance to have been selected for MS/MS), so there is nothing to stratify.
- The input spectra lack ReDU metadata (no biospecimen/health annotations), making tissue or phenotype stratification impossible.
- The question concerns structural elucidation (isomer assignment) rather than phenotype association — count tables cannot distinguish closely related isomers.

## Inputs

- filtered MASST match results table containing matched public spectra for the synthesized bile amidates
- ReDU sample metadata for each match, including biospecimen/tissue category and health phenotype (e.g. IBD vs healthy)
- list of bile acid conjugate types to stratify by

## Outputs

- tissue-by-conjugate count table (Fig. 2c equivalent)
- health-phenotype-by-conjugate count table (Fig. 2d equivalent)
- count_table.csv
- Jaccard distance matrix and ordination of samples grouped by health status

## How to apply

Load the filtered MASST match results together with the ReDU sample metadata (biospecimen/tissue and health phenotype) attached to each match. Tabulate matched spectral counts per biospecimen/tissue category crossed with conjugate type (e.g. Glu, Ile/Leu, Phe, Thr, Trp, Tyr conjugates of cholic acid) to build the tissue-distribution table, and per health phenotype (IBD vs healthy) crossed with conjugate type to build the disease-association table. Compute Jaccard distances between samples/groups based on shared bile acid conjugates and run ordination on the health-status grouping to check whether samples cluster by health status. Write both count tables to count_table.csv and verify against the reported patterns: conjugated bile acids detected mainly in fecal material and intestinal tissue (rarely blood/liver), and increased cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr in Crohn's disease cohorts.

## Related tools

- **MASST** (Provides the repository-scale spectral matches whose hits, joined with ReDU metadata, are stratified by biospecimen/tissue and health phenotype)

## Evaluation signals

- The tissue-by-conjugate table shows conjugated bile acids detected mainly in fecal material and intestinal tissue, with very few counts in blood or liver.
- The phenotype-by-conjugate table shows Glu, Ile/Leu, Phe, Thr, Trp or Tyr conjugates of cholic acid more frequent in IBD (Crohn's disease) samples than healthy individuals.
- Ordination on Jaccard distances reproduces clustering of samples by health status as reported in Fig. 2d.
- count_table.csv contains both tissue and phenotype aggregated counts with no missing ReDU annotations silently dropped.
- Expert review confirms the aggregated spectral-count patterns match Fig. 2c,d.

## Limitations

- MASST public spectra (~1.2 billion) do not encompass all molecules, limiting re-analyses of public data in the discovery phase; absent matches cannot be interpreted as true biological absence.
- Without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded, so count-table enrichment may mix isomeric structures.
- MS/MS matching does not guarantee against false discovery of closely related isomers or compounds.
- Interpretation depends on the quality and completeness of ReDU metadata deposited with public data.

## Evidence

- [results] Aggregating bile acid MASST matches by ReDU metadata shows conjugated bile acids were detected mainly in fecal material and intestinal tissue (rarely blood/liver), and Glu, Ile/Leu, Phe and Trp conjugates were more frequent in IBD samples than in healthy individuals (Fig. 2c,d).: [evidence span withheld — anchor status: not_found]
- [abstract] Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease.: "cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease"
- [discussion] the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules. This limits re-analyses of public data in the discovery phase: "the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data.: "without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [methods] 1. Load the filtered bile acid MASST match results containing matched public spectra and the ReDU sample metadata (biospecimen/tissue and health phenotype) attached to each match. 2. Group the matched spectral counts by ReDU biospecimen/tissue category and by bile acid conjugate type: [evidence span withheld — anchor status: not_found]
