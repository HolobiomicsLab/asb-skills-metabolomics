---
name: compound-class-annotation
description: Use when Denoising Search batch results on plasma MS/MS spectra (Orbitrap
  Astral vs Exploris 240) need to be aggregated into per-instrument annotation counts
  and classified by compound class via ClassyFire superclasses.
when_to_use_negative:
- The spectra have not yet been searched — annotation counts and class assignments
  require prior Denoising Search output with denoised_similarity per candidate
- Compounds with insufficient fragmentation (fewer than two fragment ions, spectral
  entropy <0.5) are the analysis target; these were excluded in the paper's workflow
- Only isomer-level differentiation is needed — the paper notes MS/MS spectral matching
  alone cannot differentiate isomeric compounds and in-source fragments
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ClassyFire
  role: Assigns the seven main compound superclasses used to classify Denoising Search
    annotations into compound classes
- name: sd.denoising_search_batch
  role: Produces the per-spectrum candidate lists with raw and denoised entropy similarities
    that are aggregated into annotation counts and class assignments
  repo: github.com/FanzhouKong/spectral_denoising
- name: sd.read_msp
  role: Loads query spectra and reference library from MSP files with basic cleaning
    (sorting peaks, removing zero-intensity ions)
  repo: github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Computes entropy similarity of raw and denoised query spectra to reference
    candidates, used for ranking top annotations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:4668a628a08efa7220d33be077330664f8baf0222a92733c5cba9d39681156fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - Exploris 240 Orbitrap
  - Orbitrap Astral
  - ms_entropy
  - sd.denoising_search_batch
  - sd.read_msp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# compound-class-annotation

## Summary

Classifies MS/MS compound annotations recovered by Denoising Search into compound classes (drug metabolites, household/industrial chemicals, pesticides) using ClassyFire superclasses, and aggregates annotation counts per instrument (Orbitrap Astral vs Exploris 240) to quantify annotation gains.

## When to use

Use when Denoising Search batch results on plasma MS/MS spectra (Orbitrap Astral vs Exploris 240) need to be aggregated into per-instrument annotation counts and classified by compound class via ClassyFire superclasses.

## When NOT to use

- The spectra have not yet been searched — annotation counts and class assignments require prior Denoising Search output with denoised_similarity per candidate
- Compounds with insufficient fragmentation (fewer than two fragment ions, spectral entropy <0.5) are the analysis target; these were excluded in the paper's workflow
- Only isomer-level differentiation is needed — the paper notes MS/MS spectral matching alone cannot differentiate isomeric compounds and in-source fragments

## Inputs

- Denoising Search batch results: list of per-spectrum precursor candidates with entropy similarities of raw and denoised spectra
- Query spectra MSP file with 'peaks' and 'precursor_mz' columns (Orbitrap Astral and Exploris 240 AD plasma MS/MS)
- Reference spectral library MSP file exposing a 'smiles' column (molecular formula fallback supported)
- ClassyFire superclass assignments for annotated compounds

## Outputs

- Annotation counts per instrument (Orbitrap Astral vs Exploris 240)
- Compound-class breakdown of annotated hits (drug metabolites, household/industrial chemicals, pesticides) per ClassyFire superclass
- denoising_search_results.csv containing the full per-spectrum candidate and denoised-similarity table

## How to apply

After running sd.denoising_search_batch over all query spectra against the MSP-format reference library (using the 10-mDa precursor mass accuracy threshold and entropy similarity for raw and denoised spectra), aggregate the top annotations per spectrum into annotation counts per instrument. Classify the annotated hits by compound class using ClassyFire superclasses — specifically the seven main superclasses — to recover expected classes such as drug metabolites, household and industrial chemicals, and pesticides. Write the full per-spectrum candidate table (reference candidates, query_peaks, query_peaks_denoised, raw entropy similarity, and denoised_similarity) to denoising_search_results.csv. Validation compares annotation counts between the Orbitrap Astral and Exploris 240 (expected ~2.3- to 2.5-fold more annotations on the Astral) and checks that the expected exposome compound classes are recovered.

## Related tools

- **ClassyFire** (Assigns the seven main compound superclasses used to classify Denoising Search annotations into compound classes)
- **sd.denoising_search_batch** (Produces the per-spectrum candidate lists with raw and denoised entropy similarities that are aggregated into annotation counts and class assignments) — github.com/FanzhouKong/spectral_denoising
- **sd.read_msp** (Loads query spectra and reference library from MSP files with basic cleaning (sorting peaks, removing zero-intensity ions)) — github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Computes entropy similarity of raw and denoised query spectra to reference candidates, used for ranking top annotations)

## Examples

```
results = sd.denoising_search_batch(query_spectra['peaks'], query_spectra['precursor_mz'], reference_library)
```

## Evaluation signals

- Orbitrap Astral annotation count reproduces approximately 2.3-fold (2.5-fold per the discussion) more annotated compounds than the Exploris 240
- Expected compound classes (drug metabolites, household/industrial chemicals, pesticides) are recovered among the annotations
- Annotations are classified across the seven main ClassyFire compound superclasses
- All reported candidates fall within the 10-mDa precursor mass accuracy threshold used to restrict candidate formulas
- denoising_search_results.csv contains, for each query spectrum, all candidate references with query_peaks, query_peaks_denoised, raw entropy similarity, and denoised_similarity

## Limitations

- Hard thresholding based exclusively on spectral similarity is suboptimal for compound annotation; results should be supplemented with retention time matching, molecular cross-section comparisons, and biological metadata screening
- Isomeric compounds and in-source fragments cannot be differentiated when relying solely on MS/MS spectral matching, affecting class-level assignments
- Reference spectra with inherently low spectral entropy are vulnerable to noise ions, potentially leading to false negatives in the annotation pool being classified

## Evidence

- [abstract] Denoising Search detected 2.5-fold more annotated compounds compared to the Exploris 240 Orbitrap instrument, including drug metabolites, household and industrial chemicals, and pesticides.: "Denoising Search detected 2.5-fold more annotated compounds compared to the Exploris 240 Orbitrap instrument, including drug metabolites, household and industrial chemicals, and pesticides."
- [results] A closer examination of the seven main ClassyFire compound superclasses: "A closer examination of the seven main ClassyFire compound superclasses"
- [results] For practical reasons, a 10-mDa mass accuracy threshold was used: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [readme] results will be a list of all correspoinding precursor mz candidates, each one with entropy similarities of both raw and denoised spectra: [evidence span withheld — anchor status: external_doc]
