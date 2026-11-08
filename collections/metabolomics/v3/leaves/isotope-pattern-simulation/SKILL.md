---
name: isotope-pattern-simulation
description: Use when a molecular formula annotation (e.g. from SIRIUS/ZODIAC) needs
  validation from high-resolution MS1 isotope peaks, or when a diagnostic elemental
  signature such as ~50:50 79Br/81Br must be checked.
when_to_use_negative:
- Input is already an MS/MS (tandem) spectrum and the goal is fragmentation-tree or
  structure annotation rather than MS1 isotope validation — use SIRIUS fragmentation
  tree / ZODIAC reranking instead.
- The compound's adduct cannot be reliably deduced from isotope pattern and MS/MS
  data and no high-quality adduct annotation is available from preprocessing — the
  paper explicitly warns that adduct information 'is challenging to deduce from isotope
  pattern and MS/MS data'.
- The measured mass spectrometer is low-resolution, so isotope peaks cannot be separated
  and matched within high-resolution mass accuracy.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: SIRIUS
  role: 'Performs the isotope pattern analysis: simulates the theoretical isotope
    pattern of the elemental composition of the protonated formula for comparison
    against the observed MS1 peak list'
  repo: https://github.com/sirius-ms/sirius
- name: matplotlib
  role: Renders the mirror plot comparing observed (top) and simulated (inverted below)
    normalized isotope patterns
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:eaf101e229fc184a0699f1f5f72a32e78a1ac7319cd58e37ba5f61cbd10283b7
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
  - SIRIUS
  - matplotlib
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# isotope-pattern-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate the theoretical isotope pattern of a candidate molecular formula (e.g. [C24H47BrNO8P + H]+) with SIRIUS and compare it, peak-by-peak, against the observed MS1 spectrum to support or refute formula annotations. This is the diagnostic used in the ZODIAC paper to validate novel formula assignments such as the brominated diatom metabolite at m/z 588.230.

## When to use

Use when a molecular formula annotation (e.g. from SIRIUS/ZODIAC) needs validation from high-resolution MS1 isotope peaks, or when a diagnostic elemental signature such as ~50:50 79Br/81Br must be checked.

## When NOT to use

- Input is already an MS/MS (tandem) spectrum and the goal is fragmentation-tree or structure annotation rather than MS1 isotope validation — use SIRIUS fragmentation tree / ZODIAC reranking instead.
- The compound's adduct cannot be reliably deduced from isotope pattern and MS/MS data and no high-quality adduct annotation is available from preprocessing — the paper explicitly warns that adduct information 'is challenging to deduce from isotope pattern and MS/MS data'.
- The measured mass spectrometer is low-resolution, so isotope peaks cannot be separated and matched within high-resolution mass accuracy.

## Inputs

- Raw LC-MS data in mzML/mzXML format (e.g. MassIVE accession MSV000081731)
- Candidate molecular formula from SIRIUS/ZODIAC annotation (e.g. C24H47BrNO8P)
- Adduct/ionization state assumption (e.g. [M+H]+)
- Retention time and monoisotopic m/z of the target compound (e.g. RT 503.97 s, m/z 588.230)

## Outputs

- Simulated theoretical isotope pattern (m/z and relative intensity per isotope peak)
- CSV table (isotope_pattern_table.csv) listing m/z and relative intensity for each observed and simulated isotope peak (M, M+1, M+2, ...)
- Mirror plot image (isotope_pattern_mirror_plot.png) with observed spectrum on top and simulated pattern inverted below
- Validated/refuted formula assignment with diagnostic element confirmation (e.g. bromine signature)

## How to apply

Extract the MS1 spectrum at the compound's retention time (in the diatoms dataset, ~505 s for the m/z 588.230 compound) and keep only peaks in the narrow m/z window around the monoisotopic peak (m/z 587–595). Use SIRIUS isotope pattern analysis to simulate the theoretical pattern of the protonated formula [C24H47BrNO8P + H]+, then normalize both observed and simulated peak lists to their base-peak intensities so the M, M+1, M+2... peaks are directly comparable. Render a mirror plot (observed on top, simulated inverted below) and export a CSV of m/z and relative intensity per isotope peak. Judge correctness by whether the simulated M+2/M ratio approaches 1 — the diagnostic 50:50 79Br/81Br signature of one bromine atom — and whether observed peaks align with simulated masses within high-resolution accuracy.

## Related tools

- **SIRIUS** (Performs the isotope pattern analysis: simulates the theoretical isotope pattern of the elemental composition of the protonated formula for comparison against the observed MS1 peak list) — https://github.com/sirius-ms/sirius
- **matplotlib** (Renders the mirror plot comparing observed (top) and simulated (inverted below) normalized isotope patterns)

## Examples

```
sirius --input diatoms_ms1_mz588.mzml --formula C24H47BrNO8P --isotope-pattern && python mirror_plot.py --observed isotope_pattern_table.csv
```

## Evaluation signals

- Simulated M+2/M relative-intensity ratio is near 1 (≈106 % of the monoisotopic peak for C24H47BrNO8P + H)+, matching the diagnostic ~50:50 79Br/81Br bromine signature
- Observed isotope peaks align with simulated peak masses within high-resolution mass accuracy (e.g. monoisotopic peak at m/z 588.230)
- Both observed and simulated peak lists are normalized to base-peak intensity so relative intensities are directly comparable
- Mirror plot and CSV table (isotope_pattern_table.csv) contain matching M, M+1, M+2, ... entries for both observed and simulated series
- No peaks should be attributed to alternative-adduct or coeluting species at m/z +18.01, +43.99 or −17.03 relative to the monoisotopic mass, consistent with the paper's novel-formula check

## Limitations

- The paper notes that a complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible — isotope pattern agreement supports but does not prove the formula.
- Adduct state must be known beforehand: 'adduct information cannot be reliably deduced from isotope pattern and MS/MS data', so wrong adduct assumptions yield wrong simulated masses.
- The diatoms dataset is 'rather complex, and many compounds may contain halogens and silicon', so additional isotope signatures (Cl, S, Si) can complicate attribution in similar samples.
- Peaks from coeluting or adduct-related species near the monoisotopic m/z must be excluded or they will corrupt the normalized observed pattern.

## Evidence

- [results] Bromine M+2 isotope signature is the diagnostic check: [evidence span withheld — anchor status: not_found]
- [discussion] Isotope/adduct deducibility limitation: [evidence span withheld — anchor status: not_found]
