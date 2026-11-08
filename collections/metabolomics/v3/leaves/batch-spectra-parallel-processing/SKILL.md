---
name: batch-spectra-parallel-processing
description: Use when a large set of MS/MS query spectra (e.g., an MSP file from AD
  plasma on Orbitrap Astral or Exploris 240) must each be denoised or matched against
  a reference library, and per-spectrum loops would be too slow.
when_to_use_negative:
- Only one or a few spectra need denoising or searching — use the single-spectrum
  sd.spectral_denoising or sd.denoising_search calls instead
- The task requires adapting the method to non-Orbitrap mass analyzers, where parameters
  must first be retuned and validated
- Input is already a feature/annotation table rather than raw or query MS/MS spectra
  in MSP form
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spectral_denoising (sd)
  role: Python package providing sd.denoising_search_batch and sd.spectral_denoising_batch,
    implemented in parallel with multiprocessing, plus sd.read_msp for loading MSP
    files
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: Denoising Search
  role: 'The analysis executed in batch mode: denoises each experimental spectrum
    using all molecular formulas within the 10-mDa precursor mass accuracy and computes
    entropy similarity over denoised spectra'
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Provides entropy similarity scores for raw vs denoised query spectra against
    reference spectra in the batch results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:a7bc2bd1a0265a69dcf3f0c55d271d677743f0cd6d3fe94dda3165a53aea609a
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
  - sd.denoising_search_batch
  - spectral_denoising (sd)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# batch-spectra-parallel-processing

## Summary

Run Spectral Denoising and Denoising Search over all query spectra from an MSP file in parallel using the spectral_denoising package's *_batch functions, which are implemented with Python multiprocessing. This enables instrument-scale annotation counting (e.g., Orbitrap Astral vs Exploris 240 AD plasma) instead of spectrum-by-spectrum calls.

## When to use

Use when a large set of MS/MS query spectra (e.g., an MSP file from AD plasma on Orbitrap Astral or Exploris 240) must each be denoised or matched against a reference library, and per-spectrum loops would be too slow.

## When NOT to use

- Only one or a few spectra need denoising or searching — use the single-spectrum sd.spectral_denoising or sd.denoising_search calls instead
- The task requires adapting the method to non-Orbitrap mass analyzers, where parameters must first be retuned and validated
- Input is already a feature/annotation table rather than raw or query MS/MS spectra in MSP form

## Inputs

- Query MSP file of experimental MS/MS spectra with 'peaks' and 'precursor_mz' columns (e.g., Orbitrap Astral or Exploris 240 AD plasma spectra)
- Reference spectral library MSP file (e.g., NIST23/MassBank.us/GNPS-derived) exposing a 'smiles' column (or a molecular formula column as fallback)
- Optional formula database (Formula_db) for formula-based candidate restriction

## Outputs

- List of per-spectrum denoising search results: precursor candidates within the mass window, each with query_peaks, query_peaks_denoised, raw entropy similarity, and denoised_similarity
- List of denoised spectra when running sd.spectral_denoising_batch
- Aggregated annotation counts per instrument and compound class (e.g., CSV such as denoising_search_results.csv)

## How to apply

Load the query spectra and reference spectral library with sd.read_msp, which sorts peaks and removes zero-intensity ions. Select the 'peaks' and 'precursor_mz' columns from the query dataframe and pass them to sd.denoising_search_batch (or sd.spectral_denoising_batch for denoising alone), running all spectra in parallel against the reference library; use the 'smiles' column by default or set smiles_col to a formula column as fallback. Because the batch functions use multiprocessing, wrap the call in a main() function guarded by if __name__ == '__main__' when running from a terminal script. Restrict candidate formulas via the precursor mass accuracy threshold (10 mDa in the paper) and collect, per spectrum, the candidate references with entropy similarity of the raw query and denoised_similarity of the denoised query. Aggregate per-spectrum results into annotation counts per instrument and write the full table (e.g., denoising_search_results.csv). Use Python >=3.8 and <3.13 since RDKit lacks a Python 3.13 distribution.

## Related tools

- **spectral_denoising (sd)** (Python package providing sd.denoising_search_batch and sd.spectral_denoising_batch, implemented in parallel with multiprocessing, plus sd.read_msp for loading MSP files) — https://github.com/FanzhouKong/spectral_denoising
- **Denoising Search** (The analysis executed in batch mode: denoises each experimental spectrum using all molecular formulas within the 10-mDa precursor mass accuracy and computes entropy similarity over denoised spectra) — https://github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Provides entropy similarity scores for raw vs denoised query spectra against reference spectra in the batch results)

## Examples

```
import spectral_denoising as sd; results = sd.denoising_search_batch(query_spectra['peaks'], query_spectra['precursor_mz'], reference_library)
```

## Evaluation signals

- Batch output is a list whose length equals the number of query spectra, each entry containing precursor candidates with both raw entropy similarity and denoised_similarity fields
- All candidates returned per spectrum fall within the 10-mDa precursor mass accuracy window of the query precursor_mz
- Reproducing the AD plasma workflow yields the Orbitrap Astral annotating approximately 2.3–2.5-fold more compounds than the Exploris 240
- Each candidate record includes 'query_peaks' and 'query_peaks_denoised' so raw and denoised spectra can be compared
- Script compiles without hanging: batch calls must be wrapped in main() under if __name__ == '__main__' due to multiprocessing

## Limitations

- Batch functions use multiprocessing; calling them at top level in a terminal script without a main() guard causes issues
- Requires Python between 3.8 and 3.12; RDKit has no compatible distribution for Python 3.13
- Hard thresholding on spectral similarity alone is suboptimal; supplement with retention time matching, molecular cross-section comparisons, and biological metadata screening
- Reference spectra with inherently low spectral entropy are vulnerable to noise ions, potentially causing false negatives; isomeric compounds and in-source fragments cannot be differentiated by MS/MS matching alone

## Evidence

- [readme] results will be a list of all correspoinding precursor mz candidates, each one with entropy similarities of both raw and denoised spectra: [evidence span withheld — anchor status: external_doc]
- [readme] wrap batch calls in main(): [evidence span withheld — anchor status: external_doc]
- [results] Denoising Search denoises using all formulas within precursor mass accuracy: "Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy"
- [results] 10-mDa threshold: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [readme] Python version constraint: "Please use Python version between 3.8 to 3.12 for this package to work. RDkit currently does not have a distribution compitable to python 3.13!!!!!"
