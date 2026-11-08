---
name: masst-match-table-parsing
description: Use when you have a raw candidate match list from a MASST search of a
  synthesized compound's tandem MS spectrum and need to retain only matches meeting
  the paper's acceptance criteria before disease-association analysis.
when_to_use_negative:
- Input is already a filtered match table that has passed the cosine ≥ 0.7, ≥ 6 fragment
  ion, and precursor m/z tolerance criteria — re-filtering will discard nothing but
  wastes a step.
- You are searching spectra from metabolite classes other than the synthesized N-acyl
  amides, fatty acid esters of hydroxy fatty acids, bile acid esters, or conjugated
  bile acids used in the paper, and the acceptance criteria may not transfer.
- Your data comes from an instrument class other than Orbitrap (Thermo QE Orbitrap)
  or Q-ToF (Bruker Q-ToF, Agilent 6560 IM-Q-ToF MS), where the 0.01 / 0.02 precursor
  m/z tolerance thresholds do not apply.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MASST
  role: Generates the raw candidate match table by searching the synthesized compound's
    fragmentation spectrum against the public repository; this skill then filters
    that table.
- name: pandas
  role: Loads and filters the raw candidate match table (cosine score, matched fragment
    ion count, precursor m/z tolerance columns) and writes the filtered CSV/TSV output.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:fcf03c9b576f95faba6f7dcaf51a77be62b4f9c3d65bcb496f835c825e79cebd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# masst-match-table-parsing

## Summary

Parses the raw MASST match table produced when a synthesized compound's MS/MS spectrum is searched against public metabolomics data and applies the paper's acceptance filter (cosine score ≥ 0.7, ≥ 6 matched fragment ions, precursor m/z tolerance of 0.01 for Orbitrap or 0.02 for Q-ToF data) to emit a filtered match table for downstream phenotypic association.

## When to use

Use when you have a raw candidate match list from a MASST search of a synthesized compound's tandem MS spectrum and need to retain only matches meeting the paper's acceptance criteria before disease-association analysis.

## When NOT to use

- Input is already a filtered match table that has passed the cosine ≥ 0.7, ≥ 6 fragment ion, and precursor m/z tolerance criteria — re-filtering will discard nothing but wastes a step.
- You are searching spectra from metabolite classes other than the synthesized N-acyl amides, fatty acid esters of hydroxy fatty acids, bile acid esters, or conjugated bile acids used in the paper, and the acceptance criteria may not transfer.
- Your data comes from an instrument class other than Orbitrap (Thermo QE Orbitrap) or Q-ToF (Bruker Q-ToF, Agilent 6560 IM-Q-ToF MS), where the 0.01 / 0.02 precursor m/z tolerance thresholds do not apply.

## Inputs

- Raw candidate match table from MASST or GNPS search of a synthesized compound's MS/MS spectrum (e.g. CSV/TSV with columns for cosine score, matched fragment ion count, and precursor m/z tolerance)

## Outputs

- Filtered match table (CSV or TSV) retaining only rows with cosine score ≥ 0.7, ≥ 6 matched fragment ions, and precursor m/z tolerance of 0.01 (Orbitrap) or 0.02 (Q-ToF)

## How to apply

Load the raw candidate match list (e.g. MASST or GNPS search result table) into pandas, retaining columns for cosine score, matched fragment ion count, and precursor m/z tolerance. Apply the cosine-score threshold of 0.7 to keep only matches at or above this acceptance level. Apply the matched-fragment-ion threshold, requiring at least 6 matched fragment ions per match. Apply the precursor m/z tolerance check, accepting 0.01 for Orbitrap-derived data and 0.02 for Q-ToF-derived data. Emit the filtered match table as a CSV (or TSV) file for downstream phenotypic association analysis. The paper describes this filtering stage but does not separately validate it, so verify retained rows against all three criteria before proceeding.

## Related tools

- **MASST** (Generates the raw candidate match table by searching the synthesized compound's fragmentation spectrum against the public repository; this skill then filters that table.)
- **pandas** (Loads and filters the raw candidate match table (cosine score, matched fragment ion count, precursor m/z tolerance columns) and writes the filtered CSV/TSV output.)

## Examples

```
import pandas as pd; df = pd.read_csv('raw_masst_matches.tsv', sep='\t'); filtered = df[(df['cosine'] >= 0.7) & (df['matched_ions'] >= 6) & (df['precursor_tol'].isin([0.01, 0.02]))]; filtered.to_csv('filtered_matches.csv', index=False)
```

## Evaluation signals

- Every retained row in the filtered table has cosine score ≥ 0.7.
- Every retained row in the filtered table has matched fragment ion count ≥ 6.
- Every retained row in the filtered table has precursor m/z tolerance of 0.01 for Orbitrap-derived data or 0.02 for Q-ToF-derived data (no other tolerance values present).
- Rows failing any of the three criteria are absent from the output table (row count matches expected intersection of the three filters).
- Output file is written as CSV or TSV and columns include cosine score, matched fragment ion count, and precursor m/z tolerance.

## Limitations

- The paper describes this filtering stage but does not separately validate it, so the exact thresholds are drawn from the workflow description rather than from a dedicated validation experiment.
- MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds; the filter prevents completely inaccurate identifications but not isomer ambiguity.
- Without isolation and NMR or X-ray structural analyses, different isomers represented by the data cannot be excluded, so filtered matches may still correspond to isomeric structures.
- The 1.2 billion spectra currently in MASST do not encompass all molecules, limiting re-analyses of public data in the discovery phase.

## Evidence

- [full_text] The spectral-match stage operates by MASST searching the synthesized compound's fragmentation spectrum against the public repository to generate candidate matches; an acceptance filter then retains only candidates passing cosine score >= 0.7, at least 6 matched fragment ions, and precursor m/z tolerance of 0.01 (Orbitrap) or 0.02 (Q-ToF), emitting the filtered match table used downstream.: [evidence span withheld — anchor status: not_found]
- [full_text] The paper describes this filtering stage but does not separately validate it.: [evidence span withheld — anchor status: not_found]
- [discussion] MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications.: [evidence span withheld — anchor status: not_found]
- [discussion] the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules.: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
