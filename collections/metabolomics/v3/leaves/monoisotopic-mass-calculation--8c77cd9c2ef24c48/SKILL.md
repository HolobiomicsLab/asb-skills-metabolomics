---
name: monoisotopic-mass-calculation
description: Use when a fragment-tree or de novo method predicts candidate molecular
  formulas and you must verify them against independently measured high-resolution
  MS m/z values using a sub-0.01 Da mass-error criterion.
when_to_use_negative:
- Measured spectra come from a low-resolution miniaturized mass spectrometer, where
  mass accuracy is insufficient to judge a 0.01 Da error threshold
- Predictions were generated in negative ion mode or from elements outside the ten
  permitted elements (C, H, O, N, P, S, Cl, Br, I, F), where TeFT's applicability
  is untested
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Mass calculator used to compute the monoisotopic exact mass of predicted molecular
    formulas
- name: pandas
  role: Compiles measured m/z, theoretical m/z, and absolute error into mass_error_verification.csv
- name: TeFT
  role: Upstream predictor supplying the molecular formulas whose monoisotopic masses
    are verified
  repo: https://github.com/thumingo/TeFT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:d5efd1dfea774a7b09ee76b3ed4214011ca491507721699d8d2767381e283583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - RDKit
  - TeFT
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# monoisotopic-mass-calculation

## Summary

Computes theoretical monoisotopic masses of predicted molecular formulas (e.g. TeFT outputs like C15H24N3O+) and validates them against high-resolution measured m/z values, flagging absolute mass errors below 0.01 Da as agreement.

## When to use

Use when a fragment-tree or de novo method predicts candidate molecular formulas and you must verify them against independently measured high-resolution MS m/z values using a sub-0.01 Da mass-error criterion.

## When NOT to use

- Measured spectra come from a low-resolution miniaturized mass spectrometer, where mass accuracy is insufficient to judge a 0.01 Da error threshold
- Predictions were generated in negative ion mode or from elements outside the ten permitted elements (C, H, O, N, P, S, Cl, Br, I, F), where TeFT's applicability is untested

## Inputs

- Measured high-resolution MS m/z values (e.g. Orbitrap peaks: 262.191 and 249.111)
- Predicted molecular formulas from TeFT (e.g. C15H24N3O+, C16H13N2O+)

## Outputs

- Summary table mass_error_verification.csv with measured m/z, theoretical m/z, and absolute mass error per fragment

## How to apply

Extract the experimentally measured m/z values (from e.g. Thermo Q-Extractive Orbitrap data, such as Supplementary Fig. S5 peaks at nominal masses 262 and 249) and list the predicted formulas (e.g. TeFT outputs C15H24N3O+ and C16H13N2O+). Compute each formula's theoretical monoisotopic mass using a mass calculator in RDKit (CalcExactMolWt on the formula as a molecule). Calculate the absolute mass error in daltons between theoretical and measured masses, and confirm the error is below the reported threshold of 0.01. Compile measured m/z, theoretical m/z, and absolute error for each fragment into a summary table saved as mass_error_verification.csv with pandas.

## Related tools

- **RDKit** (Mass calculator used to compute the monoisotopic exact mass of predicted molecular formulas)
- **pandas** (Compiles measured m/z, theoretical m/z, and absolute error into mass_error_verification.csv)
- **TeFT** (Upstream predictor supplying the molecular formulas whose monoisotopic masses are verified) — https://github.com/thumingo/TeFT

## Examples

```
from rdkit import Chem; from rdkit.Chem import rdMolDescriptors; m = rdMolDescriptors.CalcExactMolWt(Chem.MolFromSmiles('C15H24N3O'))
```

## Evaluation signals

- For each fragment, absolute mass error = |theoretical − measured| is below 0.01 Da (as reported for 262.191 matched exactly at 262.191)
- Theoretical masses reproduce reported values: 262.191 Da for C15H24N3O+ and 249.102 Da for C16H13N2O+
- mass_error_verification.csv contains exactly the columns measured m/z, theoretical m/z, and absolute error for every fragment checked

## Limitations

- Agreement is only meaningful against high-resolution measurements; the article notes MSNovelist/MassGenie-type approaches and TeFT's validation context differ in mass-accuracy requirements
- TeFT was trained only on positive ion mode (H+) spectra, restricting which predicted ions can be validated this way

## Evidence

- [other] Compute the monoisotopic masses of the TeFT-predicted formulas C15H24N3O+ (262.191) and C16H13N2O+ (249.102) using a mass calculator in RDKit.: [evidence span withheld — anchor status: not_found]
- [other] Calculate the absolute mass error in daltons between the theoretical and measured masses for each fragment, verifying the error is below 0.01 as reported.: [evidence span withheld — anchor status: not_found]
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
