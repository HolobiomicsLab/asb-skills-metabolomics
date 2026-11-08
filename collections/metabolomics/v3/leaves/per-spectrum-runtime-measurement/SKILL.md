---
name: per-spectrum-runtime-measurement
description: Use when evaluating or comparing FSA formula-ranking throughput across
  reference-list scopes or datasets, where rank#1/top-5 accuracy must be reported
  alongside an average seconds-per-spectrum runtime metric.
when_to_use_negative:
- Input is already a batch of runtime timings with no spectra to process — this skill
  measures runtime, it does not summarize pre-existing timing logs.
- The goal is formula accuracy evaluation only and runtime is irrelevant to the comparison
  being reported.
- The formula-ranking tool is not FSA (e.g. SIRIUS or Buddy), since the 0.024 s /
  0.16 s per-spectrum baselines reported here are FSA-specific.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0089
tools:
- name: FSA
  role: Formula-ranking tool whose per-spectrum execution is timed; reports candidate
    molecular formulas whose wall-clock cost is measured.
  repo: https://github.com/kslynn128171/FSA
- name: PubChem
  role: Source of the chemical formulas and monoisotopic molecular weights that make
    up the reference list; its size (928,369 → 2,613,203 entries) drives the runtime
    change being measured.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:50b24ec56417a3e22fbed0f836e79119bc3c8619a7fd54dbcbf89c0581474981
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - PubChem
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-spectrum-runtime-measurement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Measures FSA's per-spectrum processing time on MS/MS datasets by recording wall-clock time for each spectrum while running formula ranking, so accuracy gains from extended reference lists can be weighed against runtime cost.

## When to use

Use when evaluating or comparing FSA formula-ranking throughput across reference-list scopes or datasets, where rank#1/top-5 accuracy must be reported alongside an average seconds-per-spectrum runtime metric.

## When NOT to use

- Input is already a batch of runtime timings with no spectra to process — this skill measures runtime, it does not summarize pre-existing timing logs.
- The goal is formula accuracy evaluation only and runtime is irrelevant to the comparison being reported.
- The formula-ranking tool is not FSA (e.g. SIRIUS or Buddy), since the 0.024 s / 0.16 s per-spectrum baselines reported here are FSA-specific.

## Inputs

- MS/MS spectra (e.g. mzML or mgf files from LC-MS/MS metabolomics datasets)
- FSA formula reference list built from PubChem (e.g. 928,369 CHONSP formulas or 2,613,203 halogen-extended formulas)

## Outputs

- Per-spectrum wall-clock times
- Average processing time per spectrum (e.g. 0.024 s CHONSP-only; 0.16 s with halogens)
- Runtime comparison table paired with rank#1/top-5 accuracy metrics

## How to apply

While running FSA over a set of MS/MS spectra (e.g. the six non-lipid test datasets), record the wall-clock time consumed for each individual spectrum as candidate molecular formulas are generated and ranked. After the run, compute the average processing time per spectrum; in the CHONSP-only configuration with a 928,369-formula PubChem reference list this averaged 0.024 s, whereas the halogen-extended configuration (2,613,203 formulas including F, Cl, Br, I) averaged 0.16 s per spectrum. Report this runtime alongside rank#1 and top-5 accuracy so that speed–accuracy trade-offs of extending FSA's element scope (a 6.67x slowdown in the halogen case) are visible. Use the recorded times to verify that the implementation reproduces the paper's reported throughput before trusting downstream accuracy comparisons.

## Related tools

- **FSA** (Formula-ranking tool whose per-spectrum execution is timed; reports candidate molecular formulas whose wall-clock cost is measured.) — https://github.com/kslynn128171/FSA
- **PubChem** (Source of the chemical formulas and monoisotopic molecular weights that make up the reference list; its size (928,369 → 2,613,203 entries) drives the runtime change being measured.)

## Examples

```
FSA --reference-list pubchem_halogen_extended.csv --msms spectra.mgf --time-per-spectrum
```

## Evaluation signals

- Average per-spectrum runtime for the CHONSP-only PubChem reference list (928,369 formulas) should land near 0.024 s per spectrum.
- Average per-spectrum runtime for the halogen-extended reference list (2,613,203 formulas with F, Cl, Br, I) should land near 0.16 s per spectrum (≈6.67x the CHONSP run).
- The halogen-extended run should still rank the correct formula first in 77.84% of the six non-lipid datasets and top-5 in 92.60%, confirming accuracy was not sacrificed for the runtime reported.
- Reference formula count must match the configured scope: 928,369 (CHONSP) or 2,613,203 (halogen-extended) before timings are trusted.
- Timing must be recorded per spectrum, not per batch, so the average is a true per-spectrum metric.

## Limitations

- The CHONSP configuration's 0.024 s per-spectrum runtime was validated only on datasets containing compounds composed of C, H, O, N, S, and P; other element scopes may shift the timing.
- Runtime grows substantially (6.67x) when the reference list is expanded with halogens, so throughput claims must always state which element scope they apply to.
- The 0.024 s figure is FSA-specific; comparable tools such as SIRIUS took 5 h 41 min for 5518 spectra, so timings are not directly interchangeable across tools.
- FSA does not yet utilize structural information, so fast runtime does not guarantee state-of-the-art identification accuracy.
- FSA's PubChem snapshot may be outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), which affects the reference-list size that drives runtime.

## Evidence

- [other] Halogen-extended FSA runtime and accuracy across six non-lipid datasets: [evidence span withheld — anchor status: not_found]
