---
name: compound-fragmentation-entropy-quality-screening
description: Use when preparing MS/MS spectra (e.g. dilution-series or reference-library
  .msp files) for denoising or spectral matching, and you must exclude compounds whose
  spectra lack sufficient genuine fragment ions to support similarity comparisons.
when_to_use_negative:
- When the goal is to denoise or annotate spectra rather than screen reference compounds
  — low-entropy spectra can still be processed, they are only excluded from the benchmark
  analysis
- When spectra have already been denoised and filtered; applying the screen post-denoising
  would confound the entropy calculation with noise removal
- When analyzing non-Orbitrap MS/MS data without retuned parameters, since the method
  is optimized for Orbitrap instruments
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: Python package whose read_msp, sanitize/spectral_operations functions and
    spectral_entropy metric implement the fragment-ion and entropy screening
  repo: https://github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:6b9113fee07ac9647a88a98686c169149903ef3473b1c433374ed869c84aeb0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - ClassyFire
  - MS Reduce
  - Spectral Denoising
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# compound-fragmentation-entropy-quality-screening

## Summary

Pre-filters compounds for MS/MS benchmarking or annotation by computing spectral entropy and fragment-ion counts, excluding compounds with fewer than two fragment ions (spectral entropy <0.5) that are too poorly fragmented to denoise or match reliably.

## When to use

Use when preparing MS/MS spectra (e.g. dilution-series or reference-library .msp files) for denoising or spectral matching, and you must exclude compounds whose spectra lack sufficient genuine fragment ions to support similarity comparisons.

## When NOT to use

- When the goal is to denoise or annotate spectra rather than screen reference compounds — low-entropy spectra can still be processed, they are only excluded from the benchmark analysis
- When spectra have already been denoised and filtered; applying the screen post-denoising would confound the entropy calculation with noise removal
- When analyzing non-Orbitrap MS/MS data without retuned parameters, since the method is optimized for Orbitrap instruments

## Inputs

- MS/MS spectra of reference compounds (e.g. dilution-series .msp files read with sd.read_msp)
- Per-spectrum peaks arrays (mass/intensity pairs) for entropy computation
- Precursor m/z and adduct metadata for downstream denoising

## Outputs

- Filtered set of compounds/spectra with >=2 fragment ions (spectral entropy >=0.5)
- List of excluded compounds flagged as insufficiently fragmented
- Per-spectrum entropy values used for the exclusion decision

## How to apply

Load spectra with sd.read_msp and sanitize each one (remove zero-intensity ions via spectral_operations.remove_zero_ions, sort by mass). Compute the spectral entropy of each raw spectrum; flag compounds whose spectra contain fewer than two fragment ions, which corresponds to spectral entropy <0.5. Exclude these flagged compounds before running Spectral Denoising or entropy-similarity benchmarking, because low-entropy reference spectra are disproportionately vulnerable to noise ions and can yield misleading similarity scores. In the article's 240-metabolite dilution-series benchmark this screen removed 28 compounds, leaving spectra suitable for electronic_denoising, formula_denoising, and comparison against the 500-pmol ground-truth spectra.

## Related tools

- **Spectral Denoising** (Python package whose read_msp, sanitize/spectral_operations functions and spectral_entropy metric implement the fragment-ion and entropy screening) — https://github.com/FanzhouKong/spectral_denoising

## Examples

```
import spectral_denoising as sd; from spectral_denoising.spectral_operations import *; query_data = sd.read_msp('sample_data/noisy_spectra.msp'); entropies = [spectral_entropy(p) for p in query_data['peaks']]
```

## Evaluation signals

- Every excluded compound has fewer than two fragment ions and spectral entropy <0.5
- All retained spectra contain at least two fragment ions after zero-intensity ion removal
- Excluded compound count should be comparable to the article's benchmark (28 of 240 metabolites excluded)
- Retained spectra remain compatible with downstream entropy similarity >0.75 scoring against 500-pmol ground-truth spectra

## Limitations

- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives even after screening
- The screen discards poorly fragmenting compounds entirely rather than attempting denoising on them
- Method is optimized for Orbitrap instruments; adapting to other mass analyzers requires retuning and validating parameters

## Evidence

- [results] Exclusion of low-fragmentation compounds before benchmarking: "A total of 28 compounds were excluded due to insufficient fragmentation with fewer than two fragment ions (spectral entropy <0.5)"
- [discussion] Vulnerability of low-entropy reference spectra: "Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives."
- [methods] Zero-intensity ion removal as part of sanitization: "``spectral_operations.remove_zero_ions`` will remove ions with zero intensity from the query spectrum."
- [readme] Entropy computation on raw vs. noisy spectra from README usage example: [evidence span withheld — anchor status: external_doc]
