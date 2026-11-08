---
name: msms-formula-accuracy-benchmarking
description: Use when you have ranked candidate molecular formulas per MS/MS spectrum
  (from FSA or a comparable ranker) and need to quantify rank#1 accuracy, top-5 accuracy,
  and per-spectrum runtime against known metabolite ground truth.
when_to_use_negative:
- When the input spectra are already converted to a feature table or have known formulas
  for all peaks — this skill only evaluates candidate-formula ranking, it does not
  extract features or confirm formulas structurally.
- When the goal is to identify compounds by MS/MS spectral library match or in-silico
  structure prediction (e.g. using CFM-ID 4.0 or SIRIUS spectra) — this skill benchmarks
  formula-ranking accuracy, not structural identification.
- When compound scope includes elements outside C, H, O, N, S, P, or F, Cl, Br, I
  (e.g. metals, organometallics) — the reference formula list and the paper's validation
  did not cover such compositions.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FSA
  role: Runs the molecular-formula ranking over each MS/MS spectrum and records ranked
    candidate formulas and per-spectrum runtime that are then scored against ground
    truth
  repo: https://github.com/kslynn128171/FSA
- name: PubChem
  role: Supplies the reference molecular formulas and their monoisotopic molecular
    weights that constitute the formula-ranking candidate list (928,369 CHONSP entries;
    2,613,203 when halogens are added)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:020e72eb830152c9ccc500c6ae35c77795049f4aa85dce15c62bcee45dc77823
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# msms-formula-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metabolite molecular-formula ranking accuracy and runtime for FSA against a published reference (85.28% rank#1, 97.35% top-5, 0.024 s/spectrum over 30,690 MS/MS spectra), including stress-test variants such as expanding the PubChem reference list with halogen elements (F, Cl, Br, I).

## When to use

Use when you have ranked candidate molecular formulas per MS/MS spectrum (from FSA or a comparable ranker) and need to quantify rank#1 accuracy, top-5 accuracy, and per-spectrum runtime against known metabolite ground truth.

## When NOT to use

- When the input spectra are already converted to a feature table or have known formulas for all peaks — this skill only evaluates candidate-formula ranking, it does not extract features or confirm formulas structurally.
- When the goal is to identify compounds by MS/MS spectral library match or in-silico structure prediction (e.g. using CFM-ID 4.0 or SIRIUS spectra) — this skill benchmarks formula-ranking accuracy, not structural identification.
- When compound scope includes elements outside C, H, O, N, S, P, or F, Cl, Br, I (e.g. metals, organometallics) — the reference formula list and the paper's validation did not cover such compositions.

## Inputs

- MS/MS spectra with known metabolite ground-truth molecular formulas (e.g. METLIN-derived or annotated LC-MS/MS test datasets, such as the six non-lipid datasets)
- Reference molecular formula list with monoisotopic masses extracted from PubChem (928,369 entries for CHONSP-only scope; 2,613,203 entries when halogens F, Cl, Br, I are included)
- FSA configuration specifying the reference formula list and element scope

## Outputs

- Per-spectrum ranked list of candidate molecular formulas with their FSA rank positions
- Per-spectrum wall-clock runtime measurement
- Aggregate accuracy and runtime table: rank#1 accuracy (%), top-5 accuracy (%), and average runtime per spectrum (s) for each reference-scope variant

## How to apply

Run FSA over labeled MS/MS test datasets (e.g. the six non-lipid datasets used in the paper, derived from LC-MS/MS of 6792 metabolites and 30,690 spectra) with the PubChem-derived reference formula list of 928,369 CHONSP entries. For each spectrum, record the formula assigned to each ranked position and wall-clock time, then compute rank#1 accuracy, top-5 accuracy, and average runtime per spectrum. To stress-test scope, rebuild the reference list from a PubChem snapshot retaining formulas containing F, Cl, Br, or I alongside C, H, O, N, S, P (2,613,203 entries), rerun, and expect ~77.84% rank#1, 92.60% top-5, and ~0.16 s/spectrum (6.67x slower than the CHONSP-only 0.024 s baseline). Treat recomputed metrics as correct when they match the published values within a reasonable tolerance and the reference formula count matches exactly.

## Related tools

- **FSA** (Runs the molecular-formula ranking over each MS/MS spectrum and records ranked candidate formulas and per-spectrum runtime that are then scored against ground truth) — https://github.com/kslynn128171/FSA
- **PubChem** (Supplies the reference molecular formulas and their monoisotopic molecular weights that constitute the formula-ranking candidate list (928,369 CHONSP entries; 2,613,203 when halogens are added))

## Examples

```
python run_fsa.py --formulas pubchem_chonsp.csv --spectra test_msms.mzML --output ranked_formulas.csv --runtime-log fsa_runtime.log
```

## Evaluation signals

- CHONSP-only reference scope: recomputed rank#1 accuracy should reproduce 85.28%, top-5 accuracy 97.35%, and average runtime ~0.024 s/spectrum within a small tolerance across the 30,690-spectrum test set.
- Halogen-extended reference scope: recomputed rank#1 accuracy should be ~77.84%, top-5 accuracy ~92.60%, and runtime ~0.16 s/spectrum, with the reference formula count exactly matching 2,613,203 entries.
- The ratio between halogen-extended and CHONSP-only runtimes should be approximately 6.67x (0.16 s vs 0.024 s), serving as a sanity check on the runtime measurements.
- The accuracy and runtime comparison should be emitted as a structured table with one row per reference-scope variant, suitable for direct comparison against the published values.
- Per-spectrum FSA outputs must exist for every spectrum in each test dataset (no silent missing rows) before aggregate metrics are computed.

## Limitations

- FSA has not utilized structural information, so its rank#1 accuracy may not be competitive with state-of-the-art identification tools such as Buddy and SIRIUS.
- Accuracy and runtime depend on the currency of the reference formula database; the paper used a PubChem snapshot of 109,383,767 compounds, which is outdated relative to the 2022/11/27 version of 112,419,402 compounds.
- Runtime scales substantially with reference-list size — expanding from CHONSP (0.024 s/spectrum) to include halogens (0.16 s/spectrum) is a 6.67x slowdown that must be budgeted when benchmarking larger scopes.
- FSA has so far been validated on compounds composed of C, H, O, N, S, and P, and has only been extended to F, Cl, Br, I in the halogen stress test; compositions outside these elements are untested.

## Evidence
