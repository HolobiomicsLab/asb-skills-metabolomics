---
name: entropy-similarity-scoring
description: Use when quantitatively ranking MS/MS spectral matches or measuring denoising
  benefit — e.g. computing entropy similarity of raw vs. denoised query peaks against
  reference library spectra for compound annotation.
when_to_use_negative:
- Spectra with fewer than two fragment ions (spectral entropy <0.5) — insufficient
  fragmentation makes similarity scoring unreliable
- Scenarios where isomeric compounds or in-source fragments must be distinguished
  — entropy similarity alone cannot differentiate them and hard thresholding on similarity
  is suboptimal for annotation
- When the precursor ion region should be included — the method explicitly ignores
  remaining precursor ion intensity in all MS/MS similarity calculations
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spectral_denoising (sd)
  role: Provides spectral_operations.entropy_similairty and spectral_entropy used
    to score raw and denoised spectra with precursor m/z supplied
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Entropy similarity implementation underlying spectral matching scores in Denoising
    Search
- name: Denoising Search
  role: Computes entropy similarity of both raw and denoised query spectra against
    all candidate references within the precursor mass accuracy
  repo: https://github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_003
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:2e9163d107891b919fdf2d70e9c2f2031e92305480844206bf8f33e3512066e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - ClassyFire
  - Denoising Search
  - Exploris 240 Orbitrap
  - Orbitrap Astral
  - ms_entropy
  - ms_entropy (entropy similarity)
  - spectral_denoising (sd)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# entropy-similarity-scoring

## Summary

Score MS/MS spectral matches using spectral entropy and entropy similarity, computed with the precursor ion region excluded, to compare raw vs. denoised query spectra against reference library entries in the Spectral Denoising / Denoising Search workflows.

## When to use

Use when quantitatively ranking MS/MS spectral matches or measuring denoising benefit — e.g. computing entropy similarity of raw vs. denoised query peaks against reference library spectra for compound annotation.

## When NOT to use

- Spectra with fewer than two fragment ions (spectral entropy <0.5) — insufficient fragmentation makes similarity scoring unreliable
- Scenarios where isomeric compounds or in-source fragments must be distinguished — entropy similarity alone cannot differentiate them and hard thresholding on similarity is suboptimal for annotation
- When the precursor ion region should be included — the method explicitly ignores remaining precursor ion intensity in all MS/MS similarity calculations

## Inputs

- Query MS/MS peaks arrays (mass–intensity pairs) with precursor m/z, loaded via sd.read_msp from query_spectra.msp
- Reference spectral library spectra (reference_library.msp) with molecular formula or 'smiles' column
- Denoised query peaks produced by sd.spectral_denoising or sd.denoising_search

## Outputs

- Entropy similarity values for raw query vs. reference ('entrpy_similarity' column)
- Entropy similarity values for denoised query vs. reference ('denoised_similarity' column)
- Per-spectrum candidate tables (e.g. denoising_search_results.csv) with the four required columns: 'query_peaks', 'query_peaks_denoised', 'entrpy_similarity', 'denoised_similarity'

## How to apply

Compute the spectral entropy of query spectra to assess fragmentation quality (the paper's denoised example spectrum has S = 1.9 versus S = 3.3 for the contaminated spectrum, and compounds with spectral entropy <0.5 are excluded as insufficiently fragmented). Use the ms_entropy implementation of entropy similarity via spectral_operations.entropy_similairty, always supplying the precursor m/z so the precursor ion region is excluded from the score. In Denoising Search, score each candidate reference within the 10-mDa precursor mass accuracy window twice: once against the raw query peaks ('entrpy_similarity') and once against the pseudo-denoised query peaks ('denoised_similarity'). On NIST23 Orbitrap benchmark spectra, correctly denoised spectra should reach average entropy similarity greater than 0.99 relative to their references.

## Related tools

- **spectral_denoising (sd)** (Provides spectral_operations.entropy_similairty and spectral_entropy used to score raw and denoised spectra with precursor m/z supplied) — https://github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Entropy similarity implementation underlying spectral matching scores in Denoising Search)
- **Denoising Search** (Computes entropy similarity of both raw and denoised query spectra against all candidate references within the precursor mass accuracy) — https://github.com/FanzhouKong/spectral_denoising

## Examples

```
from spectral_denoising.spectral_operations import entropy_similairty; sim = entropy_similairty(peak_denoised, peak, pmz = pmz)
```

## Evaluation signals

- Result table contains the four required columns 'query_peaks', 'query_peaks_denoised', 'entrpy_similarity', and 'denoised_similarity' with per-candidate rows
- Denoised NIST23 Orbitrap benchmark spectra reach average entropy similarity greater than 0.99 against their references
- Contaminated/denoised entropy contrast reproduces the paper's direction: S = 3.3 for the contaminated spectrum vs. S = 1.9 after denoising
- Entropy similarity is computed with the precursor m/z supplied so the precursor ion region is excluded from the score
- Denoised similarity should exceed raw (noisy) similarity for true reference candidates

## Limitations

- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Solely relying on MS/MS spectral matching cannot differentiate isomeric compounds and in-source fragments; hard thresholding based exclusively on spectral similarity is suboptimal
- Inherent limitations persist when solely relying on MS/MS spectra for compound annotation, even after removing noise ions

## Evidence

- [results] Entropy similarity with precursor exclusion: "the remaining abundance of the precursor ion intensities is ignored to focus entirely on genuine fragment spectra"
- [results] Spectral entropy before/after denoising: "The denoised MS/MS example spectrum in 1.4 (Fig. 1c) would calculate S = 1.9, compared to a more disordered contaminated spectrum 1.1 with S = 3.3"
- [results] Benchmark similarity threshold: "The average similarity of the selected denoised NIST23 spectra was greater than 0.99"
- [readme] entropy_similairty usage with precursor m/z: [evidence span withheld — anchor status: external_doc]
- [readme] Entropy computation result columns: [evidence span withheld — anchor status: external_doc]
