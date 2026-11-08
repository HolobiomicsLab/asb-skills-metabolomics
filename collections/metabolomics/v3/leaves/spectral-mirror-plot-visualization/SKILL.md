---
name: spectral-mirror-plot-visualization
description: Use when validating a ZODIAC/SIRIUS molecular formula candidate against
  an observed MS1 isotope envelope (e.g. a diagnostic M+2 at ~106% of monoisotopic
  peak indicating Br), or when a publication figure of observed-vs-simulated isotope
  patterns must be reproduced.
when_to_use_negative:
- Input is only MS/MS fragmentation data with no MS1 isotope envelope — this skill
  compares isotope patterns, not fragment spectra or fragmentation trees.
- The goal is structural identification or spectral library matching (cosine score
  against library hits) rather than formula-level validation of an isotope signature.
- The compound's measured MS1 spectrum lacks a resolved isotope envelope in a narrow
  m/z window around the monoisotopic peak (e.g. co-eluting ions filling the 587–595
  window).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Simulates the theoretical isotope pattern of [C24H47BrNO8P + H]+ from its
    elemental composition for comparison with the observed MS1 peaks
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS
  role: Processing of input mzML/mzXML files to extract MS1 spectra from the diatoms
    LC-MS raw data
  repo: https://github.com/marcus-ludwig/OpenMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:b40c32a4feef1762d18fe0b62d99bc76c004caa982927a43a657a8f8eb8a8da6
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# spectral-mirror-plot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders a mirror plot comparing an observed MS1 isotope envelope against a SIRIUS-simulated theoretical isotope pattern (both base-peak normalized), enabling visual validation of a candidate molecular formula such as [C24H47BrNO8P + H]+ for the diatoms compound at m/z 588.230.

## When to use

Use when validating a ZODIAC/SIRIUS molecular formula candidate against an observed MS1 isotope envelope (e.g. a diagnostic M+2 at ~106% of monoisotopic peak indicating Br), or when a publication figure of observed-vs-simulated isotope patterns must be reproduced.

## When NOT to use

- Input is only MS/MS fragmentation data with no MS1 isotope envelope — this skill compares isotope patterns, not fragment spectra or fragmentation trees.
- The goal is structural identification or spectral library matching (cosine score against library hits) rather than formula-level validation of an isotope signature.
- The compound's measured MS1 spectrum lacks a resolved isotope envelope in a narrow m/z window around the monoisotopic peak (e.g. co-eluting ions filling the 587–595 window).

## Inputs

- LC-MS raw data in mzML/mzXML format (e.g. diatoms dataset, MassIVE accession MSV000081731)
- Candidate molecular formula from ZODIAC/SIRIUS annotation (e.g. C24H47BrNO8P) with adduct state [M+H]+
- SIRIUS-simulated theoretical isotope pattern (peak list of m/z and relative intensity)

## Outputs

- isotope_pattern_mirror_plot.png — mirror plot with observed MS1 spectrum on top and simulated isotope pattern inverted below
- isotope_pattern_table.csv — m/z and relative intensity for each observed and simulated isotope peak (M, M+1, M+2, ...)

## How to apply

Extract the MS1 spectrum at the compound's retention time (RT ~503.97 s for the diatoms compound) from the raw LC-MS data (MassIVE MSV000081731), keeping only peaks in a narrow window around the monoisotopic peak (m/z 587–595). Simulate the theoretical isotope pattern of the protonated formula [C24H47BrNO8P + H]+ with SIRIUS's isotope pattern analysis module, which ranks isotope patterns from high-resolution mass spectra. Normalize both the observed peak list and the simulated pattern to their base-peak intensities so relative intensities are directly comparable, then render a mirror plot with matplotlib (observed spectrum on top, simulated spectrum inverted below) and export m/z vs. relative intensity for each isotope peak (M, M+1, M+2, ...) as a CSV table. The rationale is that diagnostic isotope ratios — here the ~50:50 79Br/81Br signature producing an M+2 peak at 106.0% of the monoisotopic peak — independently confirm the elemental composition assigned by ZODIAC.

## Related tools

- **SIRIUS** (Simulates the theoretical isotope pattern of [C24H47BrNO8P + H]+ from its elemental composition for comparison with the observed MS1 peaks) — https://github.com/sirius-ms/sirius
- **OpenMS** (Processing of input mzML/mzXML files to extract MS1 spectra from the diatoms LC-MS raw data) — https://github.com/marcus-ludwig/OpenMS

## Examples

```
import matplotlib.pyplot as plt; plt.stem(mz_obs, i_obs); plt.stem(mz_sim, [-x for x in i_sim], markerfmt='C1'); plt.savefig('isotope_pattern_mirror_plot.png')
```

## Evaluation signals

- The simulated M+2/M relative-intensity ratio is near 1 (observed M+2 at 106.0% of the monoisotopic peak), matching the diagnostic ~50:50 79Br/81Br bromine signature.
- Observed peaks align with the simulated pattern within high-resolution mass accuracy across the m/z 587–595 window.
- Both observed and simulated peak intensities are base-peak normalized so intensities are directly comparable in the mirror plot.
- The CSV table contains one row per isotope peak (M, M+1, M+2, ...) with both observed and simulated m/z and relative intensity.
- No unexpected peaks appear in the observed MS1 spectrum at diagnostic offsets such as m/z +18.01 (water loss), +43.99 (CO2 loss) or -17.03 (ammonium adduct).

## Limitations

- Agreement of isotope patterns validates only the elemental composition; a complete evaluation of a novel formula annotation would require full structural elucidation, which is experimentally infeasible.
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data alone; high-quality adduct annotations should be established during preprocessing.
- The dataset must contain the halogen/silicon-bearing compounds this signature targets — the diatoms dataset is rather complex, and many compounds may contain halogens and silicon.

## Evidence
