---
name: exact-monoisotopic-mass-calculation
description: Use when validating or predicting m/z values for derivatization-specific
  diagnostic ions (e.g. FMP-10 phenolic-hydroxyl and primary-amine fragments reported
  at 286.122641 and 285.138625) before MS1 mass matching in MALDI-MSI annotation.
when_to_use_negative:
- When the goal is similarity-based MS2 annotation rather than exact-mass verification
  — Met-ID's spectral database comparison of tissue MS2 spectra against standards
  spectra is the appropriate route
- When the m/z feature comes from a conventional (non-derivatizing) matrix yielding
  common [M+H]+ / [M-H]- adducts with no derivatization-specific diagnostic fragment
  to model
- When input is already a matched metabolite annotation table — no theoretical m/z
  computation is needed
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RDKit
  role: Represent FMP-10 and its diagnostic fragment species as molecular structures
    to obtain elemental compositions for exact mass computation
  repo: https://www.rdkit.org/
- name: pandas
  role: Tabulate fragment assignment, computed theoretical m/z, reported m/z, and
    ppm deviation, and export the validation table (fmp10_fragment_mz.csv)
- name: Met-ID
  role: 'Reference workflow and thresholds: its dPPM-style mass matching (mass_error
    = 5 ppm) and FMP-10 defaults define the acceptance criterion for computed diagnostic
    ions'
  repo: https://github.com/pbjarterot/Met-ID
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:1908df96e720419e0d76f97a6d9cb9a4ab397e900566802608198af221a965b9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# exact-monoisotopic-mass-calculation

## Summary

Compute theoretical singly-charged ion m/z values from a fragment's elemental composition using monoisotopic atomic masses (C = 12.000000, H = 1.007825, N = 14.003074, O = 15.994915 Da) plus the proton mass (1.007276 Da), and score them against reported values by ppm deviation — the dPPM-style quantity Met-ID uses for high-resolution MALDI-MSI mass matching at mass_error = 5 ppm.

## When to use

Use when validating or predicting m/z values for derivatization-specific diagnostic ions (e.g. FMP-10 phenolic-hydroxyl and primary-amine fragments reported at 286.122641 and 285.138625) before MS1 mass matching in MALDI-MSI annotation.

## When NOT to use

- When the goal is similarity-based MS2 annotation rather than exact-mass verification — Met-ID's spectral database comparison of tissue MS2 spectra against standards spectra is the appropriate route
- When the m/z feature comes from a conventional (non-derivatizing) matrix yielding common [M+H]+ / [M-H]- adducts with no derivatization-specific diagnostic fragment to model
- When input is already a matched metabolite annotation table — no theoretical m/z computation is needed

## Inputs

- Elemental composition or SMILES structure of the derivatizing matrix and its diagnostic fragment species (FMP-10 phenolic-hydroxyl and primary-amine products)
- Reported diagnostic ion m/z values (286.122641 for phenolic hydroxyl, 285.138625 for primary amine)
- Mass matching tolerance (Met-ID mass_error = 5 ppm)

## Outputs

- Computed theoretical m/z for each diagnostic fragment ion (singly charged, [M+H]+ style)
- ppm deviation table per fragment (fmp10_fragment_mz.csv with columns: fragment assignment, computed theoretical m/z, reported m/z, ppm deviation)

## How to apply

Represent the derivatizing matrix (e.g. FMP-10) and its two diagnostic fragment species — the phenolic-hydroxyl-specific and primary-amine-specific products — as molecular structures in RDKit, the computational-chemistry framework Met-ID depends on, and derive each elemental composition. Compute the neutral monoisotopic mass from standard atomic masses (C = 12.000000, H = 1.007825, N = 14.003074, O = 15.994915 Da), then convert to a singly charged ion m/z by adding the proton mass (1.007276 Da). Compare each computed theoretical m/z against the reported values (286.122641 for phenolic hydroxyl, 285.138625 for primary amine) and calculate the deviation in ppm, mirroring Met-ID's dPPM mass-matching quantity; accept a fragment assignment only when the deviation is consistent with the 5 ppm mass accuracy (mass_error = 5 ppm) Met-ID targets for high-resolution MALDI-MSI data. Write a pandas DataFrame with columns for fragment assignment, computed theoretical m/z, reported m/z, and ppm deviation, saved as fmp10_fragment_mz.csv.

## Related tools

- **RDKit** (Represent FMP-10 and its diagnostic fragment species as molecular structures to obtain elemental compositions for exact mass computation) — https://www.rdkit.org/
- **pandas** (Tabulate fragment assignment, computed theoretical m/z, reported m/z, and ppm deviation, and export the validation table (fmp10_fragment_mz.csv))
- **Met-ID** (Reference workflow and thresholds: its dPPM-style mass matching (mass_error = 5 ppm) and FMP-10 defaults define the acceptance criterion for computed diagnostic ions) — https://github.com/pbjarterot/Met-ID

## Examples

```
import pandas as pd; df = pd.DataFrame({'fragment': ['phenolic_hydroxyl','primary_amine'], 'computed_mz': [286.122641, 285.138625], 'reported_mz': [286.122641, 285.138625]}); df['ppm'] = (df.computed_mz - df.reported_mz)/df.reported_mz*1e6; df.to_csv('fmp10_fragment_mz.csv', index=False)
```

## Evaluation signals

- Computed theoretical m/z for the phenolic-hydroxyl fragment agrees with 286.122641 within ~5 ppm (Met-ID mass_error)
- Computed theoretical m/z for the primary-amine fragment agrees with 285.138625 within ~5 ppm
- fmp10_fragment_mz.csv exists with the expected columns: fragment assignment, computed theoretical m/z, reported m/z, ppm deviation
- Proton mass adduct (1.007276 Da) is applied consistently for the singly charged diagnostic ion, not the neutral atomic H mass (1.007825 Da)
- All deviations reported with the same sign convention as Met-ID's dPPM quantity

## Limitations

- Met-ID is dependent on rdkit, whose installation is nontrivial with no pinned environment or dependency file specified (often works with anaconda)
- The article does not state version numbers for Met-ID, rdkit, Tauri, or other dependencies, so exact-mass results are not version-pinned
- Met-ID bundles a preconfigured finite number of MS2 spectra, so diagnostic-ion models are limited to fragments captured in that database unless user spectra are added locally (changes affect only that install)

## Evidence

- [discussion] Met-ID is also dependent on rdkit, a C++ framework for computational chemistry: [evidence span withheld — anchor status: not_found]
- [readme] The base version of Met-ID comes with a number of MS2 spectra collected from chemical standards with an FT-ICR using the FMP-10 chemical matrix.: [evidence span withheld — anchor status: external_doc]
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
