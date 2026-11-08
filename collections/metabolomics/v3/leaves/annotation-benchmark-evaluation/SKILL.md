---
name: annotation-benchmark-evaluation
description: Use when you need baseline annotation performance for ISDB-DNP, Sirius,
  or MS-Finder on a labelled MS/MS spectra set (e.g. 2,107 reference-identified spectra)
  before evaluating any re-ranking or rescoring method.
when_to_use_negative:
- The dataset is not a labelled benchmarking set with reference identifications, so
  rank-1 correctness cannot be judged against ground truth.
- You are evaluating re-ranking after taxonomically informed scoring is already applied
  — those baselines are then used as the denominator, not recomputed with taxonomic
  weights.
- The compounds of interest are absent from the DNP; the article notes the benchmarking
  favors ISDB-DNP because all benchmarking compounds are present in the DNP, so results
  would not be comparable.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ISDB-DNP
  role: One of the three annotation tools benchmarked; queried against the 2,107 filtered
    spectra to produce candidate annotations using its initial scoring system.
  repo: http://oolonek.github.io/ISDB/
- name: Sirius
  role: Annotated formulas and structures for the same filtered spectra to establish
    its baseline rank-1 correct count before taxonomically informed scoring.
- name: MS-Finder
  role: Annotated formulas and structures for the same filtered spectra to establish
    its baseline rank-1 correct count before taxonomically informed scoring.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:d37f28df9659a0cfc009234f05e9f93f5acec61c14b24cd9736eb8f24285ec12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ISDB-DNP
  - MS-Finder
  - Sirius
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# annotation-benchmark-evaluation

## Summary

Benchmark metabolite annotation tools by running each tool's default scoring over a shared filtered MS/MS spectra set and counting rank-1 correct annotations against reference identifications, producing baseline counts (e.g. ISDB-DNP 214/10.2%, Sirius 975/46.3%, MS-Finder 180/8.5%) against which taxonomically informed re-ranking gains are measured.

## When to use

Use when you need baseline annotation performance for ISDB-DNP, Sirius, or MS-Finder on a labelled MS/MS spectra set (e.g. 2,107 reference-identified spectra) before evaluating any re-ranking or rescoring method.

## When NOT to use

- The dataset is not a labelled benchmarking set with reference identifications, so rank-1 correctness cannot be judged against ground truth.
- You are evaluating re-ranking after taxonomically informed scoring is already applied — those baselines are then used as the denominator, not recomputed with taxonomic weights.
- The compounds of interest are absent from the DNP; the article notes the benchmarking favors ISDB-DNP because all benchmarking compounds are present in the DNP, so results would not be comparable.

## Inputs

- .mgf MS/MS spectra file filtered for [M+H]+ adducts, ≤500 most intense peaks, parent mass 100–1,500 Da, ≥6 fragments (2,107 benchmarking spectra)
- Reference identifications (ground-truth annotations) for each benchmarking spectrum
- ISDB-DNP spectral library backed by Dictionary of Natural Products (v 27.1)

## Outputs

- Per-tool annotation table with rank-1 correct annotation counts and rates (benchmarking_annotations.csv): ISDB-DNP 214 (10.2%), Sirius 975 (46.3%), MS-Finder 180 (8.5%)
- Baseline F1 scores per tool used for later comparison against taxonomically informed scoring

## How to apply

Download the benchmarking spectra and their reference identifications from the OSF deposit (https://osf.io/bvs6x/), then filter the .mgf spectral files retaining only [M+H]+ adducts, the 500 most intense peaks per spectrum, parent masses between 100 and 1,500 Da, and at least 6 fragments per spectrum. Query the 2,107 filtered spectra against each annotation tool in turn: ISDB-DNP (spectral library backed by Dictionary of Natural Products v 27.1), Sirius, and MS-Finder, using each tool's initial/default scoring system without taxonomic information. Evaluate rank-1 correctness of each tool's top annotation against the reference identifications and record per-tool counts and rates. These baselines are the denominator for computing the F1-score improvement (reported 1.5- to 7-fold) that taxonomically informed scoring later achieves. Validation: the reproduced rank-1 counts must match the reported values — ISDB-DNP 214 (10.2%), Sirius 975 (46.3%), MS-Finder 180 (8.5%).

## Related tools

- **ISDB-DNP** (One of the three annotation tools benchmarked; queried against the 2,107 filtered spectra to produce candidate annotations using its initial scoring system.) — http://oolonek.github.io/ISDB/
- **Sirius** (Annotated formulas and structures for the same filtered spectra to establish its baseline rank-1 correct count before taxonomically informed scoring.)
- **MS-Finder** (Annotated formulas and structures for the same filtered spectra to establish its baseline rank-1 correct count before taxonomically informed scoring.)

## Examples

```
Rscript -e 'filter_mgf(input = "benchmarking.mgf", adduct = "[M+H]+", top_peaks = 500, mass_range = c(100,1500), min_fragments = 6)'
```

## Evaluation signals

- Reproduced rank-1 correct counts match reported values: ISDB-DNP 214 (10.2%), Sirius 975 (46.3%), MS-Finder 180 (8.5%).
- Spectra filter invariants hold: only [M+H]+ adducts, ≤500 most intense peaks kept, parent masses 100–1,500 Da, ≥6 fragments per spectrum.
- Number of spectra after filtering equals 2,107 distinct entries.
- Rank-1 correctness is computed strictly against the reference identifications from the OSF deposit, not tool self-reported scores.

## Limitations

- The benchmarking does not allow a fair comparison of tool performances because all compounds of the benchmarking dataset are present in the DNP, favoring the ISDB-DNP tool backed by the same DB.
- The GNPS spectral libraries were also part of the Sirius training set, confounding tool comparison.
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; a list is provided as Problematic_entries.csv but erroneous entries cannot be identified by annotation tools.

## Evidence

- [discussion] Spectra filter thresholds: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] Parent mass range: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] Benchmarking dataset size: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [results] Baseline rank-1 counts: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmarking favoring caveat: "all compounds of the benchmarking dataset are present in the DNP, and the ISDB-DNP tool, which is by definition backed by the same DB is thus favored"
