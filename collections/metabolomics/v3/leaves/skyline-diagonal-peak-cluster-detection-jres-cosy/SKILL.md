---
name: skyline-diagonal-peak-cluster-detection-jres-cosy
description: Use when simulating 2D metabolomic 1H NMR spectra (J-Res or COSY) in
  MetAssimulo 2.0 and you need to locate each metabolite's peak clusters on skyline
  or diagonal projections so pH-driven peak shifts (equation 3, δL, δHL, pH0, pKa)
  can be applied per cluster rather than per 1D peak.
when_to_use_negative:
- Input is a real 1D 1H NMR spectrum with no 2D projection — use the per-peak 1D shift
  function instead.
- The metabolite of interest has no HMDB 5.0 entry and no usable pKa estimate, making
  equation (3) underdetermined.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Simulates the 2D J-Res and COSY spectra whose skyline/diagonal projections
    are scanned for peak clusters; supplies the peak parameters needed for the shift
    calculation.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Hosts the reusable cluster-detection and shift functions implemented inside
    MetAssimulo 2.0's web application.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:9ebd43588c55fbf3d6c94f09956a24f9a3a45113b50ded8db2c187980aa02c9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# skyline-diagonal-peak-cluster-detection-jres-cosy

## Summary

Identifies metabolite peak clusters on the skyline/diagonal projections of simulated 2D J-Res and COSY 1H NMR spectra so that pH-dependent chemical-shift displacements can be applied to each cluster using the MetAssimulo 2.0 Henderson-Hasselbalch transformation.

## When to use

Use when simulating 2D metabolomic 1H NMR spectra (J-Res or COSY) in MetAssimulo 2.0 and you need to locate each metabolite's peak clusters on skyline or diagonal projections so pH-driven peak shifts (equation 3, δL, δHL, pH0, pKa) can be applied per cluster rather than per 1D peak.

## When NOT to use

- Input is a real 1D 1H NMR spectrum with no 2D projection — use the per-peak 1D shift function instead.
- The metabolite of interest has no HMDB 5.0 entry and no usable pKa estimate, making equation (3) underdetermined.

## Inputs

- 2D 1H NMR spectrum (J-Res or COSY) simulated by MetAssimulo 2.0 for urine, blood, or CSF metabolite mixtures
- Peak parameter table per metabolite: delta_L, delta_HL, pH0, pH1, pKa (from HMDB 5.0; Tredwell et al. normal-distribution fallback when pKa is missing)
- Target pH value from the parameters.txt file consumed by apps/index.py

## Outputs

- List of detected peak clusters on skyline/diagonal projections, with their 2D coordinates
- Per-cluster pH-dependent chemical-shift values (Δδ) ready to be applied to the 2D peak map
- Updated 2D spectrum peak list consistent with the shifted 1D spectrum

## How to apply

First generate or load the 2D spectrum for the metabolite mixture (J-Res or COSY as produced by MetAssimulo 2.0, which covers urine, blood, and cerebral spinal fluid). Compute the skyline projection (or diagonal projection for cross-peaks) of the 2D matrix and scan it for contiguous signal regions that correspond to individual metabolite peak clusters. For each detected cluster, retrieve the associated peak parameters (delta_L, delta_HL, pH0, and pKa from HMDB 5.0; if pKa is missing, draw it from the Tredwell et al. normal distribution fallback) and the target pH read from Input/parameters.txt. Apply the same Henderson-Hasselbalch-derived shift used for 1D peaks — equation (3), Δδ computed from δL, δHL, pH0 and pH — to the whole cluster, emitting the shifted cluster coordinates so the 2D peak map stays consistent with the simulated 1D spectrum. The rationale is that cluster-level shifting preserves the multiplet/cross-peak structure of 2D spectra while reproducing the pH-dependent displacement the article quantifies in section 2.3.2.

## Related tools

- **MetAssimulo 2.0** (Simulates the 2D J-Res and COSY spectra whose skyline/diagonal projections are scanned for peak clusters; supplies the peak parameters needed for the shift calculation.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Hosts the reusable cluster-detection and shift functions implemented inside MetAssimulo 2.0's web application.) — https://github.com/yanyan5420/MetAssimulo_2.git

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Unit-test a known metabolite whose pKa and delta values exist in HMDB 5.0 and confirm the emitted Δδ matches the equation-3 derivation in section 2.3.2.
- Verify that every detected cluster on the skyline/diagonal projection has a corresponding shifted 2D coordinate in the output peak list (no unshifted clusters remain).
- Cross-check that the shifted 1D spectrum and the shifted 2D spectrum remain mutually consistent after applying the cluster-level shift.
- Confirm the pipeline runs end-to-end via apps/index.py -p Input/parameters.txt and the Dash app reports 'Dash is running on http://127.0.0.1:8060/'.

## Limitations

- pKa values are not universally available; when absent they must be sampled from the Tredwell et al. normal-distribution fallback, adding uncertainty to the shift.
- Shift application depends on accurate cluster detection; mis-detected clusters on the skyline/diagonal projection will displace the wrong set of 2D cross-peaks.
- The pH0 is usually fixed at 7.4 (the standard-spectrum pH), so spectra simulated at other baseline pH values must supply an explicit pH0 to avoid biased shifts.

## Evidence

- [methods] 2D cluster detection on skyline/diagonal projections of J-Res and COSY spectra: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 2D spectrum coverage: "Where MetAssimulo 1.0 only simulated 1D 1H spectra of human urine, MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid"
- [readme] Repository setup for running the web app: "git clone https://github.com/yanyan5420/MetAssimulo_2.git"
- [readme] Accessing the simulated spectra through the web app: "Dash is running on http://127.0.0.1:8060/"
