---
name: candidate-structure-re-ranking
description: Use when a spectral-matching tool (ISDB-DNP, MS-Finder, Sirius) returns
  multiple candidate structures for a feature and the biological source's taxonomy
  (species, genus, family) is known, so candidates can be re-ranked by taxonomic distance.
when_to_use_negative:
- Annotating ubiquitous (primary) metabolites — taxonomically informed scoring mostly
  benefits specialized metabolites and not ubiquitous molecules
- The candidate organism's taxonomy is unknown or cannot be resolved against a taxonomy
  backbone — no taxonomic distance can be computed
- A basic taxonomic filter is preferred over scoring — the article argues keeping
  individual scores independent is a better solution than a basic taxonomic filtering
  process
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3172
tools:
- name: taxo_scorer / TIMA (1_taxo_cleaner.Rmd, 2_taxo_scorer.Rmd)
  role: 'Performs the two-stage clean-and-re-rank: cleans annotation outputs, extracts
    taxonomic information with gnfinder, then complements spectral scores with taxonomic
    scores to re-rank candidates.'
  repo: https://github.com/oolonek/taxo_scorer
- name: TIMA (taxonomicallyinformedannotation/tima)
  role: Maintained repository containing everything needed to perform Taxonomically
    Informed Metabolite Annotation, including the taxo cleaner and scorer scripts.
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: gnfinder
  role: Extracts taxonomic information from free-text biological-source fields and
    matches detected names against the Catalogue of Life backbone.
  repo: https://github.com/gnames/gnfinder
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization used to find the optimal taxo-informed
    score combination (family/genus/species weights).
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: ISDB-DNP
  role: Spectral-matching tool backed by the Dictionary of Natural Products whose
    candidate list is re-ranked by taxonomic scores.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:0d335932a49fbe177087c8a481db52f6340c7d2362ed859341fb8c8b15d7f5d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - TIMA (taxonomicallyinformedannotation/tima)
  - gnfinder
  - taxo_scorer / TIMA (1_taxo_cleaner.Rmd, 2_taxo_scorer.Rmd)
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# candidate-structure-re-ranking

## Summary

Re-ranks MS/MS annotation candidates by combining classical spectral-matching scores with taxonomic-distance scores at family, genus, and species levels, using the taxonomically informed scoring system (taxo_scorer/TIMA) to promote the correct specialized-metabolite structure to rank 1.

## When to use

Use when a spectral-matching tool (ISDB-DNP, MS-Finder, Sirius) returns multiple candidate structures for a feature and the biological source's taxonomy (species, genus, family) is known, so candidates can be re-ranked by taxonomic distance.

## When NOT to use

- Annotating ubiquitous (primary) metabolites — taxonomically informed scoring mostly benefits specialized metabolites and not ubiquitous molecules
- The candidate organism's taxonomy is unknown or cannot be resolved against a taxonomy backbone — no taxonomic distance can be computed
- A basic taxonomic filter is preferred over scoring — the article argues keeping individual scores independent is a better solution than a basic taxonomic filtering process

## Inputs

- Annotation output from ISDB-DNP, MS-Finder, or Sirius (candidate structures with spectral matching scores)
- Feature quantification table with m/z and retention time (e.g. Glaucium features m/z 342.1670 at RT 1.42 min, m/z 356.1860 at RT 1.83 min)
- MGF MS/MS spectral file for the queried features
- Sample metadata containing the biological source organism (species/genus/family)
- Candidate structures with associated biological-source (taxonomy) metadata

## Outputs

- Taxo-informed re-ranked annotation table (e.g. glaucium_taxo_annotations.csv with candidates promoted to rank 1)
- Updated candidate ranking per feature combining spectral score and taxonomic distance scores
- Optimized taxo-informed score combination (e.g. 0.81/1.62/2.55 for family/genus/species)

## How to apply

