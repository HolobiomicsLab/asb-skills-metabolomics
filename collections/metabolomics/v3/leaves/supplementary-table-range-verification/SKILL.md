---
name: supplementary-table-range-verification
description: Use when a pipeline reproduces a paper's result table (e.g. pairwise
  overlap matrices from annotation runs) and you need a quantitative acceptance check
  against the ranges published in supplementary tables before trusting the reproduction.
when_to_use_negative:
- The annotations have not yet been produced by the ChemDistiller runs — first run
  annotate.py per method/filter combination before any overlap verification is possible
- You need per-candidate scoring quality (e.g. totalscore distributions) rather than
  agreement between candidate lists — this skill only measures set overlap of top
  hits
- The paper being reproduced does not publish reference ranges or summary tables to
  verify against — there is nothing to range-check
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: chemdistiller
  role: Source of the chemdb.results annotation outputs whose top-hit identifier sets
    are intersected per method/filter combination
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python
  role: Language for loading annotation lists with pandas, computing pairwise set
    intersections, and range-checking against S5
- name: NumPy
  role: Numerical averaging of per-spectrum overlap percentages by database-size group
- name: SciPy
  role: Supporting scientific library required by the ChemDistiller environment used
    to generate the inputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:d15b63dd6582a77a6dcbb74ad614e3a7c908037bf8f33fe2b08da48fabc2bde2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - Python
  - SciPy
  - chemdistiller
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Supplementary-table range verification

## Summary

A validation pattern that recomputes a published summary statistic (here, pairwise top-hit overlap percentages between ChemDistiller annotation methods and filter combinations) and checks that the recomputed values fall within the ranges reported in the paper's supplementary tables (74–93% for small databases, 27–78% for large databases per Supplementary Table S5).

## When to use

Use when a pipeline reproduces a paper's result table (e.g. pairwise overlap matrices from annotation runs) and you need a quantitative acceptance check against the ranges published in supplementary tables before trusting the reproduction.

## When NOT to use

- The annotations have not yet been produced by the ChemDistiller runs — first run annotate.py per method/filter combination before any overlap verification is possible
- You need per-candidate scoring quality (e.g. totalscore distributions) rather than agreement between candidate lists — this skill only measures set overlap of top hits
- The paper being reproduced does not publish reference ranges or summary tables to verify against — there is nothing to range-check

## Inputs

- Per-spectrum top-hit annotation lists from upstream ChemDistiller annotation runs (chemdb.results output)
- Mapping of spectrum -> top-hit compound identifiers for each method/filter/scorer combination (element, formula, InChI filters; fingerprint, fragprint, totalscore scorers)
- Expected ranges from Supplementary Table S5: 74–93% (small databases: HMDB, NIST14, MassBank, ChEBI) and 27–78% (large databases)

## Outputs

- result_overlap_table.csv — pairwise overlap percentage matrix between all method/filter combinations, with cells outside the reported ranges flagged
- Flag list of pairwise combinations whose recomputed overlap falls outside the Supplementary Table S5 ranges

## How to apply

Load the per-spectrum top-hit annotation lists produced by the ChemDistiller annotation runs (chemdb.results output) with pandas, and for each spectrum define the top-hit identifier set per method/filter combination — element, formula, and InChI filters crossed with fingerprint, fragprint, and totalscore scorers. Compute pairwise overlap percentages between every pair of combinations as the size of the intersection of their top-hit sets relative to the compared lists, then average per-spectrum overlaps separately for small-database runs (HMDB, NIST14, MassBank, ChEBI) and large-database runs. The rationale is that the paper reports these overlap ranges in Supplementary Table S5, so any recomputed pairwise combination falling outside 74–93% (small databases) or 27–78% (large databases) signals a reproduction error — wrong scorer, wrong filter, or wrong top-hit definition — and must be flagged rather than silently accepted. Write the resulting pairwise overlap matrix to result_overlap_table.csv with out-of-range cells explicitly flagged.

## Related tools

- **chemdistiller** (Source of the chemdb.results annotation outputs whose top-hit identifier sets are intersected per method/filter combination) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python** (Language for loading annotation lists with pandas, computing pairwise set intersections, and range-checking against S5)
- **NumPy** (Numerical averaging of per-spectrum overlap percentages by database-size group)
- **SciPy** (Supporting scientific library required by the ChemDistiller environment used to generate the inputs)

## Examples

```
import pandas as pd; ann = pd.read_json('annotated_spectra.json'); ov = len(set(a)&set(b))/len(set(a)|set(b))*100; ok = 74 <= ov <= 93
```

## Evaluation signals

- Every recomputed pairwise overlap for small molecular databases (HMDB, NIST14, MassBank, ChEBI) falls within 74–93%
- Every recomputed pairwise overlap for large molecular databases falls within 27–78%
- result_overlap_table.csv contains one row/column per method/filter combination with no missing pairwise cells
- Per-spectrum overlaps are averaged separately for small- and large-database runs, not pooled
- Any out-of-range pairwise combination is explicitly flagged rather than dropped or imputed

## Limitations

- The reported ranges (74–93% small; 27–78% large, Supplementary Table S5) are the only acceptance criteria; a reproduction could match the ranges yet differ in which specific pairwise combinations overlap
- Overlap is computed on top-hit identifier sets only, so lower-ranked candidates and their scores are ignored
- Large-database runs show much wider overlap variance (27–78%), so single out-of-range values are less diagnostic there than for small databases
- No discussion-section content was available in the document (only '_No changelog found._' and a source reference), so caveats from the paper's discussion could not be incorporated

## Evidence

- [supplementary] Pairwise result-overlap percentages between methods and filter combinations range from 74% to 93% for small molecular databases (HMDB, NIST14, MassBank, ChEBI) and from 27% to 78% for large molecular databases: [evidence span withheld — anchor status: not_found]
- [methods] chemdb.results output module is the source of per-spectrum annotation lists: [evidence span withheld — anchor status: not_found]
- [readme] ChemDistiller test-mode output includes annotated_spectra.json used for retrieval statistics: [evidence span withheld — anchor status: external_doc]
- [methods] Filters applied to candidates are element, formula, and InChI: "chemdistiller.filters.element module ------------------------------------ .. automodule:: chemdistiller.filters.element"
