---
name: hmdb-metabolite-annotation-mapping
description: Use when you have a metabolite list (e.g. HMDB IDs from urine, blood,
  or CSF) and need per-peak values of delta_L, delta_HL, pKa and pH0 to drive Henderson-Hasselbalch
  peak-shift simulation.
when_to_use_negative:
- When you already have calibrated in-house shift/pKa values and do not need HMDB
  5.0 provenance
- When the metabolite of interest has no HMDB 5.0 entry and no valid Tredwell et al.
  fallback pKa distribution is available
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Web application whose metabolite configuration and parameters.txt drive the
    HMDB annotation lookup and downstream equation-3 shift computation.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language for the reusable peak_shift functions that consume
    the HMDB-derived parameters.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:636ff7a5e9e59d982e6ddbe3c52d555df7cd65937a7d2ec3fa86917478b31281
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hmdb-metabolite-annotation-mapping

## Summary

Maps metabolite identifiers to HMDB 5.0 annotations so that pH-dependent peak shifts can be computed with paper-traceable chemical-shift and pKa parameters.

## When to use

Use when you have a metabolite list (e.g. HMDB IDs from urine, blood, or CSF) and need per-peak values of delta_L, delta_HL, pKa and pH0 to drive Henderson-Hasselbalch peak-shift simulation.

## When NOT to use

- When you already have calibrated in-house shift/pKa values and do not need HMDB 5.0 provenance
- When the metabolite of interest has no HMDB 5.0 entry and no valid Tredwell et al. fallback pKa distribution is available

## Inputs

- Metabolite list with HMDB identifiers (from urine, blood, or CSF metabolite configuration)
- Target pH value from Input/parameters.txt
- HMDB 5.0 database records for delta_L, delta_HL and pKa

## Outputs

- Per-peak chemical-shift values (delta_L, delta_HL) and pKa
- Recomputed peak-shift values for each 1D 1H peak
- Peak-cluster shift values for 2D J-Res and COSY projections

## How to apply

Start from the metabolite configuration used by MetAssimulo 2.0's apps/index.py with the target pH read from Input/parameters.txt (pH0 is normally 7.4). For each metabolite, pull its HMDB 5.0 record to obtain the peak positions delta_L and delta_HL in the acid and basic limits and the pKa; the article states that HMDB 5.0 is the source for these peak parameters. Where pKa is missing from the curated record, draw it from the Tredwell et al. normal-distribution fallback so every peak still receives a shift value. The mapping output must be structured as reusable functions — one that applies the equation-3 Henderson-Hasselbalch transformation to single 1D 1H peaks, and a second that handles peak clusters identified on skyline/diagonal projections of 2D J-Res and COSY spectra — so the same annotation mapping feeds both 1D and 2D simulation.

## Related tools

- **MetAssimulo 2.0** (Web application whose metabolite configuration and parameters.txt drive the HMDB annotation lookup and downstream equation-3 shift computation.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language for the reusable peak_shift functions that consume the HMDB-derived parameters.)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The mapped pKa and delta values reproduce the Henderson-Hasselbalch shift of at least one metabolite whose parameters exist in HMDB 5.0.
- The output is consistent with the equation-3 derivation in section 2.3.2 of the article.
- Both the 1D peak function and the 2D cluster function run without error and produce shift values for all peaks/clusters in the metabolite configuration.
- Unit tests on a known metabolite confirm the shift magnitude matches the paper's expected pH-dependent displacement between pH0 and pH1.

## Limitations

- pKa values may be missing in HMDB 5.0, in which case they must be drawn from the Tredwell et al. normal-distribution fallback rather than a curated value.
- The mapping only covers metabolites represented in HMDB 5.0; metabolites absent from that release cannot be annotated this way.
- pH0 is assumed to be 7.4 (the standard-spectrum pH); using a different pH0 requires re-derivation of the shift.

## Evidence

- [methods] pKa drawn from HMDB 5.0: [evidence span withheld — anchor status: not_found]
- [methods] Equation-3 shift definition: [evidence span withheld — anchor status: not_found]
- [methods] Tredwell pKa fallback: [evidence span withheld — anchor status: not_found]
- [methods] Reusable functions for 1D and 2D: [evidence span withheld — anchor status: not_found]
- [methods] Validation via known metabolite: [evidence span withheld — anchor status: not_found]
