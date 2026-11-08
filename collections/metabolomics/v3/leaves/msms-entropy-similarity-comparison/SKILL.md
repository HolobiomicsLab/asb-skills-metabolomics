---
name: msms-entropy-similarity-comparison
description: Use when you need to quantify agreement between a noisy/denoised query
  MS/MS spectrum and a clean reference spectrum (e.g., validating electronic+chemical
  denoising of .msp spectra) or to score spectral matches in Denoising Search.
when_to_use_negative:
- The reference spectrum has inherently low spectral entropy — these are particularly
  vulnerable to noise ions and can yield false negatives
- The goal is differentiating isomeric compounds or in-source fragments — entropy
  similarity alone cannot distinguish them
- The spectra were acquired on non-Orbitrap mass analyzers — the denoising parameters
  this comparison was built on are optimized for Orbitrap instruments and would require
  retuning
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Spectral Denoising
  role: Python package providing spectral_entropy, normalized_entropy, and entropy_similairty
    functions used to compare raw, noisy, and denoised MS/MS spectra
  repo: github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:484aba4331156686c35773091e7683b314606a98fe4f5182f46b66af88afe7d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spectral Denoising
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# msms-entropy-similarity-comparison

## Summary

Compares raw, noisy, and denoised MS/MS spectra using spectral entropy and entropy similarity, as implemented in the spectral_denoising package, to validate denoising quality and score compound annotation candidates.

## When to use

Use when you need to quantify agreement between a noisy/denoised query MS/MS spectrum and a clean reference spectrum (e.g., validating electronic+chemical denoising of .msp spectra) or to score spectral matches in Denoising Search.

## When NOT to use

- The reference spectrum has inherently low spectral entropy — these are particularly vulnerable to noise ions and can yield false negatives
- The goal is differentiating isomeric compounds or in-source fragments — entropy similarity alone cannot distinguish them
- The spectra were acquired on non-Orbitrap mass analyzers — the denoising parameters this comparison was built on are optimized for Orbitrap instruments and would require retuning

## Inputs

- MS/MS peak array (m/z and intensity columns) from sd.read_msp on a .msp file, e.g. sample_data/noisy_spectra.msp or sample_data/query_spectra.msp
- Clean reference spectrum peak array (m/z and intensity) at the same precursor m/z
- Precursor m/z value for the spectrum pair being compared

## Outputs

- Spectral entropy S (e.g. 1.9 denoised vs 3.3 contaminated) and normalized entropy of the spectrum
- Entropy similarity score between query and reference spectra (0–1; 1.00 = recovered clean spectrum, 0.37 = noisy spectrum)
- Per-candidate entropy similarities (raw and denoised) from denoising_search / denoising_search_batch results

## How to apply

Load spectra from .msp files with sd.read_msp, which sorts peaks and drops zero-intensity ions. Compute spectral_entropy (S) and normalized_entropy on the peak array to gauge spectral disorder; in the paper's example, denoising reduced S from 3.3 to 1.9. Call sd.entropy_similairty(query_peaks, reference_peaks, pmz=precursor_mz) to obtain a similarity score in which the remaining precursor ion intensity is ignored so scoring focuses entirely on genuine fragment ions. A similarity of 1.00 indicates the denoised spectrum matches the clean reference (versus 0.37 for the noisy spectrum), and the average similarity of selected denoised NIST23 spectra exceeded 0.99. In Denoising Search, entropy similarities are computed for both raw and denoised spectra against each precursor candidate; additionally exclude compounds with insufficient fragmentation (spectral entropy <0.5) before downstream similarity analysis.

## Related tools

- **Spectral Denoising** (Python package providing spectral_entropy, normalized_entropy, and entropy_similairty functions used to compare raw, noisy, and denoised MS/MS spectra) — github.com/FanzhouKong/spectral_denoising

## Examples

```
from spectral_denoising.spectral_operations import entropy_similairty; sim = entropy_similairty(peak_denoised, peak, pmz = pmz)
```

## Evaluation signals

- Entropy similarity of the denoised spectrum vs the clean reference recovers 1.00, while the noisy spectrum scores 0.37
- Spectral entropy S drops after denoising (documented example: S = 1.9 denoised vs S = 3.3 contaminated spectrum)
- Average entropy similarity of selected denoised NIST23 spectra is greater than 0.99
- Compounds with fewer than two fragment ions (spectral entropy <0.5) are excluded from similarity analysis
- Precursor ion intensity region contributes nothing to the similarity score (remove_precursor / precursor intensity ignored)

## Limitations

- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Inability to differentiate isomeric compounds and in-source fragments when relying solely on MS/MS spectral similarity
- Method is optimized for Orbitrap instruments (e.g. Orbitrap Astral, Exploris 240); adapting to other mass analyzers requires retuning and validating parameters

## Evidence

- [results] The denoised MS/MS example spectrum in 1.4 (Fig. 1c) would calculate S = 1.9, compared to a more disordered contaminated spectrum 1.1 with S = 3.3: "The denoised MS/MS example spectrum in 1.4 (Fig. 1c) would calculate S = 1.9, compared to a more disordered contaminated spectrum 1.1 with S = 3.3"
- [results] the remaining abundance of the precursor ion intensities is ignored to focus entirely on genuine fragment spectra: "the remaining abundance of the precursor ion intensities is ignored to focus entirely on genuine fragment spectra"
- [methods] ``spectral_operations.remove_zero_ions`` will remove ions with zero intensity from the query spectrum.: "``spectral_operations.remove_zero_ions`` will remove ions with zero intensity from the query spectrum."
- [readme] print(f'the entropy similarity of denoised spectrum and the raw spectrum is {entropy_similairty(peak_denoised, peak, pmz = pmz):.2f}'): [evidence span withheld — anchor status: external_doc]
