---
name: cosine-spectral-similarity-benchmarking
description: Use when tandem-MS fragmentation spectra (fragment m/z lists plus precursor
  m/z values) are available and the goal is to test whether a new alignment method
  finds structural associations that cosine-based scoring misses, especially for compounds
  with multiple structural modifications.
when_to_use_negative:
- Spectra are not in protonation/deprotonation mode — SIMILE is limited to protonation/deprotonation
  analysis, so other adduct types fall outside this benchmark's scope
- Only a single scoring method is needed and no head-to-head comparison against cosine
  similarity is intended; the skill's value is the differential call (SIMILE-significant
  vs. cosine-hit) per pair
- Spectra lack precursor m/z or fragment m/z lists, since the pair-specific similarity
  matrix and matching-ions report both require mzs and pmzs
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIMILE
  role: 'Method under benchmark: generates the fragmentation similarity matrix (similarity_matrix),
    max-weight matching (multiple_match), pro/con matrix (sym_compare), z_test significance,
    and matching_ions_report'
  repo: https://github.com/biorack/simile
- name: GNPS
  role: Cosine-based spectral similarity scoring used as the comparison/baseline method
    for molecular networking
- name: pandas
  role: Produces the matching-ions report dataframe and the compiled simile_vs_cosine_pairs.csv
    comparison table
- name: numpy
  role: Dependency of the simile package for matrix construction and significance
    testing
- name: scipy
  role: Dependency of the simile package used for the statistical z-test machinery
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:04a1196c8fca7b0ad565c953abaee4e1ffceaf46cf4779e34001991679455032
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - GNPS
  - SIMILE
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cosine-spectral-similarity-benchmarking

## Summary

Benchmark a tandem-MS spectral alignment/scoring method (SIMILE) against cosine-based similarity scoring (GNPS) on curated compound pairs, recording which structurally related pairs each method uniquely finds and characterizing SIMILE-unique pairs by their mass deltas and maximal common substructure (MCS) categories.

## When to use

Use when tandem-MS fragmentation spectra (fragment m/z lists plus precursor m/z values) are available and the goal is to test whether a new alignment method finds structural associations that cosine-based scoring misses, especially for compounds with multiple structural modifications.

## When NOT to use

- Spectra are not in protonation/deprotonation mode — SIMILE is limited to protonation/deprotonation analysis, so other adduct types fall outside this benchmark's scope
- Only a single scoring method is needed and no head-to-head comparison against cosine similarity is intended; the skill's value is the differential call (SIMILE-significant vs. cosine-hit) per pair
- Spectra lack precursor m/z or fragment m/z lists, since the pair-specific similarity matrix and matching-ions report both require mzs and pmzs

## Inputs

- Fragmentation spectra as fragment m/z lists (mzs) with precursor m/z values (pmzs), curated from the Berkeley Lab public GNPS reference library (20 flavonoids, 20 indoles)
- GNPS cosine-based spectral similarity scores for all compound pairs (comparison method)
- tolerance parameter for fragment m/z matching in sml.similarity_matrix (value not specified in the article's snippets)
- Structural annotations/MCS categories for SIMILE-unique compound pairs

## Outputs

- simile_vs_cosine_pairs.csv — table with each pair's SIMILE significance call, GNPS-cosine call, mass deltas and scores, and MCS categories for SIMILE-unique pairs
- Per-pair SIMILE p-values from sml.z_test (with the permutation null_dist returned)
- Matching-ions report dataframe (df) from sml.matching_ions_report

## How to apply

Curate reference-library spectra (e.g., 20 flavonoids and 20 indoles from the Berkeley Lab public GNPS reference library), extracting each spectrum's fragment m/z list (mzs) and precursor m/z (pmzs). Run the SIMILE pipeline with the simile Python package: sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance) builds the pair-specific fragmentation similarity matrix from row-normalized mass-difference frequencies converted to the pseudo-inverse of the normalized Laplacian (average commute time distance); sml.multiple_match and sml.sym_compare give the max-weight matching M and pro/con matrix C; sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5) assigns p-values from a null distribution built by permuting intra- and inter-spectral fragment similarity scores. In parallel, score all pairs with GNPS cosine-based spectral similarity as the established comparison method. Compile results with sml.matching_ions_report(S, M, C, mzs, pmzs) into simile_vs_cosine_pairs.csv, listing each pair's SIMILE significance call, GNPS-cosine call, mass deltas and scores, and MCS categories for pairs found uniquely by SIMILE. Judge success by whether the table covers every pair (40 spectra = 780 pairs) and whether SIMILE-significant pairs show a superset-different enrichment over cosine hits (e.g., 97 flavonoid pairs uniquely found by SIMILE) with multiple structural modifications.

## Related tools

- **SIMILE** (Method under benchmark: generates the fragmentation similarity matrix (similarity_matrix), max-weight matching (multiple_match), pro/con matrix (sym_compare), z_test significance, and matching_ions_report) — https://github.com/biorack/simile
- **GNPS** (Cosine-based spectral similarity scoring used as the comparison/baseline method for molecular networking)
- **pandas** (Produces the matching-ions report dataframe and the compiled simile_vs_cosine_pairs.csv comparison table)
- **numpy** (Dependency of the simile package for matrix construction and significance testing)
- **scipy** (Dependency of the simile package used for the statistical z-test machinery)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance); M = sml.multiple_match(S, spec_ids)
```

## Evaluation signals

- Completeness check: the comparison table reports both a SIMILE and a cosine call for every one of the 780 pairs (40 choose 2)
- Enrichment check: the set of SIMILE-significant pairs is a superset-different enrichment relative to cosine hits, consistent with 97 flavonoid pairs found uniquely by SIMILE
- Statistic validity: sml.z_test returns finite spec_scores and pval along with null_dist, computed with log_size=5 from permuted intra- and inter-spectral fragment similarity scores
- Report schema: sml.matching_ions_report output includes mass deltas, scores, and MCS categories for SIMILE-unique pairs
- Biological sanity: significant SIMILE alignments correspond to moderate-to-high structural similarity (verified via MCS categories and characterized mass deltas, e.g., 30.983 ≈ [+O2 −H])

## Limitations

- SIMILE is currently limited to analysis of protonation/deprotonation; comprehensively addressing additional adducts is left to future research
- Parameters used to compare alignment algorithms were not optimized; optimization entails unquantified trade-offs (e.g., improving specificity vs. number of hits)
- There will likely be classes of molecules more appropriate for cosine-based scoring than SIMILE-based scoring; without statistical significance, intermediate similarity scores are to some degree uninterpretable
- The simile package is pinned to Python 3.7 due to non-SIMILE bugs

## Evidence

- [abstract] SIMILE, applied to two molecular networking datasets, identified spectral alignments between small molecules with multiple differences that were not found by cosine-based similarity scores: [evidence span withheld — anchor status: not_found]
- [methods] z_test call with null distribution: "spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [discussion] SIMILE yields more associations than cosine: "our alignment method, SIMILE, yields different and typically more associations than cosine-based scoring algorithms"
- [discussion] Recommendation to complement with cosine scoring: "We recommend complementing SIMILE with the use of cosine-based scoring for compound identification."
- [readme] Null distribution leveraging intraspectral comparisons: "A null distribution for spectral similarity which leverages intraspectral comparisons to add confidence to interspectral comparisons."
