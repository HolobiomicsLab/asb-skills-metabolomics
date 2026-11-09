---
name: relative-error-concentration-assessment
description: Use when estimated mixture proportions (e.g. πi from Magnetstein's optimal-transport
  deconvolution) must be scored against gravimetrically known ground-truth composition,
  or when comparing quantifiers like Magnetstein, MANIQ, and ACD/TP on the same mixture.
when_to_use_negative:
- No gravimetrically known composition is available for the mixture, so neither RE
  nor AvRE can be computed against ground truth
- Only a single-component (neat) sample is analyzed — relative error across mixture
  components is undefined
- Comparing tools on spectra where the library contains the exact reference spectrum
  and trueness claims from Table S1 do not apply to your instrument or field strength
  without recalibration
edam_operation: http://edamontology.org/operation_3799
edam_topics: []
tools:
- name: Magnetstein
  role: Produces the estimated relative concentrations π̂_i whose trueness is scored
    as RE and AvRE against gravimetric ground truth
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: Peak-picking competitor whose AvRE/RE baselines (e.g. AvRE 28.8% in Experiment
    3) are used for benchmark comparison
- name: ACD/Spectrus-Target-Protected Profiling
  role: Peak-picking competitor whose AvRE/RE baselines (e.g. AvRE 56.9% in Experiment
    3) are used for benchmark comparison
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:e09f7270712d04eb01af5f9e2e0f7f5d24d71f40d1dfedd039f36f7c8946811c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - ACD/Spectrus-Target-Protected Profiling
  - MANIQ
  - Magnetstein
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# relative-error-concentration-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses NMR mixture-component quantification trueness by computing per-component relative error (RE = |estimated − gravimetric| / gravimetric × 100%) and the average relative error (AvRE) over components, benchmarked against values reported for Magnetstein's Experiments 1–5.

## When to use

Use when estimated mixture proportions (e.g. πi from Magnetstein's optimal-transport deconvolution) must be scored against gravimetrically known ground-truth composition, or when comparing quantifiers like Magnetstein, MANIQ, and ACD/TP on the same mixture.

## When NOT to use

- No gravimetrically known composition is available for the mixture, so neither RE nor AvRE can be computed against ground truth
- Only a single-component (neat) sample is analyzed — relative error across mixture components is undefined
- Comparing tools on spectra where the library contains the exact reference spectrum and trueness claims from Table S1 do not apply to your instrument or field strength without recalibration

## Inputs

- Estimated relative concentrations π̂_i of mixture components (e.g. from Magnetstein, MANIQ, or ACD/TP)
- Gravimetrically known ground-truth relative composition of the mixture (per component)
- Experiment identifier / scenario label (Experiment 1–5) and κmixture, κcomponents settings used for the quantification run

## Outputs

- Per-component relative error (RE) values, e.g. RE for α-pinene and limonene
- Average relative error (AvRE) over components for the run
- Benchmark comparison table against reported Magnetstein / ACD/TP / MANIQ values (cf. Table S1), e.g. experiment3_quantification.csv

## How to apply

For each mixture component i, compute RE_i = |π̂_i − π_true_i| / π_true_i × 100% using the gravimetrically prepared composition as ground truth, then average the per-component REs into AvRE. Apply this separately per experiment scenario, e.g. Experiment 1 (two components, ~1:10 ratio), Experiment 3 (α-pinene and limonene on a Magritek 43 MHz benchtop 43 MHz 1H NMR spectrum), Experiment 4 (inhomogeneous field, lineshape distortion), and Experiment 5 (36.7 °C mixture vs. 22.27 °C library, shifted peaks). When benchmarking Magnetstein, run the optimal-transport deconvolution with tuned penalties (κcomponents higher than κmixture for Experiment 3) and compare the resulting AvRE against the reference values: AvRE 0.0% for Experiment 1, 9.0% (RE α-pinene 0.0%, limonene 33.3%) for Experiment 3, and 3.6% for Experiments 4 and 5, with competitor baselines ACD/TP (18.3%, 56.9%, 39.7%, 35.8%) and MANIQ (12.2%, 28.8%, 34.9%, 7.2%). Cross-check tuned-parameter results against Table S1, which reports outcomes with optimal κmixture and κcomponents. When validating a reproduced workflow, accept the run only if the recomputed RE/AvRE values match the reported ones within a reasonable tolerance.

## Related tools

- **Magnetstein** (Produces the estimated relative concentrations π̂_i whose trueness is scored as RE and AvRE against gravimetric ground truth) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (Peak-picking competitor whose AvRE/RE baselines (e.g. AvRE 28.8% in Experiment 3) are used for benchmark comparison)
- **ACD/Spectrus-Target-Protected Profiling** (Peak-picking competitor whose AvRE/RE baselines (e.g. AvRE 56.9% in Experiment 3) are used for benchmark comparison)

## Examples

```
python -c "est=[0.9,0.1]; tru=[0.9,0.07]; re=[abs(e-t)/t*100 for e,t in zip(est,tru)]; avre=sum(re)/len(re); print(re, avre)"
```

## Evaluation signals

- Experiment 1 reproduction: AvRE = 0.0% for Magnetstein versus reported competitor AvRE of 18.3% (ACD/TP) and 12.2% (MANIQ)
- Experiment 3 reproduction: AvRE = 9.0% with RE for α-pinene = 0.0% and RE for limonene = 33.3%; competitor failures at AvRE 56.9% (ACD/TP) and 28.8% (MANIQ)
- Lineshape-distortion and peak-shift scenarios (Experiments 4 and 5): reproduced AvRE = 3.6% matches the reported Magnetstein trueness
- Output artifacts (e.g. experiment3_quantification.csv) contain per-component REs, AvRE, and the κmixture/κcomponents settings, consistent with Table S1 optimal-parameter results
- Sensitivity check: AvRE should not blow up at the chosen κ settings — remember that small κmixture drives the smaller component into vortex ω

## Limitations

- Small κmixture values cause Magnetstein to assign the smaller component to auxiliary point ω, producing large relative error — RE/AvRE degradation there reflects parameter choice, not quantifier trueness
- Competitor failures can be component-specific: in Experiment 3 ACD/TP and MANIQ failed to estimate α-pinene and/or limonene (RE α-pinene 125.0%/100.0%, limonene 100.0%/11.1%), so a single-component RE can dominate the AvRE comparison
- Reported RE values are exact benchmark targets (e.g. limonene RE 33.3%); reproduced values will only approximate them, and tolerance for 'matching' must be stated explicitly
- Absolute concentrations require the external-standard workflow (adding a known-concentration spectrum); RE/AvRE alone only score relative proportions

## Evidence