Run the two-stage TIMA workflow: first (1_taxo_cleaner.Rmd) clean and standardize the annotation tool output and resolve candidate biological-source names against the Catalogue of Life backbone using gnfinder; then (2_taxo_scorer.Rmd) complement the spectral score with taxonomic scores — for ISDB-DNP the optimized combination is 0.81 at family level, 1.62 at genus level, and 2.55 at species level — while keeping the individual scores independent. Weighting is optimized by a parallelizable Bayesian optimization (ParBayesianOptimization) against the benchmarking dataset of 2,107 entries, but treat the resulting weights as training-set-dependent rather than absolute optima. Applying this scoring re-ranked the correct structures predicentrine and glaucine for the Glaucium grandiflorum features m/z 342.1670 (RT 1.42 min) and m/z 356.1860 (RT 1.83 min) from ranks 9 and 7 to rank 1, improving F1 score by a reported 1.5- to 7-fold. Judge success by verifying that the taxo-informed top-ranked candidates match experimentally confirmed structures and by increased recall and precision over classical spectral matching alone.

## Related tools

- **taxo_scorer / TIMA (1_taxo_cleaner.Rmd, 2_taxo_scorer.Rmd)** (Performs the two-stage clean-and-re-rank: cleans annotation outputs, extracts taxonomic information with gnfinder, then complements spectral scores with taxonomic scores to re-rank candidates.) — https://github.com/oolonek/taxo_scorer
- **TIMA (taxonomicallyinformedannotation/tima)** (Maintained repository containing everything needed to perform Taxonomically Informed Metabolite Annotation, including the taxo cleaner and scorer scripts.) — https://github.com/taxonomicallyinformedannotation/tima
- **gnfinder** (Extracts taxonomic information from free-text biological-source fields and matches detected names against the Catalogue of Life backbone.) — https://github.com/gnames/gnfinder
- **ParBayesianOptimization** (Parallelizable Bayesian optimization used to find the optimal taxo-informed score combination (family/genus/species weights).) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **ISDB-DNP** (Spectral-matching tool backed by the Dictionary of Natural Products whose candidate list is re-ranked by taxonomic scores.)

## Examples

```
Rscript -e 'rmarkdown::render("2_taxo_scorer.Rmd", params = list(input_annotation = "glaucium_isdb_annotations.csv", weights = c(0.81, 1.62, 2.55)))'
```

## Evaluation signals

- After taxo-informed scoring, the correct candidate structure should move to rank 1 for target features (verified for G. grandiflorum: predicentrine and glaucine promoted from ranks 9 and 7 to rank 1)
- F1 score over the benchmarking dataset should increase (reported 1.5- to 7-fold improvement versus spectral matching alone)
- Optimal score combination for ISDB-DNP should fall at approximately 0.81 (family), 1.62 (genus), 2.55 (species)
- Individual spectral and taxonomic scores must remain independent — the combination must not break score independence
- All retained entries must have taxonomy resolved against Catalogue of Life and complete down to species level

## Limitations

- Optimized score combination is heavily dependent on the training sets and should not be taken as absolute optimal values
- Taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules
- ISDB-DNP is favored in benchmarking because all benchmarking compounds are present in the DNP; GNPS spectral libraries were also part of the Sirius training set, confounding fair tool comparison
- No publicly available and downloadable databases compiling structures with fully and homogeneously resolved taxonomy currently exist

## Evidence

- [abstract] Taxo-informed scoring complements spectral score with taxonomic levels: "A script is proposed to automatically input such information at various granularity levels (species, genus, and family) and complement the score obtained"
- [discussion] Clean-and-re-rank two-stage workflow with named scripts: "1) cleaning and standardization of the outputs (1_taxo_cleaner.Rmd) 2) taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
- [discussion] Gnfinder resolves taxonomy against Catalogue of Life: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] Optimized ISDB-DNP taxo scores at family/genus/species levels: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] Reported F1 improvement from taxonomic re-ranking: "systematically improve the F1 score of the evaluated solutions (ISDB-DNP, Sirius, MS-Finder) with a 1.5 to 7-fold increase"
- [discussion] Scores optimized by Bayesian optimization, training-set dependent: "A parallelizable Bayesian optimization algorithm"
- [results] Source-card finding: taxo re-ranking promotes predicentrine/glaucine to rank 1: [evidence span withheld — anchor status: not_found]
