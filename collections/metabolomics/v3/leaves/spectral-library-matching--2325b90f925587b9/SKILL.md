---
name: spectral-library-matching
description: Use when query MS/MS spectra in MSP format must be annotated against
  a reference spectral library, especially when electronic/chemical noise degrades
  similarity scores or low-abundance exposome compounds must be recovered.
when_to_use_negative:
- The query spectra already have trusted molecular formula assignments and the goal
  is only noise removal — plain spectral_denoising (electronic + formula denoising)
  suffices without library search
- Sole reliance on spectral matching is insufficient to distinguish isomeric compounds
  or in-source fragments — supplement with retention time matching, molecular cross-section
  comparisons, or biological metadata screening instead
- Spectra come from a non-Orbitrap analyzer without retuned parameters — the electronic
  denoising threshold was calibrated on 230,000 NIST23 Orbitrap spectra and is instrument-specific
edam_operation: http://edamontology.org/operation_3649
tools:
- name: spectral_denoising
  role: Provides sd.read_msp (MSP ingestion/cleaning), sd.denoising_search and sd.denoising_search_batch
    (parallel library matching with per-candidate denoising), and sd.spectral_denoising
    for pre-denoising spectra
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Computes the entropy similarity used as the spectral matching score between
    query (raw and denoised) and reference spectra
- name: ClassyFire
  role: Classifies annotated hits into the seven main compound superclasses for biological
    interpretation (drug metabolites, household/industrial chemicals, pesticides)
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:f4426020806caad491a732e643a53407b8e051916050e09b6d729824cd439692
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - ClassyFire
  - Exploris 240 Orbitrap
  - Orbitrap Astral
  - Spectral Denoising
  - ms_entropy
  - spectral_denoising
  edam_operation: http://edamontology.org/operation_3649
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# spectral-library-matching

## Summary

Annotates experimental MS/MS spectra by matching them against a reference spectral library (MSP format) using entropy similarity, with precursor mass filtering and optional per-candidate denoising (Denoising Search) to boost annotation yield on noisy Orbitrap Astral data.

## When to use

Use when query MS/MS spectra in MSP format must be annotated against a reference spectral library, especially when electronic/chemical noise degrades similarity scores or low-abundance exposome compounds must be recovered.

## When NOT to use

- The query spectra already have trusted molecular formula assignments and the goal is only noise removal — plain spectral_denoising (electronic + formula denoising) suffices without library search
- Sole reliance on spectral matching is insufficient to distinguish isomeric compounds or in-source fragments — supplement with retention time matching, molecular cross-section comparisons, or biological metadata screening instead
- Spectra come from a non-Orbitrap analyzer without retuned parameters — the electronic denoising threshold was calibrated on 230,000 NIST23 Orbitrap spectra and is instrument-specific

## Inputs

- Query MS/MS spectra in MSP format (dataframe with 'peaks' and 'precursor_mz' columns)
- Reference spectral library in MSP format with a 'smiles' column (molecular formula fallback supported)
- Precursor mass accuracy threshold (10 mDa per the paper; 5 ppm windows are the typical conventional search)

## Outputs

- Per-spectrum list of precursor candidates with raw-query entropy similarity and denoised_similarity to each reference
- Annotation counts per instrument/condition and compound-class breakdown (ClassyFire superclasses)
- Full per-spectrum candidate/similarity table (e.g., denoising_search_results.csv)

## How to apply

Load query spectra and the reference library with sd.read_msp, which sorts peaks and removes zero-intensity ions. Select the 'peaks' and 'precursor_mz' columns from the query dataframe and run sd.denoising_search_batch (parallel, wrap in main() for multiprocessing) against the reference library, restricting candidate formulas to those within the 10-mDa precursor mass accuracy window. For each query spectrum the search returns all precursor candidates with entropy similarity of the raw query and denoised_similarity of the denoised query to each reference; the precursor ion region is excluded from similarity calculation so only genuine fragments contribute. The library must expose a 'smiles' column, with molecular formula as fallback (pass smiles_col='formula' if only formulas are available). Aggregate top annotations per spectrum and, for benchmarking like the AD plasma comparison, count annotations per instrument (Orbitrap Astral vs Exploris 240) and classify hits by ClassyFire superclasses (e.g., drug metabolites, household/industrial chemicals, pesticides).

## Related tools

- **spectral_denoising** (Provides sd.read_msp (MSP ingestion/cleaning), sd.denoising_search and sd.denoising_search_batch (parallel library matching with per-candidate denoising), and sd.spectral_denoising for pre-denoising spectra) — https://github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Computes the entropy similarity used as the spectral matching score between query (raw and denoised) and reference spectra)
- **ClassyFire** (Classifies annotated hits into the seven main compound superclasses for biological interpretation (drug metabolites, household/industrial chemicals, pesticides))

## Examples

```
import spectral_denoising as sd; results = sd.denoising_search_batch(query_spectra['peaks'], query_spectra['precursor_mz'], reference_library)
```

## Evaluation signals

- On denoised NIST23 benchmark spectra, average entropy similarity should be greater than 0.99
- Denoising Search on Alzheimer's disease plasma Orbitrap Astral data should reproduce ~2.3-fold (2.5-fold per discussion) more annotated compounds than raw Exploris 240 MS/MS spectra
- Every returned candidate must fall within the precursor mass accuracy window (10 mDa in the paper) of the query precursor_mz
- Expected exposome compound classes (drug metabolites, household/industrial chemicals, pesticides) should be recovered among annotations
- Similarity values must be computed with the precursor region excluded; the denoised-spectrum entropy example dropped from S = 3.3 to S = 1.9 after denoising

## Limitations

- Chemical plausibility checks are limited to Seven Golden Rules without considering molecular connectivity, so not all noise ions are removed
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Inability to differentiate isomeric compounds and in-source fragments when relying solely on MS/MS spectral matching
- Hard thresholding based exclusively on spectral similarity is suboptimal for compound annotation
- Parameters are optimized for Orbitrap instruments; non-Orbitrap analyzers require retuning and validation of the electronic-noise threshold and mass tolerance
- Python version must be 3.8–3.12 (RDKit lacks Python 3.13 wheels); batch functions must be called inside main() due to multiprocessing

## Evidence

- [results] Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy: "Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy"
- [results] 10-mDa threshold: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [results] precursor exclusion: "the remaining abundance of the precursor ion intensities is ignored to focus entirely on genuine fragment spectra"
- [results] NIST23 benchmark similarity: "The average similarity of the selected denoised NIST23 spectra was greater than 0.99"
- [readme] README search output contract: [evidence span withheld — anchor status: external_doc]
- [methods] formula denoising step: "The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion."
