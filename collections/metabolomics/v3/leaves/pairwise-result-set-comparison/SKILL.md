---
name: pairwise-result-set-comparison
description: Use when you have top-hit annotation lists from multiple scorer/filter
  configurations (e.g. fingerprint, fragprint, totalscore scorers with element, formula,
  or InChI filters) and need to quantify whether they retrieve the same or distinct
  candidate subsets per spectrum.
when_to_use_negative:
- Input is raw MS/MS spectra or single unannotated candidate lists — run ChemDistiller
  annotation first (python annotate.py) before comparing result sets
- The goal is accuracy/retrieval-rate assessment against known ground-truth compounds
  rather than overlap between methods — overlap says nothing about which method is
  correct
- Comparing results from only one scorer/filter combination — pairwise overlap requires
  at least two configurations
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Source engine whose per-spectrum top-hit annotation results (chemdb.results)
    provide the candidate sets being compared
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python
  role: Implementation language (Python 2 and 3, 64-bit recommended) for loading results
    with pandas and computing set overlaps
- name: NumPy
  role: Numerical library required by ChemDistiller, used for vectorized overlap computations
    and matrix averaging
- name: SciPy
  role: Scientific library required by ChemDistiller, supporting the statistical aggregation
    of per-spectrum overlaps
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:1cd14cb6a42597d2d7cf8cdcfa2631aaee2da3886a8b4147b518faf8f01eac37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pairwise-result-set-comparison

## Summary

Computes pairwise percentage overlap between top-hit candidate sets returned by different annotation methods and filter combinations in ChemDistiller, quantifying how complementary each scorer/filter configuration is. This is the analysis behind Supplementary Table S5, where small-database overlaps (74–93%) are contrasted with large-database overlaps (27–78%).

## When to use

Use when you have top-hit annotation lists from multiple scorer/filter configurations (e.g. fingerprint, fragprint, totalscore scorers with element, formula, or InChI filters) and need to quantify whether they retrieve the same or distinct candidate subsets per spectrum.

## When NOT to use

- Input is raw MS/MS spectra or single unannotated candidate lists — run ChemDistiller annotation first (python annotate.py) before comparing result sets
- The goal is accuracy/retrieval-rate assessment against known ground-truth compounds rather than overlap between methods — overlap says nothing about which method is correct
- Comparing results from only one scorer/filter combination — pairwise overlap requires at least two configurations

## Inputs

- Per-spectrum top-hit annotation lists from ChemDistiller runs (chemdb.results output, e.g. annotated_spectra.json or per-spectrum result files)
- Set of method/filter configuration identifiers (scorers: fingerprint, fragprint, totalscore; filters: element, formula, InChI)
- Database scale labels separating small (HMDB, NIST14, MassBank, ChEBI) from large molecular database runs

## Outputs

- Pairwise overlap matrix (CSV) of percentage overlaps between all method/filter combinations, e.g. result_overlap_table.csv
- Per-scale mean overlap values (small vs large databases) for comparison with Supplementary Table S5 ranges

## How to apply

Load the per-spectrum top-hit annotation lists produced by the ChemDistiller annotation runs via the chemdb.results output (annotated_spectra.json / hierarchical result files) with pandas. For each spectrum, define the top-hit identifier set per method/filter combination — element, formula, and InChI filters combined with fingerprint, fragprint, and totalscore scorers. Compute pairwise overlap as the size of the intersection of the two top-hit sets relative to the compared lists, for every pair of configurations. Average the per-spectrum overlaps separately for small molecular database runs (HMDB, NIST14, MassBank, ChEBI) and large database runs, since the article reports markedly different overlap regimes for the two scales. Write the resulting pairwise overlap matrix to result_overlap_table.csv and compare against the reported ranges.

## Related tools

- **ChemDistiller** (Source engine whose per-spectrum top-hit annotation results (chemdb.results) provide the candidate sets being compared) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python** (Implementation language (Python 2 and 3, 64-bit recommended) for loading results with pandas and computing set overlaps)
- **NumPy** (Numerical library required by ChemDistiller, used for vectorized overlap computations and matrix averaging)
- **SciPy** (Scientific library required by ChemDistiller, supporting the statistical aggregation of per-spectrum overlaps)

## Examples

```
python -c "import pandas as pd; sets={m:set(df[df.method==m].id) for m in methods}; ov={(a,b):len(sets[a]&sets[b])/len(sets[b])*100 for a in methods for b in methods}; pd.DataFrame(ov).to_csv('result_overlap_table.csv')"
```

## Evaluation signals

- Computed pairwise overlap percentages fall within the reported ranges: 74–93% for small molecular databases (HMDB, NIST14, MassBank, ChEBI)
- Large-database overlaps fall within 27–78%; flag any pairwise combination outside these ranges
- Overlap matrix is symmetric with 100% on the diagonal (self-comparison of each method/filter combination)
- result_overlap_table.csv contains one row and column per method/filter combination covering all element, formula, and InChI filter × fingerprint, fragprint, totalscore scorer pairs
- Per-spectrum intersection sizes never exceed the size of the smaller of the two compared top-hit sets

## Limitations

- Overlap ranges differ sharply by database scale (74–93% small vs 27–78% large), so small-database calibration cannot be transferred to large databases; --max_results (default 10, increase for large compound databases) directly affects top-hit set size and thus overlap
- The README notes the shipped databases and SVMs are a small subset — full-scale comparisons require downloading the larger compound databases in HDF5 (*.h5) format and Radial_weighted SVMs, so overlaps computed on the bundled test databases may not generalize
- MS1 m/z search tolerance (--delta_mz, in ppm) and the per-spectrum candidate limit change which compounds appear as top hits, making overlap sensitive to run parameters rather than purely to method choice

## Evidence

- [supplementary] Pairwise result-overlap percentages between methods and filter combinations range from 74% to 93% for small molecular databases: [evidence span withheld — anchor status: not_found]
- [supplementary] from 27% to 78% for large molecular databases, per Supplementary Table S5: [evidence span withheld — anchor status: not_found]
- [methods] chemdb.results module: "chemdistiller.chemdb.results module
-----------------------------------"
- [methods] chemdistiller.filters submodules: "chemdistiller.filters.element module
------------------------------------"
- [readme] annotate.py test run output location and per-candidate limit: [evidence span withheld — anchor status: not_found]
