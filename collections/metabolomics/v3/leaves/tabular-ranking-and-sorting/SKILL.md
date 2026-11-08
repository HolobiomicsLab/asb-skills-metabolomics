---
name: tabular-ranking-and-sorting
description: Use when a feature/compound table with multi-tool annotations (GNPS,
  Sirius 6, ISDB-LOTUS) must be scored with the decision-theory k function and sorted
  by increasing k to rank natural products for novelty prioritization.
when_to_use_negative:
- Input lacks the three annotation sources (GNPS, Sirius, ISDB-LOTUS) or their reconstructed
  indicators — k cannot be aggregated from a single-tool annotation list.
- Data are raw spectra rather than an FBMN quantitative .csv plus .mgf plus Sirius
  structure_identifications.tsv.
- The goal is a generic statistical ranking (e.g., p-value sorting) unrelated to decision-theory
  knownness scoring of natural product novelty.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MS2DECIDE
  role: Provides the K_estimation function that compiles GNPS, Sirius, and ISDB-LOTUS
    annotations, filters, and sorts by k values into a ranked .tsv.
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of FBMN library annotations (strict or iterative weighted analog search)
    feeding one indicator stream of the k aggregation.
- name: Sirius
  role: Provides Sirius 6 structure_identifications.tsv confidence scores used as
    an indicator stream; missing annotations are scored 0.5.
- name: ISDB-LOTUS
  role: Provides in-silico spectral annotations (strict 0.02 Da search via spectral_lib_matcher/MatchMS);
    a zero answer is treated as novelty information.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:eeb4172d6528ef9a3afb32256b47a1150a1c492f0c71948c424c667a07a98be6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tabular-ranking-and-sorting

## Summary

Aggregate multiannotated tandem MS data (GNPS, Sirius, ISDB-LOTUS) into a decision-theory knownness score k and sort compounds in increasing k order to produce a prioritization ranking, as MS2DECIDE does for its 95-NP chemodiverse library (Table 3).

## When to use

Use when a feature/compound table with multi-tool annotations (GNPS, Sirius 6, ISDB-LOTUS) must be scored with the decision-theory k function and sorted by increasing k to rank natural products for novelty prioritization.

## When NOT to use

- Input lacks the three annotation sources (GNPS, Sirius, ISDB-LOTUS) or their reconstructed indicators — k cannot be aggregated from a single-tool annotation list.
- Data are raw spectra rather than an FBMN quantitative .csv plus .mgf plus Sirius structure_identifications.tsv.
- The goal is a generic statistical ranking (e.g., p-value sorting) unrelated to decision-theory knownness scoring of natural product novelty.

## Inputs

- Quantitative feature table (.csv, MZmine FBMN export >2.53)
- MGF file of tandem mass spectra
- Sirius 6 annotation file (structure_identifications.tsv)
- Reconstructed annotation indicators and k function from MejriY/MS2DECIDE_article_data

## Outputs

- Ranked .tsv of compounds sorted by increasing k (K_estimation output)
- table3_ranking.csv with NP identifier, k value, and rank columns
- empty.tsv report of empty annotations (optional, iterative GNPS analog search)

## How to apply

Load the reconstructed annotation-tool indicators and the decision-theory k function from the GitHub repository MejriY/MS2DECIDE_article_data, or regenerate indicators by running K_estimation on the FBMN quantitative .csv (MZmine >2.53 export), the .mgf of tandem mass spectra, and the Sirius 6 structure_identifications.tsv. Compute the k score for each compound, treating uncomputable GNPS–Sirius Tanimoto cases with the default 0.7 for T_gs and T_gi and assigning 0.5 to the Sirius matching score when Sirius proposes no annotation. Sort compounds in order of increasing k and assign sequential ranks (1 through 95 for the chemodiverse library), since lower k signals higher novelty. Assemble the ranking table (e.g., table3_ranking.csv with NP identifier, k value, and rank columns) or export the ranked .tsv that K_estimation produces. Validate by confirming the row count, checking that previously unreported NPs fall in the top ranks, and spot-checking reference entries.

## Related tools

- **MS2DECIDE** (Provides the K_estimation function that compiles GNPS, Sirius, and ISDB-LOTUS annotations, filters, and sorts by k values into a ranked .tsv.) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of FBMN library annotations (strict or iterative weighted analog search) feeding one indicator stream of the k aggregation.)
- **Sirius** (Provides Sirius 6 structure_identifications.tsv confidence scores used as an indicator stream; missing annotations are scored 0.5.)
- **ISDB-LOTUS** (Provides in-silico spectral annotations (strict 0.02 Da search via spectral_lib_matcher/MatchMS); a zero answer is treated as novelty information.)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Regenerated ranking has exactly 95 rows for the chemodiverse spectral library.
- All six unreported NPs (ids 90-95) appear within the top 10 ranks of the increasing-k ordering.
- Spot-check reference entries: Swietenin with k=-4.5 at rank 3 and Caffeic acid with k=115.5 at rank 94.
- Output is a valid .tsv with compounds in ascending k order, ranks assigned 1..N.
- Imputed defaults appear where expected: T_gs/T_gi = 0.7 when the GNPS–Sirius Tanimoto is uncomputable, Sirius matching score = 0.5 when Sirius proposes no annotation.

## Limitations

- The iterative weighted GNPS analog search launches 27 FBMN jobs and, with the 0.001 library score threshold, can fail; the README recommends raising the threshold (e.g., 0.1).
- Redundant features in Sirius structure_identifications.tsv (duplicate mappingFeatureId) raise 'ValueError: DataFrame index must be unique for orient=index' and must be cleaned first.
- Missing annotations are handled by imputed defaults (T=0.7, Sirius score 0.5), which inject assumptions into the k computation.
- ISDB-LOTUS requires a mass tolerance < 0.5 (default 0.02), and a strict zero answer is deliberately treated as novelty information.

## Evidence

- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"
- [readme] The dataframe is filtered and sorted by K-values.: [evidence span withheld — anchor status: external_doc]
- [readme] a default value of 0.7 was assigned to T_gs and T_gi in these instances: [evidence span withheld — anchor status: external_doc]
- [readme] we associated a value of 0.5 to Sirius matching score: [evidence span withheld — anchor status: external_doc]
- [readme] The function returns a tsv file containing the processed and ranked results.: [evidence span withheld — anchor status: external_doc]
