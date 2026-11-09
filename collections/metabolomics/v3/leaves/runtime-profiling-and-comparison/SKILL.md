---
name: runtime-profiling-and-comparison
description: Use when comparing computational cost of spectrum-prediction or other
  molecular-modeling methods, to verify a claimed runtime speedup on your own hardware
  before adopting a model for large-scale prediction.
when_to_use_negative:
- The goal is predictive accuracy (recall-at-10, library matching) rather than computational
  cost — use an accuracy-benchmarking skill instead.
- The method under test requires quantum-chemical simulation outputs (fragment trajectories,
  isotope peaks) where per-molecule runtimes of 1000 s are inherent, and speed is
  not the comparison target.
- Only a handful of molecules need predicting once — profiling overhead outweighs
  the benefit of a timing benchmark.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3418
- http://edamontology.org/topic_3361
tools:
- name: Tensorflow
  role: Constructs and runs the NEIMS MLP model whose per-molecule inference time
    is being profiled
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Converts input molecules to additive ECFP fingerprints (length 4096, radius
    2) fed to the model during timing runs
  repo: github.com/brain-research/deep-molecular-massspec
- name: NEIMS
  role: The lightweight neural network model whose 5 ms/molecule prediction speed
    is benchmarked against CFM-EI and QCEIMS
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:020fddf8996d240694d7b7506d5544d14cfcbda401bc60515dafef3f0b077a7d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3418
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# runtime-profiling-and-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmarks the per-molecule wall-clock time of NEIMS spectrum prediction (~5 ms per molecule) against competing methods (CFM-EI, first-principles QCEIMS) to quantify orders-of-magnitude speed advantages for library augmentation.

## When to use

Use when comparing computational cost of spectrum-prediction or other molecular-modeling methods, to verify a claimed runtime speedup on your own hardware before adopting a model for large-scale prediction.

## When NOT to use

- The goal is predictive accuracy (recall-at-10, library matching) rather than computational cost — use an accuracy-benchmarking skill instead.
- The method under test requires quantum-chemical simulation outputs (fragment trajectories, isotope peaks) where per-molecule runtimes of 1000 s are inherent, and speed is not the comparison target.
- Only a handful of molecules need predicting once — profiling overhead outweighs the benefit of a timing benchmark.

## Inputs

- Trained NEIMS TensorFlow model checkpoint (massspec_weights.zip from github.com/brain-research/deep-molecular-massspec)
- Small-molecule inputs (e.g. .sdf or SMILES) from the NIST 2017 Mass Spectral Library
- Published runtime figures for CFM-EI (up to ~10 min/molecule) and QCEIMS-style first-principles methods (>=1000 s/molecule)

## Outputs

- Average per-molecule prediction time for NEIMS (milliseconds)
- Timing comparison table (runtime_comparison.csv) with NEIMS vs CFM-EI vs QCEIMS
- Orders-of-magnitude speedup statement grounded in measured timings

## How to apply

Load the pretrained NEIMS TensorFlow checkpoint (MLP with seven layers of 2000 nodes) from the deep-molecular-massspec repository and convert input molecules from the NIST 2017 Mass Spectral Library to additive ECFP fingerprints using RDKit with fingerprint length 4096 and radius 2. Run inference through the bidirectional (forward + reverse + gate) prediction head while timing each per-molecule prediction, then compute the average prediction time per molecule — expected on the order of 5 ms. Tabulate the measured NEIMS runtime alongside published runtimes of the competing methods: CFM-EI, which can take up to 10 min for large molecules, and QCEIMS-style first-principles methods, which require 1000 s or more per molecule. Write the timing comparison to a results file (e.g. runtime_comparison.csv) and sanity-check that the measured average remains in the millisecond range on CPU-class hardware.

## Related tools

- **Tensorflow** (Constructs and runs the NEIMS MLP model whose per-molecule inference time is being profiled) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Converts input molecules to additive ECFP fingerprints (length 4096, radius 2) fed to the model during timing runs) — github.com/brain-research/deep-molecular-massspec
- **NEIMS** (The lightweight neural network model whose 5 ms/molecule prediction speed is benchmarked against CFM-EI and QCEIMS) — github.com/brain-research/deep-molecular-massspec

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Measured average per-molecule prediction time is on the order of milliseconds (~5 ms), several orders of magnitude below the ~10-min CFM-EI and >=1000 s QCEIMS reference values.
- Timing runs use the full bidirectional prediction head (forward + reverse + gate), so per-molecule cost covers all three inference passes, not a single forward pass.
- Input fingerprints are ECFP of length 4096 with radius 2 — different fingerprint settings invalidate runtime comparability with the published benchmark.
- Timing is measured on CPU-class hardware consistent with the paper's claim of ~1 million molecules in 90 min on CPU.
- Output table contains all three methods (NEIMS, CFM-EI, first-principles/QCEIMS) with comparable per-molecule time units (ms vs min vs s converted consistently).

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments, so runtime comparisons do not cover isotope-peak modeling cost.
- The 5 ms figure reflects the published MLP architecture (seven layers of 2000 nodes); substantially different model sizes or GPU hardware change the measured runtime.
- Runtime comparisons against CFM-EI and QCEIMS rely on published values rather than identical hardware, so absolute speedup factors are approximate.

## Evidence

- [readme] README model weights download for prediction runs: [evidence span withheld — anchor status: external_doc]
