---
name: msms-spectral-data-format-conversion
description: Use when raw benchmark MS/MS files must be converted to mgf with title,
  precursor_mz, precursor_type and collision_energy fields so FIDDLE (or SIRIUS/BUDDY
  baselines) can consume them for formula-identification evaluation.
when_to_use_negative:
- Input spectra are already in mgf format with all required fields — rerunning conversion
  only risks corrupting metadata
- The task is internal FIDDLE training on NIST20/NIST23/MoNA/GNPS data rather than
  external CASMI/EMBL-MCF benchmark evaluation
- Spectra fall outside the small-molecule scope (precursor m/z > 1500 or fewer than
  five peaks), which the pipeline explicitly excludes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIDDLE
  role: Downstream consumer of the converted mgf files; run_fiddle.py reads mgf records
    with title, precursor_mz, precursor_type, and collision_energy to predict molecular
    formulas
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI/CLI distribution of FIDDLE that accepts MGF files as prediction input
  repo: https://github.com/josiehong/msfiddle
- name: SIRIUS
  role: Baseline tool run on the same converted benchmark spectra; its outputs are
    formatted as output_sirius.csv for the combined-method comparison
  repo: https://github.com/sirius-ms/sirius
- name: BUDDY
  role: Baseline tool run on the same converted benchmark spectra; its outputs are
    formatted as output_buddy.csv for the combined-method comparison
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:c4593ede30819e2a204a62611299d85b2c4a2541dedde2d0885c21fbd96585d7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - MIST-CF
  - Q-TOF
  - SIRIUS
  - msfiddle
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# MS/MS spectral data format conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts external benchmark tandem MS spectra (CASMI 2016/2017, EMBL-MCF 2.0) into MGF format with the metadata fields FIDDLE's run_fiddle.py pipeline requires, and applies the article's spectrum-level filters before evaluation.

## When to use

Use when raw benchmark MS/MS files must be converted to mgf with title, precursor_mz, precursor_type and collision_energy fields so FIDDLE (or SIRIUS/BUDDY baselines) can consume them for formula-identification evaluation.

## When NOT to use

- Input spectra are already in mgf format with all required fields — rerunning conversion only risks corrupting metadata
- The task is internal FIDDLE training on NIST20/NIST23/MoNA/GNPS data rather than external CASMI/EMBL-MCF benchmark evaluation
- Spectra fall outside the small-molecule scope (precursor m/z > 1500 or fewer than five peaks), which the pipeline explicitly excludes

## Inputs

- CASMI 2016 benchmark spectra (raw/native format)
- CASMI 2017 benchmark spectra (raw/native format)
- EMBL-MCF 2.0 benchmark spectra (raw/native format)
- Precursor metadata: precursor_mz, precursor_type, collision_energy, instrument type (Orbitrap vs Q-TOF)

## Outputs

- Benchmark spectra in mgf format with title, precursor_mz, precursor_type, and collision_energy fields, filtered to m/z (0, 1500] and ≥5 peaks
- Instrument-tagged mgf sets ready for Orbitrap (5 ppm) or Q-TOF (10 ppm) evaluation

## How to apply

Export or rewrite each benchmark spectrum into mgf format, ensuring every record carries title, precursor_mz, precursor_type, and collision_energy fields as expected by run_fiddle.py in the FIDDLE repository. While converting, apply the article's spectrum filters: keep precursor m/z in (0, 1500], drop peaks below 50 m/z, exclude spectra with fewer than five peaks, and retain only common precursor adduct types (e.g., [M+H]+, [M+Na]+, [2M+H]+). Convert records into the correct instrument class — Orbitrap spectra are evaluated with a 5 ppm precursor tolerance and Q-TOF spectra with 10 ppm, so instrument metadata must be preserved or tagged per record. The same converted mgf files are then reused for FIDDLE, SIRIUS, and BUDDY so that top-1/top-5 accuracy comparisons operate on identical inputs.

## Related tools

- **FIDDLE** (Downstream consumer of the converted mgf files; run_fiddle.py reads mgf records with title, precursor_mz, precursor_type, and collision_energy to predict molecular formulas) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI/CLI distribution of FIDDLE that accepts MGF files as prediction input) — https://github.com/josiehong/msfiddle
- **SIRIUS** (Baseline tool run on the same converted benchmark spectra; its outputs are formatted as output_sirius.csv for the combined-method comparison) — https://github.com/sirius-ms/sirius
- **BUDDY** (Baseline tool run on the same converted benchmark spectra; its outputs are formatted as output_buddy.csv for the combined-method comparison) — https://github.com/Philipbear/msbuddy

## Examples

```
bash ./running_scripts/download_models.sh && bash running_scripts/experiments_test_benchmark.sh
```

## Evaluation signals

- Every output mgf record contains title, precursor_mz, precursor_type, and collision_energy fields
- All converted spectra have precursor m/z within (0, 1500]; out-of-range records were dropped or excluded
- No retained spectrum has fewer than five peaks and no retained peak lies below 50 m/z
- Instrument class per spectrum is identifiable so the correct tolerance (5 ppm Orbitrap, 10 ppm Q-TOF) is applied downstream
- Rerunning run_fiddle.py on the converted mgf reproduces benchmark accuracies close to the reported values (FIDDLE alone 75.1% positive / 66.2% negative top-5)

## Limitations

- Spectra with fewer than five peaks are excluded as potentially unreliable, so conversion cannot recover them for evaluation
- Accuracy after conversion-dependent evaluation varies across experimental conditions, especially those with less training data
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases; conversion does not resolve mixed spectra

## Evidence

- [readme] Required mgf fields for benchmark conversion: [evidence span withheld — anchor status: not_found]
- [readme] msfiddle accepts MGF files: [evidence span withheld — anchor status: external_doc]
