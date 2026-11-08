---
name: mass-spectrometry-match-thresholding
description: Use when a MASST or GNPS search result table of candidate spectral matches
  for a synthesized compound's MS/MS spectrum must be filtered to trusted matches
  before phenotypic association analysis.
when_to_use_negative:
- When the input is raw mass spectrometry data (e.g. mzML) rather than an already-generated
  MASST candidate match list — the filter operates on search results, not spectra
- When performing library-based metabolite annotation of unknowns rather than reverse
  metabolomics searching of synthesized compound spectra
- When isomer-level structural confirmation is required — the protocol does not guarantee
  against false discovery of closely related isomers or compounds
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MASST
  role: Generates the raw candidate spectral matches by searching the synthesized
    compound's MS/MS spectrum against public repository data; the filtered match table
    is produced from its output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:04a07db05955385f291261b7902b6e8615961d6d4e803d5264310da668a74d5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-spectrometry-match-thresholding

## Summary

Filters raw MASST candidate matches from reverse metabolomics repository searches using an acceptance filter of cosine score >= 0.7, at least 6 matched fragment ions, and instrument-specific precursor m/z tolerance (0.01 Orbitrap, 0.02 Q-ToF), emitting a filtered match table for downstream phenotypic analysis.

## When to use

Use when a MASST or GNPS search result table of candidate spectral matches for a synthesized compound's MS/MS spectrum must be filtered to trusted matches before phenotypic association analysis.

## When NOT to use

- When the input is raw mass spectrometry data (e.g. mzML) rather than an already-generated MASST candidate match list — the filter operates on search results, not spectra
- When performing library-based metabolite annotation of unknowns rather than reverse metabolomics searching of synthesized compound spectra
- When isomer-level structural confirmation is required — the protocol does not guarantee against false discovery of closely related isomers or compounds

## Inputs

- MASST/GNPS candidate match table (CSV/TSV) with columns for cosine score, matched fragment ion count, and precursor m/z tolerance
- MS/MS fragmentation spectrum from a newly synthesized compound
- Instrument class metadata (Orbitrap vs Q-ToF) for tolerance selection

## Outputs

- Filtered match table (CSV/TSV) containing only accepted spectral matches
- Validation report of excluded candidates failing any acceptance criterion

## How to apply

Load the raw MASST candidate match table with pandas, retaining the cosine score, matched fragment ion count, and precursor m/z tolerance columns. Retain only rows with cosine score >= 0.7 and at least 6 matched fragment ions. Apply the precursor tolerance check by instrument class: 0.01 for Orbitrap-derived data (e.g. Thermo QE Orbitrap in negative mode) and 0.02 for Q-ToF-derived data (e.g. Bruker Q-ToF or Agilent 6560 IM-Q-ToF). The rationale is that this protocol, as described in the paper, prevents completely inaccurate identifications while connecting synthesized compound spectra to public-data hits; matches failing any criterion are excluded from the output table. Emit the filtered match table as CSV/TSV for downstream phenotypic analysis (e.g. IBD cohort association of conjugated bile acids).

## Related tools

- **MASST** (Generates the raw candidate spectral matches by searching the synthesized compound's MS/MS spectrum against public repository data; the filtered match table is produced from its output)

## Examples

```
df = pd.read_csv('masst_matches.tsv', sep='\t'); filt = df[(df['cosine'] >= 0.7) & (df['matched_ions'] >= 6) & df['precursor_tol'].isin([0.01, 0.02])]; filt.to_csv('filtered_matches.csv', index=False)
```

## Evaluation signals

- Every retained row has cosine score >= 0.7
- Every retained row has matched fragment ion count >= 6
- Precursor m/z tolerance values are within {0.01, 0.02}, matching the respective instrument class (0.01 for Orbitrap, 0.02 for Q-ToF)
- All rows failing any criterion are absent from the emitted filtered match table
- USI and MASST job identifiers are retained for all synthesized compounds as traceability

## Limitations

- The paper describes this filtering stage but does not separately validate it
- The matching protocol does not guarantee against false discovery of closely related isomers or compounds, though it prevents completely inaccurate identifications
- Without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded
- MASST public spectra (~1.2 billion) do not encompass all molecules, limiting re-analyses of public data in the discovery phase

## Evidence

- [other] cosine score >= 0.7, at least 6 matched fragment ions, and precursor m/z tolerance of 0.01 (Orbitrap) or 0.02 (Q-ToF), emitting the filtered match table used downstream: [evidence span withheld — anchor status: not_found]
- [discussion] Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [supplementary] conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode: "conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode."
- [supplementary] USI and MASST job identifiers for all synthesized compounds.: "USI and MASST job identifiers for all synthesized compounds."
