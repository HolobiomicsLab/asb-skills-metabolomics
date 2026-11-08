---
name: molecular-formula-isotope-matching
description: Use when a feature's formula annotation (e.g., a novel ZODIAC/SIRIUS
  formula like C24H47BrNO8P at m/z 588.230) needs independent confirmation from its
  MS1 isotope envelope, especially for halogen-containing formulas.
when_to_use_negative:
- Input is already a processed feature table without access to raw MS1 scans — the
  isotope envelope cannot be re-extracted.
- The candidate formula lacks diagnostic heteroatoms and the MS1 isotope envelope
  is too low-intensity or unresolved to discriminate between elemental compositions.
- The goal is structural elucidation rather than formula validation — isotope pattern
  matching cannot distinguish isomers and adduct information should come from preprocessing,
  not the isotope pattern.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS
  role: Simulates the theoretical isotope pattern of the protonated molecular formula
    from its elemental composition and ranks isotope patterns from high-resolution
    mass spectra
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS
  role: Processes input mzML/mzXML files and supports extraction of MS1 peaks for
    the isotope envelope; a modified OpenMS 2.4.0 was used in the ZODIAC workflow
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:a7b5e3600e5c114104d0d2647dba982eb030a0ab7e184773b22070af129a8447
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - OpenMS
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# molecular-formula-isotope-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a candidate molecular formula annotation by comparing the measured high-resolution MS1 isotope pattern of a feature against the theoretical isotope pattern simulated from the elemental composition (e.g., SIRIUS isotope pattern analysis), checking diagnostic signatures such as the ~50:50 79Br/81Br M+2 peak.

## When to use

Use when a feature's formula annotation (e.g., a novel ZODIAC/SIRIUS formula like C24H47BrNO8P at m/z 588.230) needs independent confirmation from its MS1 isotope envelope, especially for halogen-containing formulas.

## When NOT to use

- Input is already a processed feature table without access to raw MS1 scans — the isotope envelope cannot be re-extracted.
- The candidate formula lacks diagnostic heteroatoms and the MS1 isotope envelope is too low-intensity or unresolved to discriminate between elemental compositions.
- The goal is structural elucidation rather than formula validation — isotope pattern matching cannot distinguish isomers and adduct information should come from preprocessing, not the isotope pattern.

## Inputs

- LC-MS/MS raw data (mzML/mzXML, e.g., MassIVE MSV000081731 diatoms dataset)
- Candidate molecular formula with adduct assignment (e.g., [C24H47BrNO8P + H]+) from SIRIUS/ZODIAC
- MS1 peak list (m/z and intensity) at the feature retention time

## Outputs

- isotope_pattern_mirror_plot.png (matplotlib mirror plot of observed vs. simulated pattern)
- isotope_pattern_table.csv (m/z and relative intensity for observed and simulated M, M+1, M+2, ... peaks)
- Verdict on whether the measured isotope pattern supports the annotated formula

## How to apply

Extract the MS1 spectrum at the feature's retention time (e.g., ~505 s in the diatoms dataset) and keep only peaks in a narrow m/z window around the monoisotopic peak (e.g., 587–595 for m/z 588.230). Simulate the theoretical isotope pattern of the protonated formula [C24H47BrNO8P + H]+ with SIRIUS isotope pattern analysis. Normalize both observed and simulated peak lists to base-peak intensity, then render a mirror plot (observed on top, simulated inverted) and export a CSV table of m/z and relative intensity for M, M+1, M+2, ... peaks. Validation hinges on the M+2/M relative-intensity ratio being near 1 (the diagnostic ~50:50 79Br/81Br signature for one bromine) and observed peaks aligning with the simulated pattern within high-resolution mass accuracy; the article additionally checked MS1 for peaks at m/z +18.01 (water loss), +43.99 (CO2 loss), and -17.03 (ammonium adduct) to rule out alternative formulas.

## Related tools

- **SIRIUS** (Simulates the theoretical isotope pattern of the protonated molecular formula from its elemental composition and ranks isotope patterns from high-resolution mass spectra) — https://github.com/sirius-ms/sirius
- **OpenMS** (Processes input mzML/mzXML files and supports extraction of MS1 peaks for the isotope envelope; a modified OpenMS 2.4.0 was used in the ZODIAC workflow) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
sirius --isotope-pattern --formula C24H47BrNO8P --ionization '[M+H]+' raw.mzML && python mirror_plot.py --observed ms1_588.csv --simulated sirius_iso.csv --out isotope_pattern_mirror_plot.png
```

## Evaluation signals

- Simulated M+2/M relative-intensity ratio is near 1 for a mono-brominated formula, matching the ~50:50 79Br/81Br signature.
- Measured M+2 peak intensity (e.g., ~106.0% of the monoisotopic peak) agrees with the theoretical pattern after base-peak normalization.
- Observed isotope peaks align with the simulated pattern m/z values within high-resolution mass accuracy.
- No unexplained MS1 peaks at m/z +18.01 (water loss), +43.99 (CO2 loss), or -17.03 (ammonium adduct) that would suggest alternative formulas ±H2O/CO2/NH3.
- The mirror plot shows observed (top) and inverted simulated (bottom) spectra with coinciding peaks after normalization.

## Limitations

- Isotope pattern agreement alone cannot confirm a structure; complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible.
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data — high-quality adduct annotations should be established during preprocessing.
- Searching a compound with a novel molecular formula in a structure database will always return an incorrect hit, so validation must remain database-independent.
- SIRIUS's formula annotation performance drops substantially for compounds above 500 Da, affecting the reliability of the candidate formula being tested.

## Evidence

- [results] MS1 spectrum peaks at m/z +18.01 (water loss), +43.99 (CO2 loss) or -17.03 (ammonium adduct) absent: [evidence span withheld — anchor status: not_found]
- [discussion] adduct information cannot be reliably deduced from isotope pattern and MS/MS data: [evidence span withheld — anchor status: not_found]
