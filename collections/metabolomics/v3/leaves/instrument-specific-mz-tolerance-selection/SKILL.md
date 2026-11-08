---
name: instrument-specific-mz-tolerance-selection
description: Use when filtering MASST candidate matches for a synthesized compound's
  MS/MS spectrum and the precursor m/z tolerance must be set per instrument class
  (0.01 for Orbitrap-derived data, 0.02 for Q-ToF-derived data).
when_to_use_negative:
- The data were acquired on instruments outside the Orbitrap/Q-ToF classes covered
  here — the {0.01, 0.02} tolerance set is not defined for other platforms and would
  need re-derivation.
- The goal is not filtering MASST candidate matches (e.g. de novo library curation
  or spectral library generation rather than acceptance filtering of repository search
  results).
- Isomer-level structural confirmation is required — the article notes that without
  isolation and NMR or X-ray analyses, different isomers cannot be excluded regardless
  of the spectral filter.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MASST
  role: Searches the synthesized compound's fragmentation spectrum against public
    repositories to generate candidate matches whose precursor tolerance is then set
    by instrument class
- name: Thermo QE Orbitrap
  role: Orbitrap-class instrument whose data use the tighter 0.01 precursor m/z tolerance
- name: Bruker Q-ToF
  role: Q-ToF-class instrument whose data use the looser 0.02 precursor m/z tolerance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:61f09be9784d967f5acd5c8beb408eb6d6960ea5333ca88824b5206da9255ded
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bruker Q-ToF
  - MASST
  - Thermo QE Orbitrap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# instrument-specific-mz-tolerance-selection

## Summary

Selects the precursor m/z tolerance for spectral-match filtering based on the acquisition instrument class — 0.01 for Orbitrap data and 0.02 for Q-ToF data — as part of the reverse metabolomics MASST match acceptance filter (with cosine >= 0.7 and >= 6 matched fragment ions).

## When to use

Use when filtering MASST candidate matches for a synthesized compound's MS/MS spectrum and the precursor m/z tolerance must be set per instrument class (0.01 for Orbitrap-derived data, 0.02 for Q-ToF-derived data).

## When NOT to use

- The data were acquired on instruments outside the Orbitrap/Q-ToF classes covered here — the {0.01, 0.02} tolerance set is not defined for other platforms and would need re-derivation.
- The goal is not filtering MASST candidate matches (e.g. de novo library curation or spectral library generation rather than acceptance filtering of repository search results).
- Isomer-level structural confirmation is required — the article notes that without isolation and NMR or X-ray analyses, different isomers cannot be excluded regardless of the spectral filter.

## Inputs

- Raw MASST/GNPS candidate match table (pandas-readable CSV/TSV) with cosine score, matched fragment ion count, and precursor m/z columns
- Instrument metadata per match (Orbitrap vs. Q-ToF class, e.g. Thermo QE Orbitrap, Bruker Q-ToF, Agilent 6560 IM-Q-ToF)
- Synthesized compound MS/MS fragmentation spectrum used as the MASST query

## Outputs

- Filtered match table (CSV/TSV) retaining only candidates passing cosine >= 0.7, >= 6 matched fragment ions, and the instrument-appropriate precursor m/z tolerance (0.01 Orbitrap / 0.02 Q-ToF)

## How to apply

First determine the instrument class of each public-data match (e.g. Thermo QE Orbitrap vs. Bruker Q-ToF or Agilent 6560 IM-Q-ToF) from the MASST result metadata. Assign a precursor m/z tolerance of 0.01 to Orbitrap-derived data and 0.02 to Q-ToF-derived data, since these reflect the mass accuracy of each platform. Combine this tolerance check with the other acceptance criteria used in the paper's reverse metabolomics workflow: cosine score >= 0.7 and at least 6 matched fragment ions. Apply all three filters jointly to the raw candidate match table (retaining columns for cosine score, matched fragment ion count, and precursor m/z tolerance), then emit the filtered match table as CSV/TSV for downstream phenotypic analysis.

## Related tools

- **MASST** (Searches the synthesized compound's fragmentation spectrum against public repositories to generate candidate matches whose precursor tolerance is then set by instrument class)
- **Thermo QE Orbitrap** (Orbitrap-class instrument whose data use the tighter 0.01 precursor m/z tolerance)
- **Bruker Q-ToF** (Q-ToF-class instrument whose data use the looser 0.02 precursor m/z tolerance)

## Examples

```
df = df[(df['cosine'] >= 0.7) & (df['matched_ions'] >= 6) & (df['precursor_tol'].isin([0.01] if inst=='orbitrap' else [0.02]))]
```

## Evaluation signals

- Every retained row has cosine score >= 0.7
- Every retained row has at least 6 matched fragment ions
- Precursor m/z tolerance values are within {0.01} for Orbitrap rows and {0.02} for Q-ToF rows
- No row failing any criterion appears in the emitted filtered table
- Filtered match table is emitted as a CSV/TSV consumable by downstream phenotypic analysis

## Limitations

- The paper describes this filtering stage but does not separately validate it.
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, although it prevents completely inaccurate identifications.
- Only two instrument classes (Orbitrap: 0.01; Q-ToF: 0.02) define the tolerance set; other instrument types are not covered.

## Evidence

- [other] an acceptance filter then retains only candidates passing cosine score >= 0.7, at least 6 matched fragment ions, and precursor m/z tolerance of 0.01 (Orbitrap) or 0.02 (Q-ToF): [evidence span withheld — anchor status: not_found]
- [supplementary] conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode.: "conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode."
- [supplementary] conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode.: "conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode."
