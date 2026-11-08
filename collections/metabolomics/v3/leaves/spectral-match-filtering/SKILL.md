---
name: spectral-match-filtering
description: Use when a MASST search of a synthesized compound's fragmentation spectrum
  against public metabolomics repositories has produced a raw candidate match table
  and only threshold-passing matches should proceed to phenotypic/disease-association
  analysis.
when_to_use_negative:
- The goal is de novo or isomer-level structural resolution — the article notes that
  without isolation and NMR or X-ray analysis, different isomers cannot be excluded
- Candidates are spectral-library identifications of unknowns rather than reverse-MASST
  searches of newly synthesized compounds' MS/MS spectra
- The input is already a filtered/curated match table — re-filtering would discard
  valid rows
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: MASST
  role: Searches the synthesized compound's fragmentation spectrum against public
    metabolomics repositories to generate the candidate match list that this skill
    filters
- name: Nextflow
  role: Workflow engine in which the match-generation/filtering stage of the reverse
    metabolomics pipeline runs
  repo: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:57828abba505652136eb02d73ea95e95e38ae1ed3e8f1c3db174df7e56f64f67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
  tools:
  - MASST
  - Nextflow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spectral-match-filtering

## Summary

Applies the reverse-metabolomics acceptance criteria (cosine >= 0.7, >= 6 matched fragment ions, precursor m/z tolerance 0.01 for Orbitrap / 0.02 for Q-ToF) to MASST candidate matches of a synthesized compound's MS/MS spectrum, emitting the filtered match table used for downstream phenotypic analysis.

## When to use

Use when a MASST search of a synthesized compound's fragmentation spectrum against public metabolomics repositories has produced a raw candidate match table and only threshold-passing matches should proceed to phenotypic/disease-association analysis.

## When NOT to use

- The goal is de novo or isomer-level structural resolution — the article notes that without isolation and NMR or X-ray analysis, different isomers cannot be excluded
- Candidates are spectral-library identifications of unknowns rather than reverse-MASST searches of newly synthesized compounds' MS/MS spectra
- The input is already a filtered/curated match table — re-filtering would discard valid rows

## Inputs

- Raw MASST/GNPS candidate match table (CSV/TSV) for a synthesized compound searched against public repository spectra
- Instrument-class metadata indicating Orbitrap vs. Q-ToF acquisition

## Outputs

- Filtered match table (CSV/TSV) containing only rows with cosine >= 0.7, matched fragment ion count >= 6, and instrument-appropriate precursor m/z tolerance

## How to apply

Load the MASST (or GNPS) candidate match table with pandas, retaining the cosine score, matched fragment ion count, and precursor m/z tolerance columns. Retain rows with cosine >= 0.7 and at least 6 matched fragment ions, then enforce the precursor tolerance check: 0.01 for Orbitrap-derived data (e.g. Thermo QE Orbitrap) and 0.02 for Q-ToF-derived data (e.g. Bruker Q-ToF, Agilent 6560 IM-Q-ToF). Candidates failing any criterion are excluded. Emit the filtered table as CSV/TSV for downstream phenotype association (e.g. IBD cohort analysis). Note the article presents these criteria but does not separately validate this filtering stage.

## Related tools

- **MASST** (Searches the synthesized compound's fragmentation spectrum against public metabolomics repositories to generate the candidate match list that this skill filters)
- **Nextflow** (Workflow engine in which the match-generation/filtering stage of the reverse metabolomics pipeline runs) — https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation

## Examples

```
df = pd.read_csv('masst_matches.tsv', sep='\t'); df[(df.cosine >= 0.7) & (df.matched_ions >= 6) & df.precursor_tol.isin([0.01, 0.02])].to_csv('filtered_matches.csv', index=False)
```

## Evaluation signals

- Every retained row has cosine score >= 0.7
- Every retained row has at least 6 matched fragment ions
- Precursor m/z tolerance values are exactly 0.01 for Orbitrap-derived matches and 0.02 for Q-ToF-derived matches
- Any candidate failing one or more criteria is absent from the output table
- Output row count equals the number of raw candidates passing all three thresholds

## Limitations

- The protocol does not guarantee against false discovery of closely related isomers or compounds, though it prevents completely inaccurate identifications
- The article describes this filtering stage but does not separately validate it
- MASST's ~1.2 billion public spectra do not encompass all molecules, limiting re-analysis coverage in the discovery phase

## Evidence

- [supplementary] cosine >= 0.7, at least 6 matched fragment ions, and precursor m/z tolerance of 0.01 (Orbitrap) or 0.02 (Q-ToF): [evidence span withheld — anchor status: not_found]
- [discussion] Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [abstract] whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [other] The paper describes this filtering stage but does not separately validate it.: [evidence span withheld — anchor status: not_found]
