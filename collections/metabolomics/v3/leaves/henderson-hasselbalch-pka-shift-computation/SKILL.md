---
name: henderson-hasselbalch-pka-shift-computation
description: Use when simulating 1H NMR spectra at a target pH different from the
  standard pH0 = 7.4 and you need per-peak or per-cluster ppm shifts derived from
  δL, δHL and HMDB 5.0 pKa values.
when_to_use_negative:
- The target pH equals the standard pH0 = 7.4, so no shift needs to be computed
- The metabolite lacks any acid/base δL and δHL peak positions in the configuration,
  leaving equation (3) undefined
- You are simulating non-pH-sensitive peak features such as the blood protein background
  rather than metabolite peak positions
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetAssimulo 2.0
  role: Hosts the peak_shift.py module and parameters.txt configuration that carry
    out the equation (3) shift computation during spectral simulation
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: MetAssimulo 2
  role: Web app front-end whose simulation of 1D and 2D metabolomic 1H NMR spectra
    consumes the computed Δδ shifts
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Language in which the reusable shift functions for single peaks and projected
    2D clusters are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:d029dd4849fb520748ddd7098b0afc7934939a0731aeaf777fffd6bf5cb4cf6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# henderson-hasselbalch-pka-shift-computation

## Summary

Computes the pH-dependent chemical-shift displacement (Δδ, in ppm) for each metabolite 1H peak using the Henderson–Hasselbalch transformation of equation (3), which couples δL and δHL peak positions with pH0, pH1 and the metabolite pKa, as implemented in MetAssimulo 2.0's peak_shift.py.

## When to use

Use when simulating 1H NMR spectra at a target pH different from the standard pH0 = 7.4 and you need per-peak or per-cluster ppm shifts derived from δL, δHL and HMDB 5.0 pKa values.

## When NOT to use

- The target pH equals the standard pH0 = 7.4, so no shift needs to be computed
- The metabolite lacks any acid/base δL and δHL peak positions in the configuration, leaving equation (3) undefined
- You are simulating non-pH-sensitive peak features such as the blood protein background rather than metabolite peak positions

## Inputs

- Per-metabolite peak parameter set: delta_L, delta_HL, pH0 (typically 7.4), pH1, pKa from HMDB 5.0
- Target pH value read from the parameters.txt file used by apps/index.py
- Peak-cluster coordinates identified on skyline/diagonal projections of 2D J-Res or COSY spectra

## Outputs

- Δδ peak-shift values in ppm for every 1D 1H peak, written via peak_shift.py
- Δδ shift values for each peak cluster on 2D J-Res/COSY projections
- Unit-test output reproducing the pH-dependent shift for a metabolite with known HMDB 5.0 parameters

## How to apply

Load each metabolite's peak parameters — delta_L, delta_HL (the chemical shifts in the acid and basic limits), pH0 (usually 7.4), target pH1, and pKa sourced from HMDB 5.0 — from the metabolite configuration and the parameters.txt file consumed by apps/index.py. Apply equation (3), the Henderson–Hasselbalch-derived transformation, to compute Δδ, the amount each peak is shifted in ppm between pH0 and pH1; where a pKa is missing, fall back to the Tredwell et al. normal-distribution draw. Implement the computation as a reusable function so it operates on single 1D 1H peaks, and write a second function that applies the same transformation to peak clusters identified on skyline/diagonal projections of 2D J-Res and COSY spectra. Emit the shift values into peak_shift.py's output and validate against a known metabolite whose pKa and δ values exist in HMDB 5.0, confirming consistency with the equation-3 derivation in section 2.3.2.

## Related tools

- **MetAssimulo 2.0** (Hosts the peak_shift.py module and parameters.txt configuration that carry out the equation (3) shift computation during spectral simulation) — https://github.com/yanyan5420/MetAssimulo_2.git
- **MetAssimulo 2** (Web app front-end whose simulation of 1D and 2D metabolomic 1H NMR spectra consumes the computed Δδ shifts) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Language in which the reusable shift functions for single peaks and projected 2D clusters are implemented)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The module runs and reproduces the pH-dependent shift for at least one metabolite whose pKa and δ values exist in HMDB 5.0
- Output Δδ values agree with the equation-3 derivation given in section 2.3.2 of the article
- Shift functions return identical results when applied to a single 1D peak and to the equivalent peak cluster on a 2D projection
- Missing pKa entries are filled from the Tredwell et al. normal-distribution fallback rather than left null

## Limitations

- Requires both δL and δHL peak positions; metabolites without a resolved acid/basic pair cannot be shifted
- pKa values absent from HMDB 5.0 rely on a distributional fallback, introducing uncertainty in the computed Δδ
- The computation is calibrated against the standard spectrum pH0 of 7.4; deviating from this reference changes the baseline definition

## Evidence

- [methods] Equation (3) defines Δδ, the amount the peak is shifted in ppm, computed from δL and δHL (the peak positions in the acid and basic limits) and pH0: [evidence span withheld — anchor status: not_found]
- [methods] Load the peak parameters for each metabolite (delta_L, delta_HL, pH0, pH1, and pKa from HMDB 5.0): [evidence span withheld — anchor status: not_found]
- [methods] where pKa is missing, draw it from the Tredwell et al. normal distribution fallback: [evidence span withheld — anchor status: not_found]
- [methods] applied to single 1D 1H peaks and, in a second function, to peak clusters identified on skyline/diagonal projections of 2D J-Res and COSY spectra: [evidence span withheld — anchor status: not_found]
- [methods] the pH of the standard spectrum, usually 7.4: [evidence span withheld — anchor status: not_found]
