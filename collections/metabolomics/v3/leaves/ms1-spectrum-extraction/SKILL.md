---
name: ms1-spectrum-extraction
description: Use when validating a ZODIAC/SIRIUS-annotated molecular formula for a
  single LC-MS feature by comparing its measured MS1 isotope pattern (M, M+1, M+2,
  ...) against a simulated pattern from the elemental composition.
when_to_use_negative:
- The goal is de novo molecular formula annotation from MS/MS fragmentation trees
  (use SIRIUS/ZODIAC directly instead of isotope pattern validation)
- Input is already a processed peak/feature table and the raw MS1 scans containing
  the isotope envelope are unavailable
- Adduct state is unknown and cannot be established from preprocessing — the article
  notes adduct information cannot be reliably deduced from isotope pattern and MS/MS
  data alone
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Simulates the theoretical isotope pattern of the protonated formula from its
    elemental composition for comparison with the observed MS1 envelope
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS
  role: Processes the input mzML/mzXML files from which the MS1 spectrum at the feature's
    retention time is extracted
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: matplotlib
  role: Renders the normalized observed-vs-simulated isotope pattern mirror plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:27de9cc956a3c1b84b7d139d977613ee2dd22ade189bbc655bd1a04139f3a697
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS
  - SIRIUS
  - matplotlib
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# MS1 spectrum extraction and isotope pattern comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the MS1 spectrum around a specific precursor (e.g. m/z 588.230 at RT ~505 s from the diatoms dataset, MassIVE MSV000081731), normalizes observed and simulated isotope peaks to base-peak intensity, and renders a mirror plot plus CSV table to validate a candidate molecular formula such as C24H47BrNO8P by its bromine-like M+2 signature.

## When to use

Use when validating a ZODIAC/SIRIUS-annotated molecular formula for a single LC-MS feature by comparing its measured MS1 isotope pattern (M, M+1, M+2, ...) against a simulated pattern from the elemental composition.

## When NOT to use

- The goal is de novo molecular formula annotation from MS/MS fragmentation trees (use SIRIUS/ZODIAC directly instead of isotope pattern validation)
- Input is already a processed peak/feature table and the raw MS1 scans containing the isotope envelope are unavailable
- Adduct state is unknown and cannot be established from preprocessing — the article notes adduct information cannot be reliably deduced from isotope pattern and MS/MS data alone

## Inputs

- LC-MS raw data in mzML/mzXML format (e.g. diatoms dataset, MassIVE accession MSV000081731)
- Target precursor m/z and retention time (e.g. m/z 588.230, RT 503.97 s)
- Candidate elemental composition (e.g. C24H47BrNO8P) with adduct annotation ([M+H]+)

## Outputs

- isotope_pattern_mirror_plot.png (observed spectrum above, simulated inverted below)
- isotope_pattern_table.csv listing m/z and relative intensity for each observed and simulated isotope peak (M, M+1, M+2, ...)
- Extracted MS1 peak list restricted to the precursor isotope window

## How to apply

Download the LC-MS raw data (e.g. MassIVE MSV000081731) and extract the MS1 spectrum at the feature's retention time (~503.97–505 s), keeping only peaks in a narrow m/z window around the monoisotopic peak (m/z 587–595 for the m/z 588.230 compound). Simulate the theoretical isotope pattern of the protonated formula [C24H47BrNO8P + H]+ using SIRIUS isotope pattern analysis, which ranks isotope patterns from high-resolution mass spectra. Normalize both the observed MS1 peak list and the simulated pattern to their base-peak intensities so intensities are directly comparable. Render a mirror plot (observed on top, simulated inverted below) with matplotlib and export a CSV of m/z and relative intensity for each isotope peak. Validate by checking that the simulated and observed M+2/M ratios are near 1 — the diagnostic ~50:50 79Br/81Br signature — and that observed peaks align with the simulation within high-resolution mass accuracy.

## Related tools

- **SIRIUS** (Simulates the theoretical isotope pattern of the protonated formula from its elemental composition for comparison with the observed MS1 envelope) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS** (Processes the input mzML/mzXML files from which the MS1 spectrum at the feature's retention time is extracted) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **matplotlib** (Renders the normalized observed-vs-simulated isotope pattern mirror plot)

## Evaluation signals

- Simulated M+2/M relative-intensity ratio is near 1, matching the diagnostic ~50:50 79Br/81Br bromine signature
- Observed M+2 peak sits at ~106.0% of the monoisotopic peak intensity after base-peak normalization
- Observed isotope peaks align with simulated peaks within high-resolution mass accuracy across M, M+1, M+2
- CSV table contains one row per isotope peak for both observed and simulated series with comparable normalized intensities
- Negative control checks: no MS1 peaks at m/z +18.01 (water loss), +43.99 (CO2 loss) or -17.03 (ammonium adduct) around the precursor

## Limitations

- The diatoms dataset is complex and many compounds may contain halogens and silicon, so isotope envelopes of coeluting or halogen/silicon-bearing species can confound peak assignment
- A complete validation of a novel molecular formula would require full structural elucidation, which is experimentally infeasible; isotope agreement supports but does not prove the formula
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data alone; high-quality adduct annotations must be established during preprocessing

## Evidence

- [results] negative MS1 checks for novel formula: [evidence span withheld — anchor status: not_found]
