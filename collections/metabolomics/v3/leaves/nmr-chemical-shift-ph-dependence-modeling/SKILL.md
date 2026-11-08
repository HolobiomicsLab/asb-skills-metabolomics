---
name: nmr-chemical-shift-ph-dependence-modeling
description: Use when simulating 1D or 2D metabolomic 1H NMR spectra at a target pH
  different from the standard pH0 (usually 7.4) and you must reposition peaks that
  shift with pH.
when_to_use_negative:
- The metabolite's peaks do not shift with pH in the region of interest (Δδ effectively
  zero between pH0 and pH1) — applying the transform adds no realism
- You only need the standard spectrum at pH0 itself; no shift correction is required
- Target peaks are 13C or non-exchangeable nuclei without an ionizable group modelled
  by δL/δHL and a pKa in HMDB 5.0
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetAssimulo 2.0
  role: Web application whose simulation pipeline applies equation (3) to shift 1D
    and 2D metabolite peaks to the target pH
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: MetAssimulo 2
  role: Hosts the peak_shift.py module implementing the reusable shift functions for
    1D peaks and 2D clusters
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Language in which the shift computation functions are written and unit-tested
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:c0404b636ad6f23dd66f251ac10853b7dce5e90c5e828972db5500752e19d933
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# nmr-chemical-shift-ph-dependence-modeling

## Summary

Quantifies pH-dependent chemical-shift changes of metabolite peaks using a Henderson-Hasselbalch-derived transformation (equation 3), computing Δδ in ppm from δL, δHL, pH0 and pH1, for both 1D 1H peaks and projected 2D J-Res/COSY peak clusters in MetAssimulo 2.0 spectral simulation.

## When to use

Use when simulating 1D or 2D metabolomic 1H NMR spectra at a target pH different from the standard pH0 (usually 7.4) and you must reposition peaks that shift with pH.

## When NOT to use

- The metabolite's peaks do not shift with pH in the region of interest (Δδ effectively zero between pH0 and pH1) — applying the transform adds no realism
- You only need the standard spectrum at pH0 itself; no shift correction is required
- Target peaks are 13C or non-exchangeable nuclei without an ionizable group modelled by δL/δHL and a pKa in HMDB 5.0

## Inputs

- Metabolite peak parameters: delta_L and delta_HL chemical shifts (ppm) for acid and basic limits, from the metabolite configuration
- pKa values sourced from HMDB 5.0 (with Tredwell et al. normal distribution fallback where pKa is missing)
- pH0: pH of the standard spectrum (usually 7.4)
- Target pH (pH1) read from Input/parameters.txt used by apps/index.py
- 2D peak clusters identified on skyline/diagonal projections of J-Res and COSY spectra

## Outputs

- Per-peak Δδ shift values in ppm for all 1D 1H peaks
- Per-cluster Δδ shift values for 2D J-Res and COSY peak clusters
- peak_shift.py module output consumed by the spectral simulator

## How to apply

Load each metabolite's peak parameters — delta_L and delta_HL (peak positions in the acid and basic limits), the pKa (from HMDB 5.0, falling back to the Tredwell et al. normal distribution when missing), and pH0 (the standard-spectrum pH, usually 7.4) — together with the target pH from the parameters.txt file consumed by apps/index.py. Apply the Henderson-Hasselbalch-derived transformation of equation (3) to compute Δδ, the amount each peak is shifted in ppm between pH0 and pH1, which is the protonation-fraction-weighted change between δL and δHL. Implement this as a reusable function for single 1D 1H peaks, and a second function for peak clusters identified on skyline/diagonal projections of 2D J-Res and COSY spectra. Emit the computed shift values for all peaks and clusters (e.g. via peak_shift.py) so the simulator can place peaks at pH-corrected positions; validate by unit-testing a metabolite whose pKa and δ values exist in HMDB 5.0.

## Related tools

- **MetAssimulo 2.0** (Web application whose simulation pipeline applies equation (3) to shift 1D and 2D metabolite peaks to the target pH) — https://github.com/yanyan5420/MetAssimulo_2.git
- **MetAssimulo 2** (Hosts the peak_shift.py module implementing the reusable shift functions for 1D peaks and 2D clusters) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Language in which the shift computation functions are written and unit-tested) — https://github.com/yanyan5420/MetAssimulo_2.git

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Unit test reproduces the equation-3 Δδ for at least one metabolite whose pKa and delta values exist in HMDB 5.0
- Shift function outputs are validated as consistent with the equation-3 derivation in section 2.3.2 of the paper
- All metabolite peaks and 2D J-Res/COSY clusters receive a shift value before spectrum assembly (no unshifted peaks remain)
- At pH1 == pH0 (7.4) the computed Δδ is zero for every peak
- A second function confirms cluster-level shifts on skyline/diagonal projections match the sum of member-peak shifts

## Limitations

- pKa is not always available in HMDB 5.0; missing values must be drawn from the Tredwell et al. normal distribution fallback, adding uncertainty
- The standard pH0 is fixed (usually 7.4), so pH0-dependent bias cannot be tuned per run
- 2D shift application depends on prior correct identification of peak clusters on skyline/diagonal projections of J-Res and COSY spectra

## Evidence

- [methods] The peak shift is quantified by equation (3), which defines Δδ, the amount the peak is shifted in ppm: [evidence span withheld — anchor status: not_found]
- [results] Reproduce the pH-dependent shift for a known metabolite: [evidence span withheld — anchor status: not_found]
- [readme] Instructions for cloning the MetAssimulo_2 repository: "git clone https://github.com/yanyan5420/MetAssimulo_2.git
cd MetAssimulo_2"
