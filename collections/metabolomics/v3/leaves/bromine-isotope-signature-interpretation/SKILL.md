---
name: bromine-isotope-signature-interpretation
description: Use when a SIRIUS/ZODIAC-annotated molecular formula contains Br (or
  another element with a distinctive isotope signature) and the MS1 isotope pattern
  of the annotated precursor peak can be extracted to confirm or refute the formula
  assignment.
when_to_use_negative:
- The goal is de novo molecular formula annotation from MS/MS fragmentation trees
  (SIRIUS/ZODIAC ranking), not validation of an already-annotated formula against
  its MS1 isotope pattern.
- The adduct type of the precursor is unknown — the paper notes adduct information
  cannot be reliably deduced from isotope pattern and MS/MS data, so a [M+H]+ assumption
  must come from preprocessing.
- The query compound's M+2 intensity pattern is expected to be weak or ambiguous (no
  halogen; e.g. a single Cl gives only ~32% M+2, not the near-100% bromine signature),
  leaving the pattern non-diagnostic.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Simulates/ranks theoretical isotope patterns of a candidate elemental composition
    for comparison with the measured MS1 pattern
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS
  role: Preprocessing of input mzML/mzXML files to obtain peak lists for the MS1 isotope
    comparison
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: matplotlib
  role: Renders the mirror plot of observed vs. simulated isotope patterns
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:7d9d29d900d72f82614d89a281b00ab8836b43c567cbadf696d7ebe5fde52ae1
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
  - matplotlib
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# bromine-isotope-signature-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a bromine-containing molecular formula annotation (e.g. the novel ZODIAC formula C24H47BrNO8P from the diatoms dataset) by comparing the measured high-resolution MS1 isotope pattern against a SIRIUS-simulated pattern, exploiting the diagnostic ~50:50 79Br/81Br signature that places M+2 at ~100% of the monoisotopic peak.

## When to use

Use when a SIRIUS/ZODIAC-annotated molecular formula contains Br (or another element with a distinctive isotope signature) and the MS1 isotope pattern of the annotated precursor peak can be extracted to confirm or refute the formula assignment.

## When NOT to use

- The goal is de novo molecular formula annotation from MS/MS fragmentation trees (SIRIUS/ZODIAC ranking), not validation of an already-annotated formula against its MS1 isotope pattern.
- The adduct type of the precursor is unknown — the paper notes adduct information cannot be reliably deduced from isotope pattern and MS/MS data, so a [M+H]+ assumption must come from preprocessing.
- The query compound's M+2 intensity pattern is expected to be weak or ambiguous (no halogen; e.g. a single Cl gives only ~32% M+2, not the near-100% bromine signature), leaving the pattern non-diagnostic.

## Inputs

- High-resolution LC-MS raw data (mzML/mzXML) containing the precursor peak, e.g. the diatoms dataset MassIVE accession MSV000081731
- MS1 peak list in the m/z window around the precursor (m/z 587–595 for m/z 588.230 at RT ~503.97 s)
- Candidate elemental composition from SIRIUS/ZODIAC, e.g. C24H47BrNO8P with [M+H]+ adduct assignment

## Outputs

- isotope_pattern_mirror_plot.png — mirror plot of observed (top) vs. simulated (inverted, bottom) isotope pattern
- isotope_pattern_table.csv — m/z and relative intensity for each observed and simulated isotope peak (M, M+1, M+2, ...)
- M+2/M relative-intensity ratio quantifying the bromine signature

## How to apply

Extract the MS1 spectrum around the compound's retention time (e.g. RT ~505 s in the diatoms dataset, MassIVE MSV000081731) and keep only peaks in a narrow window around the precursor (m/z 587–595, monoisotopic peak at m/z 588.230). Simulate the theoretical isotope pattern of the protonated formula ([C24H47BrNO8P + H]+) with SIRIUS isotope pattern analysis, which ranks formulas by matching isotope patterns from high-resolution spectra. Normalize both the observed peak list and the simulated pattern to base-peak intensity so relative intensities are directly comparable, then render a mirror plot (observed on top, simulated inverted below) with matplotlib. Judge success by whether the observed M+2 peak sits at ~100% of the monoisotopic peak (measured: 106.0% for m/z 588.230, consistent with one bromine atom) and whether observed peaks align with simulated m/z values within high-resolution mass accuracy.

## Related tools

- **SIRIUS** (Simulates/ranks theoretical isotope patterns of a candidate elemental composition for comparison with the measured MS1 pattern) — https://github.com/sirius-ms/sirius
- **OpenMS** (Preprocessing of input mzML/mzXML files to obtain peak lists for the MS1 isotope comparison) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **matplotlib** (Renders the mirror plot of observed vs. simulated isotope patterns)

## Examples

```
obs = obs/obs.max(); sim = sim/sim.max(); plt.bar(obs_mz, obs); plt.bar(sim_mz, -sim); plt.savefig('isotope_pattern_mirror_plot.png')
```

## Evaluation signals

- Simulated M+2/M relative-intensity ratio is near 1 (the diagnostic ~50:50 79Br/81Br signature for one bromine atom).
- Observed M+2 peak intensity falls at ~100% of the monoisotopic peak (measured 106.0% for the m/z 588.230 compound), matching the simulation within experimental error.
- Observed isotope peaks align with simulated m/z positions within high-resolution mass accuracy; both spectra are normalized to base-peak intensity before comparison.
- Alternative-formula hypotheses that would require water loss (+18.01), CO2 loss (+43.99), or ammonium adduct (−17.03) peaks in MS1 are absent, supporting the assigned formula.
- The exported CSV contains matched m/z / relative-intensity rows for M, M+1, M+2, ... for both the observed and simulated series.

## Limitations

- Adduct identity cannot be reliably deduced from isotope pattern and MS/MS data alone; high-quality adduct annotations must be established during preprocessing.
- A complete evaluation of a novel molecular formula assignment would require full structural elucidation, which is experimentally infeasible; isotope-pattern agreement validates the elemental composition, not the structure.
- Searching a compound with a novel molecular formula (not present in PubChem or ChemSpider) in a structure database will always result in an incorrect hit, often unnoticed — isotope validation should be reported instead of a database structure hit.

## Evidence
