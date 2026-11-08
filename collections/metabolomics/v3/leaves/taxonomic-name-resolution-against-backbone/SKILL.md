---
name: taxonomic-name-resolution-against-backbone
description: Use when a structural database or annotation output contains free-text
  organism names and you need each name resolved to a species/genus/family position
  on a taxonomy backbone (Catalogue of Life) before taxonomically informed re-ranking
  of candidates.
when_to_use_negative:
- The input already carries machine-readable, backbone-resolved taxonomy (species/genus/family
  columns) and only needs scoring, not name resolution
- The target molecules are ubiquitous primary metabolites with no taxonomic linkage
  — taxonomic resolution adds little value for such annotations
- The biological-source text is not scientific names (e.g. common names or commercial
  product names), which gnfinder's dictionary/NLP detection will not reliably resolve
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0621
tools:
- name: gnfinder
  role: Detects scientific names in free-text organism fields and verifies them against
    Global Names index / Catalogue of Life backbone
  repo: https://github.com/gnames/gnfinder
- name: taxo_scorer
  role: Consumes the backbone-resolved taxonomy to apply taxonomically informed scoring
    and re-rank candidate structures
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained repository hosting the taxonomically informed annotation pipeline,
    including 1_taxo_cleaner.Rmd and 2_taxo_scorer.Rmd that implement name cleaning
    and scoring
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:8f5669af1188c8e7b5645cc2a47744a19b1502a316de94c5b7f52c30f289e401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - ISDB-DNP
  - MS-Finder
  - gnfinder
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# taxonomic-name-resolution-against-backbone

## Summary

Resolves biological-source names found in structural databases (e.g. DNP organism fields) against the Catalogue of Life taxonomy backbone using the gnfinder program, so that candidate annotations can be scored by taxonomic distance to the sample's organism.

## When to use

Use when a structural database or annotation output contains free-text organism names and you need each name resolved to a species/genus/family position on a taxonomy backbone (Catalogue of Life) before taxonomically informed re-ranking of candidates.

## When NOT to use

- The input already carries machine-readable, backbone-resolved taxonomy (species/genus/family columns) and only needs scoring, not name resolution
- The target molecules are ubiquitous primary metabolites with no taxonomic linkage — taxonomic resolution adds little value for such annotations
- The biological-source text is not scientific names (e.g. common names or commercial product names), which gnfinder's dictionary/NLP detection will not reliably resolve

## Inputs

- Structural database entries with free-text biological-source fields (e.g. DNP v27.1 organism fields)
- Annotation tool outputs containing candidate source organisms (ISDB-DNP, MS-Finder, Sirius)
- Catalogue of Life taxonomy backbone (via Global Names index services)

## Outputs

- Taxonomically resolved database/annotation records with species, genus and family levels mapped to Catalogue of Life
- Filtered structural database restricted to entries whose taxonomy is complete down to species level

## How to apply

Extract taxonomic information from the free-text organism fields of the structural database entries using the gnfinder program, which detects scientific names via dictionary and NLP approaches and matches detected names against the Global Names index services. Set the preferred taxonomy backbone to Catalogue of Life and complete each detected name down to the species level; retain only database entries whose biological-source taxonomy resolves fully to species. The resolved species/genus/family labels are then used by the taxo_scorer / taxonomically informed scoring step (e.g. weights 0.81 family, 1.62 genus, 2.55 species for ISDB-DNP) to complement classical spectral-matching scores and re-rank candidate structures.

## Related tools

- **gnfinder** (Detects scientific names in free-text organism fields and verifies them against Global Names index / Catalogue of Life backbone) — https://github.com/gnames/gnfinder
- **taxo_scorer** (Consumes the backbone-resolved taxonomy to apply taxonomically informed scoring and re-rank candidate structures) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained repository hosting the taxonomically informed annotation pipeline, including 1_taxo_cleaner.Rmd and 2_taxo_scorer.Rmd that implement name cleaning and scoring) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
Rscript 2_taxo_scorer.Rmd --organism-organism 'Glaucium grandiflorum' --isdb-dnp-weights 0.81,1.62,2.55
```

## Evaluation signals

- Every retained database entry has a biological source resolved against the Catalogue of Life and complete down to the species level; entries failing resolution are dropped
- Each detected organism name matches exactly one (or a small set of) Catalogue of Life backbone entries, with species/genus/family strings consistent with the backbone lineage
- After downstream taxonomically informed scoring, the correct structures for known features (e.g. predicentrine and glaucine in Glaucium grandiflorum) rise to rank 1 from ranks 9 and 7 under classical spectral matching alone
- The F1 score of the annotation tool improves (reported 1.5- to 7-fold increase) relative to the un-scored spectral-only baseline

## Limitations

- Taxonomic scoring benefits specialized metabolites, not ubiquitous molecules
- The optimized score combination is heavily dependent on the training set and should not be taken as absolute optimal values
- gnfinder requires an Internet connection for full name-verification functionality against the Global Names index services
- No publicly available and downloadable database compiling structures with fully and homogeneously resolved taxonomy currently exists; resolution must be performed per-database

## Evidence

- [discussion] In order to extract taxonomic information out of the free text contents, the gnfinder program: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] the gnfinder program (https://github.com/gnames/gnfinder) was used: "the gnfinder program (https://github.com/gnames/gnfinder) was used"
- [discussion] gnfinder allows to match the detected names against the Global Names index services: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] The preferred taxonomy backbone was set to be Catalogue of Life: "The preferred taxonomy backbone was set to be Catalogue of Life"
- [discussion] only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [abstract] a score, which is usually established between the acquired data and experimental or theoretical spectral databases: "a score, which is usually established between the acquired data and experimental or theoretical spectral databases"
- [abstract] A script is proposed to automatically input such information at various granularity levels (species, genus, and family) and complement the score: "A script is proposed to automatically input such information at various granularity levels (species, genus, and family) and complement the score"
- [readme] Very fast finder of scientific names. It uses dictionary and NLP approaches.: [evidence span withheld — anchor status: external_doc]
- [readme] Works with many file formats and includes names verification against many biological databases.: [evidence span withheld — anchor status: external_doc]
- [readme] For full functionality it requires an Internet connection.: [evidence span withheld — anchor status: external_doc]
- [discussion] the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively): "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
