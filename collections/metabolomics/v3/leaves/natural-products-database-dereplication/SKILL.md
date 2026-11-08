---
name: natural-products-database-dereplication
description: Use when filtered MS/MS spectra ([M+H]+ adducts, >=6 fragments, 500 most
  intense peaks, parent masses 100-1,500 Da) from specialized metabolites need annotation
  against a natural products structural database before any taxonomic re-ranking.
when_to_use_negative:
- When the goal is to re-rank or score existing candidate annotations using organism
  taxonomy - use the taxonomically informed scoring step (taxo_scorer) instead of
  a fresh database lookup
- When the benchmark aim is a fair cross-tool comparison, since all compounds in the
  benchmarking dataset are present in the DNP and ISDB-DNP is by definition backed
  by the same DB and thus favored
- When spectra target ubiquitous/primary metabolites rather than specialized metabolites,
  which is the scope the ISDB-DNP database and taxonomic scoring are designed for
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: ISDB-DNP
  role: Spectral library lookup engine backed by the Dictionary of Natural Products
    (v 27.1) used to assign ranked candidate annotations to filtered spectra
  repo: http://oolonek.github.io/ISDB/
- name: tima
  role: Repository providing the mgf filtering and annotation-pipeline scripts surrounding
    the ISDB-DNP dereplication step
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:23da66922bf849a582d6e6145d0a84229aa3cc4493fa409b00fa71d46025539f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ISDB-DNP
  - MS-Finder
  - Sirius
  - tima
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# natural-products-database-dereplication

## Summary

Dereplication of MS/MS spectra from natural product extracts by querying them against a specialized-metabolite spectral library such as ISDB-DNP (backed by the Dictionary of Natural Products v 27.1), producing ranked candidate annotations whose rank-1 correctness can be evaluated against reference identifications.

## When to use

Use when filtered MS/MS spectra ([M+H]+ adducts, >=6 fragments, 500 most intense peaks, parent masses 100-1,500 Da) from specialized metabolites need annotation against a natural products structural database before any taxonomic re-ranking.

## When NOT to use

- When the goal is to re-rank or score existing candidate annotations using organism taxonomy - use the taxonomically informed scoring step (taxo_scorer) instead of a fresh database lookup
- When the benchmark aim is a fair cross-tool comparison, since all compounds in the benchmarking dataset are present in the DNP and ISDB-DNP is by definition backed by the same DB and thus favored
- When spectra target ubiquitous/primary metabolites rather than specialized metabolites, which is the scope the ISDB-DNP database and taxonomic scoring are designed for

## Inputs

- Filtered MS/MS spectra in .mgf format ([M+H]+ adducts, >=6 fragments, 500 most intense peaks, parent mass 100-1,500 Da)
- Natural products structural/spectral database (Dictionary of Natural Products v 27.1 backing ISDB-DNP, or ISDB-UNPD as an open alternative)
- Reference identifications for benchmarking (e.g. from the OSF deposit https://osf.io/bvs6x/)

## Outputs

- Per-spectrum annotation table with ranked candidate structures and scores (e.g. benchmarking_annotations.csv)
- Rank-1 correct annotation counts and rates (ISDB-DNP: 214 (10.2%) on the 2,107-spectrum benchmark)

## How to apply

First filter the .mgf spectral file: retain only [M+H]+ adduct spectra, keep the 500 most intense peaks per spectrum, require at least 6 fragments, and keep entries with parent masses between 100 and 1,500 Da. Then query the filtered spectra against the ISDB-DNP spectral library, which is backed by the Dictionary of Natural Products (DNP v 27.1); in the library, remove duplicate structures and stereoisomers by keeping distinct InChIKey values according to the first layer (first 14 characters). Optionally, a version backed by the freely available Universal Natural Products Database (ISDB-UNPD) can be substituted when DNP access is unavailable. Finally, evaluate rank-1 correctness of each spectrum's top annotation against reference identifications: on the 2,107-spectrum benchmarking dataset ISDB-DNP returned 214 (10.2%) correct rank-1 annotations using its initial scoring system.

## Related tools

- **ISDB-DNP** (Spectral library lookup engine backed by the Dictionary of Natural Products (v 27.1) used to assign ranked candidate annotations to filtered spectra) — http://oolonek.github.io/ISDB/
- **tima** (Repository providing the mgf filtering and annotation-pipeline scripts surrounding the ISDB-DNP dereplication step) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
docker pull adafede/tima-r
```

## Evaluation signals

- On the 2,107-spectrum benchmark, ISDB-DNP with its initial scoring must return 214 correct annotations at rank 1 (10.2%)
- Every retained spectrum has >=6 fragments and at most 500 peaks, with parent mass between 100 and 1,500 Da and [M+H]+ adduct
- Library deduplication check: no two retained entries share the first 14 characters (first layer) of their InChIKey
- Output table contains one ranked candidate list per input spectrum with a machine-readable correctness flag against the reference identifications

## Limitations

- All compounds of the benchmarking dataset are present in the DNP, so ISDB-DNP is favored in any benchmark built from DNP-identified molecules
- 197 benchmarking entries showed large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the problematic entries are listed in Problematic_entries.csv but annotation tools cannot identify them automatically
- The DNP (v 27.1) is not freely downloadable, limiting reproducibility; ISDB-UNPD is the freely available substitute but with different coverage

## Evidence

- [intro] This process includes dereplication (the annotation of previously described molecules prior to any physical isolation process): "This process includes dereplication (the annotation of previously described molecules prior to any physical isolation process)"
- [discussion] filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] filtered to keep entries which parent masses were comprised between 100 and 1,500 Da: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters): "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] A version using the freely available Universal Natural Products Database (ISDB-UNPD) is available online (http://oolonek.github.io/ISDB/): "A version using the freely available Universal Natural Products Database (ISDB-UNPD) is available online (http://oolonek.github.io/ISDB/)"
- [discussion] all compounds of the benchmarking dataset are present in the DNP, and the ISDB-DNP tool, which is by definition backed by the same DB is thus favored: "all compounds of the benchmarking dataset are present in the DNP, and the ISDB-DNP tool, which is by definition backed by the same DB is thus favored"
