---
name: spectral-noise-denoising
description: Use when MS/MS spectra contain electronic or chemical noise ions that
  degrade spectral entropy and similarity-based matching, especially for Orbitrap
  Astral or Exploris 240 metabolomics/exposome queries.
when_to_use_negative:
- When no precursor molecular formula or SMILES is available at all — chemical denoising
  depends on subformula-loss checks against the precursor; in that case use Denoising
  Search, which fetches candidate formulas from the reference library within a 10-mDa
  mass window instead
- When working outside Orbitrap instruments — the method is optimized for Orbitrap
  data and adapting to other mass analyzers requires retuning and validating parameters
- When the query spectrum is already a denoised library spectrum or a processed feature
  table rather than raw peak lists
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: 'Core algorithm: electronic_denoising removes ''grass'' noise ions independent
    of relative intensity; formula_denoising removes chemical noise via subformula-loss
    checks'
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: spectral_denoising (Python package)
  role: pip-installable implementation providing spectral_denoising, spectral_denoising_batch,
    read_msp, entropy_similarity, and plotting utilities
  repo: https://github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:9730f3d5f4fe8d47de4ece3c656b5849aef333361c43820a7ec81ca19605456f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - Denoising Search
  - Spectral Denoising
  - spectral_denoising (Python package)
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# spectral-noise-denoising

## Summary

Removes both electronic ('grass') noise and chemically implausible fragment ions from MS/MS spectra using the spectral_denoising Python package, improving high-confidence compound annotation for Orbitrap data.

## When to use

Use when MS/MS spectra contain electronic or chemical noise ions that degrade spectral entropy and similarity-based matching, especially for Orbitrap Astral or Exploris 240 metabolomics/exposome queries.

## When NOT to use

- When no precursor molecular formula or SMILES is available at all — chemical denoising depends on subformula-loss checks against the precursor; in that case use Denoising Search, which fetches candidate formulas from the reference library within a 10-mDa mass window instead
- When working outside Orbitrap instruments — the method is optimized for Orbitrap data and adapting to other mass analyzers requires retuning and validating parameters
- When the query spectrum is already a denoised library spectrum or a processed feature table rather than raw peak lists

## Inputs

- MS/MS query spectra loaded from .msp files via spectral_denoising.read_msp, with 'peaks' and 'precursor_mz' columns
- Molecular identity of the precursor: SMILES string or molecular formula (e.g. 'C5H4N4O'), plus adduct (e.g. '[M+Na]+')
- Reference spectra or noisy spectra from sample_data directories for comparison

## Outputs

- Denoised peaks array per query spectrum (electronic + chemical noise removed)
- Spectral entropy and normalized entropy values (raw, contaminated, and denoised spectra)
- Entropy similarity scores between denoised and raw/reference spectra (values approaching 0.99+ for clean library spectra)

## How to apply

First run electronic_denoising, which removes ions with identical intensities greater than 4 (a threshold empirically derived from NIST23, where >99.5% of spectra have fewer than four ions within ±0.1% relative intensities), plus remove_zero_ions. Then apply formula_denoising: check whether each fragment's exact mass can be a chemically plausible subformula loss from the precursor ion, validating losses with the Seven Golden Rules algorithm while ignoring LEWIS and SENIOR checks, and accounting for [M+N2+H2O] reactions in substituted aromatic compounds. The denoising functions accept either SMILES or molecular formula input, with adduct-specific precursor m/z computed via calculate_precursormz. Compare raw vs. denoised spectra with spectral_entropy and entropy_similarity, excluding the precursor ion region. For batch work, call spectral_denoising_batch inside a main() guard because it uses multiprocessing, and use Python 3.8–3.12 since RDKit has no Python 3.13 distribution.

## Related tools

- **Spectral Denoising** (Core algorithm: electronic_denoising removes 'grass' noise ions independent of relative intensity; formula_denoising removes chemical noise via subformula-loss checks) — https://github.com/FanzhouKong/spectral_denoising
- **spectral_denoising (Python package)** (pip-installable implementation providing spectral_denoising, spectral_denoising_batch, read_msp, entropy_similarity, and plotting utilities) — https://github.com/FanzhouKong/spectral_denoising

## Examples

```
import spectral_denoising as sd; peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct); print(entropy_similairty(peak_denoised, peak, pmz = pmz))
```

## Evaluation signals

- Denoised spectrum's spectral entropy should drop substantially relative to the noisy input (paper example: S = 3.3 contaminated → S = 1.9 denoised)
- Entropy similarity between denoised and known-clean reference spectrum should approach 1.0; paper reports average similarity > 0.99 for benchmark NIST23 spectra
- Denoising is robust: correct fragments should survive even when noise ions exceed 150-fold the intensity of true fragment ions
- Removed electronic-noise ions should match the NIST23-derived criterion (identical-intensity ions occurring >4 times per spectrum); no genuine fragment ions should be dropped unless their masses fail the Seven Golden Rules subformula-loss check
- Batch runs must complete only when wrapped in a main() function (multiprocessing requirement) and output should contain one denoised spectrum per input

## Limitations

- Not all added noise ions are removed because chemical plausibility checks are limited to the Seven Golden Rules method without considering molecular connectivity
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Denoising cannot differentiate isomeric compounds or in-source fragments; supplement with retention time matching, molecular cross-section comparisons, and biological metadata screening
- Python version must be 3.8–3.12 (RDKit lacks a Python 3.13-compatible distribution); batch functions require a main() guard due to multiprocessing

## Evidence

- [results] Electronic denoising threshold from NIST23: "These data served as a valid threshold to automatically identify electronic noise ions in experimental MS/MS spectra and remove these ions, independent of the relative intensity"
- [results] Chemical denoising via subformula loss: "chemical denoising identifies and removes chemical noise ions by calculating whether the exact mass of each fragment can logically be associated with a subformula loss from the parent molecular ion"
- [results] Seven Golden Rules validation with LEWIS/SENIOR ignored: "The chemical plausibility of relative loss subformulas was validated using the Seven Golden Rules algorithm to discard chemically impossible losses"
- [readme] README usage of spectral_denoising with SMILES or formula: "peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)"
- [readme] README batch mode multiprocessing caveat: [evidence span withheld — anchor status: external_doc]
- [abstract] Robustness to >150-fold noise intensity: "even with >150-fold higher intensity of noise ions than true fragment ions"
