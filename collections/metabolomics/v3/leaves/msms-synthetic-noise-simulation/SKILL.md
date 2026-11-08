---
name: msms-synthetic-noise-simulation
description: 'Use when validating a denoising pipeline on clean query MS/MS spectra:
  synthetic chemical noise (from formula_db) and electronic noise must be injected
  before running spectral_denoising and scoring recovery.'
when_to_use_negative:
- The spectra are already experimental/noisy field data (e.g. real Orbitrap Astral
  plasma runs) — use Denoising Search rather than fabricating synthetic noise
- The goal is library matching/annotation rather than benchmarking denoising robustness
  — sd.denoising_search is the appropriate entry point
- Compounds have insufficient fragmentation (fewer than two fragment ions, spectral
  entropy <0.5), where recovery scoring is unreliable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spectral Denoising
  role: Core package providing sd.read_msp, noise generation functions, and sd.spectral_denoising
    used in the contamination-and-recovery test
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: spectral_denoising
  role: The denoising function under test (electronic_denoising + formula_denoising)
    applied to each contaminated spectrum
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Underlies sd.entropy_similairty scoring of contaminated and denoised spectra
    against ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:2743c33e4bcf351917a3a9a11f221ab1f38223d078a4014d581eb6c49ec2ef8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - Spectral Denoising
  - ms_entropy
  - numpy
  - pandas
  - spectral_denoising
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# msms-synthetic-noise-simulation

## Summary

Simulates artificial chemical and electronic ('grass') noise contamination on MS/MS spectra to stress-test spectral denoising algorithms, then quantifies recovery via entropy similarity against the ground-truth originals.

## When to use

Use when validating a denoising pipeline on clean query MS/MS spectra: synthetic chemical noise (from formula_db) and electronic noise must be injected before running spectral_denoising and scoring recovery.

## When NOT to use

- The spectra are already experimental/noisy field data (e.g. real Orbitrap Astral plasma runs) — use Denoising Search rather than fabricating synthetic noise
- The goal is library matching/annotation rather than benchmarking denoising robustness — sd.denoising_search is the appropriate entry point
- Compounds have insufficient fragmentation (fewer than two fragment ions, spectral entropy <0.5), where recovery scoring is unreliable

## Inputs

- Query spectra .msp file (e.g. sample_data/query_spectra.msp) with peaks, smiles/formula, adduct and precursor_mz columns
- Formula_db reference (sorted by mass) for drawing chemical-noise m/z values
- Clean ground-truth peak arrays (numpy float32 [m/z, intensity]) for each query spectrum

## Outputs

- Contaminated spectra (original + chemical noise + electronic noise)
- Denoised peak arrays returned by sd.spectral_denoising
- Entropy similarity scores of contaminated vs. denoised spectra against ground truth (with precursor region excluded via pmz)

## How to apply

Load query spectra with sd.read_msp from an .msp file (e.g. sample_data/query_spectra.msp) plus the sorted formula_db used for chemical-noise generation. Generate chemical noise with sd.noise.generate_chemical_noise, drawing m/z values randomly from the formula_db across 50 to precursor m/z and intensities from a Poisson distribution at lambda=50, then merge into each true spectrum via sd.noise.add_noise. Next generate electronic ('grass') noise with sd.noise.generate_noise, drawing m/z uniformly across 50 to pmz with Poisson intensities at lambda=5, and add it to the spectra. Run sd.spectral_denoising on each contaminated spectrum — it sequentially applies electronic_denoising (removes ions with identical intensities greater than 4) and formula_denoising (keeps ions explainable by a chemically plausible subformula loss, validated with Seven Golden Rules checks). Score both contaminated and denoised spectra against the ground truth using sd.entropy_similairty with the precursor region removed via the pmz argument; robustness must hold even when noise-ion intensities exceed true fragment-ion intensities by more than 150-fold.

## Related tools

- **Spectral Denoising** (Core package providing sd.read_msp, noise generation functions, and sd.spectral_denoising used in the contamination-and-recovery test) — https://github.com/FanzhouKong/spectral_denoising
- **spectral_denoising** (The denoising function under test (electronic_denoising + formula_denoising) applied to each contaminated spectrum) — https://github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Underlies sd.entropy_similairty scoring of contaminated and denoised spectra against ground truth)

## Examples

```
peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct); print(f'entropy similarity: {entropy_similairty(peak_denoised, peak, pmz = pmz):.2f}')
```

## Evaluation signals

- Contaminated spectra should score entropy similarity ~0.33–0.47 against the originals; denoised spectra must recover ~0.99–1.00
- Robustness must hold even when noise-ion intensities exceed true fragment-ion intensities by more than 150-fold
- Electronic denoising must remove ions with identical intensities greater than 4 (the NIST23-derived threshold)
- Chemical denoising must keep only ions explainable by a chemically plausible subformula loss validated by Seven Golden Rules (ignoring LEWIS and SENIOR checks)
- Spectral entropy should drop after denoising (published example: S = 3.3 contaminated vs. S = 1.9 denoised)

## Limitations

- Not all added noise ions are removed because chemical plausibility checks are limited to the Seven Golden Rules method without considering molecular connectivity
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Python version must be >=3.8 and <3.13 (RDkit lacks a Python 3.13-compatible distribution); batch mode must be wrapped in main() because it uses multiprocessing

## Evidence

- [readme] Noise generation workflow: [evidence span withheld — anchor status: external_doc]
- [readme] Denoising invocation: "peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)"
- [methods] Electronic denoising threshold: "The ``electronic_denoising`` function removes ions with identical intensities greater than 4."
- [abstract] Robustness finding: "even with >150-fold higher intensity of noise ions than true fragment ions"
- [discussion] Chemical-check limitation: "chemical plausibility checks were limited to the algorithms embedded in the Seven Golden Rules method32, without considering"
