---
name: molecular-mass-mz-consistency-checking
description: Use when curating a CCS or metabolomics reference table where each row
  carries a structure identifier (InChI/SMILES), an adduct type, a listed theoretical
  m/z, and an m/z tolerance window for structural validation.
when_to_use_negative:
- Input entries lack a structure representation (InChI/SMILES) from which m/z can
  be recomputed — only the listed m/z is available, so no independent consistency
  check is possible
- The goal is CCS prediction or annotation filtering (ΔCCS threshold of 2.8% and 10
  ppm mass error), not training-data scrubbing — use CCSP 2.0 prediction instead
- Target adducts are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source
  fragments, which the CCSP 2.0 training sets do not support
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Data-cleaning workflow in which the m/z consistency check is applied to McLean
    Compendium entries before model training
  repo: https://github.com/facundof2016/CCSP2.0
- name: Python
  role: Language used to implement the scrubbing and m/z filtering logic
- name: Pandas
  role: Used to load, average duplicate CCS entries, and filter entries whose m/z
    deviates from theory
- name: PubChem
  role: Identifier Exchange Service used to convert neutral InChI strings to isomeric
    SMILES from which structures are processed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:fdf11ee7b9cd51e9fb7adf3aa3f704dfc9a96c668a75601803c8553134f851fc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Pandas
  - PubChem
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# molecular-mass-mz-consistency-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that theoretical m/z values listed for database entries match m/z recomputed from the entry's structure and ionic species, flagging mismatches beyond a small tolerance (e.g. >1 mDa) so corrupt or mislabelled records can be excluded before modeling.

## When to use

Use when curating a CCS or metabolomics reference table where each row carries a structure identifier (InChI/SMILES), an adduct type, a listed theoretical m/z, and an m/z tolerance window for structural validation.

## When NOT to use

- Input entries lack a structure representation (InChI/SMILES) from which m/z can be recomputed — only the listed m/z is available, so no independent consistency check is possible
- The goal is CCS prediction or annotation filtering (ΔCCS threshold of 2.8% and 10 ppm mass error), not training-data scrubbing — use CCSP 2.0 prediction instead
- Target adducts are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, which the CCSP 2.0 training sets do not support

## Inputs

- McLean Unified CCS Compendium adduct spreadsheets ([M-H]-, [M+H]+, [M+Na]+) with InChI strings, ionic species, listed theoretical m/z, and measured CCS values
- Comma-separated or spreadsheet database entries with structure identifiers and adduct annotations

## Outputs

- Filtered training tables with m/z-inconsistent entries removed (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions)
- Cleaned CSV files per adduct ready for CCS model training
- List of excluded entries flagged for m/z deviation greater than 1 mDa

## How to apply

Load each adduct sheet from the McLean Unified CCS Compendium and, for every entry, compute the theoretical m/z from the provided neutral InChI string and the stated ionic species. Compare the recomputed m/z against the listed theoretical m/z with a strict tolerance of 1 mDa; entries deviating more than this are excluded because the discrepancy typically reflects isotopically-labelled compounds misrepresented as monoisotopic. This scrubbing step runs in pandas alongside the other cleaning rules (duplicate CCS averaging, rare-element removal, mass-range compatibility with DeepCCS, AllCCS or CCSbase) and feeds the final training tables. Correct application yields cleaned sets of 567 [M-H]-, 518 [M+H]+ and 461 [M+Na]+ ions.

## Related tools

- **CCSP 2.0** (Data-cleaning workflow in which the m/z consistency check is applied to McLean Compendium entries before model training) — https://github.com/facundof2016/CCSP2.0
- **Python** (Language used to implement the scrubbing and m/z filtering logic)
- **Pandas** (Used to load, average duplicate CCS entries, and filter entries whose m/z deviates from theory)
- **PubChem** (Identifier Exchange Service used to convert neutral InChI strings to isomeric SMILES from which structures are processed)

## Examples

```
import pandas as pd; df['mz_calc'] = [calc_mz(inchi, adduct) for inchi, adduct in zip(df.InChI, df.adduct)]; df = df[abs(df.mz_calc - df.mz_theory) <= 0.001]
```

## Evaluation signals

- After cleaning, the three adduct tables match the reported post-scrubbing counts of 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions
- No retained entry has a recomputed m/z differing from the listed theoretical m/z by more than 1 mDa
- Excluded entries are dominated by isotopically-labelled compounds whose label mass was embedded in the listed m/z
- Remaining entries are compatible with the mass ranges set in the ML algorithms used by DeepCCS, AllCCS or CCSbase
- Duplicate CCS entries have been replaced by single averaged-CCS entries prior to the m/z filter

## Limitations

- The 1 mDa tolerance targets isotopically-labelled compound misrepresentation; other systematic m/z errors may need a different tolerance
- Requires a correctly specified ionic species per entry — adduct misannotation would corrupt the recomputed m/z
- Entries containing chemical elements appearing in fewer than five database entries are also excluded, so rare-element structures are removed regardless of m/z consistency

## Evidence

- [other] Exclude entries whose m/z computed from the provided InChI and ionic species deviates from the listed theoretical m/z by more than 1 mDa (typically isotopically-labelled compound misrepresentation): [evidence span withheld — anchor status: not_found]
- [other] confirm the cleaned tables match the reported post-scrubbing counts of 567 singly deprotonated, 518 singly protonated and 461 singly sodiated ions: [evidence span withheld — anchor status: not_found]
