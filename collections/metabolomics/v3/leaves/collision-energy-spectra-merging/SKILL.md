---
name: collision-energy-spectra-merging
description: Use when LC-MS/MS data contain multiple fragmentation spectra per feature
  at different collision energies (e.
when_to_use_negative:
- When only a single collision energy spectrum exists per feature and no repeated
  acquisitions are available — merging cannot create fragment coverage that is not
  there
- When LC-MS/MS runs contain spectra in profiled mode or negative ion mode only —
  these are discarded per the article's preprocessing rules, leaving nothing to merge
- When spectra already come from a reference spectral library and the goal is library
  matching rather than de novo annotation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS 4
  role: Collects fragmentation spectra and precursor information, searches for features,
    and merges spectra assigned to the same feature via agglomerative clustering based
    on cosine distance before de novo formula determination
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: 'Downstream consumer of merged spectra: the merged spectrum per feature is
    what drives higher annotation rates and better confidence-score separation'
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:259b3f6a519b07094897fbf34f0a3dff31dc670814260cd90c0d59f8928c3b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# collision-energy-spectra-merging

## Summary

Merges fragmentation spectra recorded at different collision energies (10, 20, 40 eV) for the same precursor feature into a single consensus spectrum, plus combines multiple MS/MS runs, using agglomerative clustering based on cosine distance; this improves de novo formula and structure annotation rates over any single collision energy.

## When to use

Use when LC-MS/MS data contain multiple fragmentation spectra per feature at different collision energies (e.g. Orbitrap 10/20/40 eV stepwise) or repeated acquisitions across runs, and the goal is higher-confidence SIRIUS 4 / CSI:FingerID formula and structure annotation than any single-energy spectrum supports.

## When NOT to use

- When only a single collision energy spectrum exists per feature and no repeated acquisitions are available — merging cannot create fragment coverage that is not there
- When LC-MS/MS runs contain spectra in profiled mode or negative ion mode only — these are discarded per the article's preprocessing rules, leaving nothing to merge
- When spectra already come from a reference spectral library and the goal is library matching rather than de novo annotation

## Inputs

- LC-MS/MS runs in .mzML or .mzXML format (centroided mode)
- Multiple fragmentation spectra per feature acquired at 10, 20 and 40 eV collision energies
- Precursor ion information (m/z, adduct, isotope peaks) associated with each fragmentation spectrum

## Outputs

- One merged fragmentation spectrum per feature (precursor ion)
- Per-feature spectrum used for downstream SIRIUS 4 formula determination and CSI:FingerID structure ranking

## How to apply

Assign every fragmentation spectrum in the mzML or mzXML LC-MS/MS run to its precursor feature (precursor ion, adduct, isotope peaks), then merge spectra assigned to the same feature using an agglomerative clustering algorithm based on cosine distance. Within each cluster, combine peaks across collision energies so that merged spectra typically cover more fragments than any single 10, 20 or 40 eV spectrum. SIRIUS 4 handles the merging internally during processing: it first collects all fragmentation spectra and precursor information, searches for features, then clusters and merges per feature. Rationale: merged spectra improve both annotation rates and the separation of the COSMIC confidence score, usually outperforming the best single collision energy (typically 40 eV next-best). Before merging, discard spectra with at most one peak, and discard a compound entirely if any of its collision-energy or noise conditions yields such a low-information spectrum.

## Related tools

- **SIRIUS 4** (Collects fragmentation spectra and precursor information, searches for features, and merges spectra assigned to the same feature via agglomerative clustering based on cosine distance before de novo formula determination) — https://github.com/sirius-ms/sirius
- **COSMIC** (Downstream consumer of merged spectra: the merged spectrum per feature is what drives higher annotation rates and better confidence-score separation) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input sample.mzML --merge-spectra --output merged.sirius
```

## Evaluation signals

- Annotation rates (fraction of compounds with a correct top-ranked structure) for merged spectra should exceed those for any single collision energy, usually followed by 40 eV spectra as next best
- The separation of the COSMIC confidence score (correct vs incorrect hits) should be consistently better for merged spectra than for 10/20/40 eV alone
- Correct-hit totals at ROC end should reproduce reported values, e.g. 1,901 (40 eV) vs 1,948 (merged spectra) for the independent QTOF set
- Each merged spectrum must correspond to exactly one feature (precursor ion) and no feature should have multiple merged spectra
- Spectra reduced to at most one peak must be excluded before merging, and any compound producing such a spectrum at any collision energy or noise level must be dropped from all libraries

## Limitations

- Merging cannot help when a feature has only one fragmentation spectrum, or when the available spectra are extremely sparse (≤1 peak) and must be discarded
- The benefit is limited to fragmentation spectra from the same precursor feature; spectra from different features or different adducts are not merged
- Merging does not recover stereochemistry — only the two-dimensional structure is considered, as establishing stereochemistry from fragmentation spectra is beyond automated search engines

## Evidence

- [discussion] agglomerative clustering by cosine distance: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] merged spectra usually best: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] only .mzML/.mzXML, profiled runs discarded: [evidence span withheld — anchor status: not_found]
- [discussion] discard compounds with ≤1-peak spectra: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
- [readme] SIRIUS collects and merges per feature: "SIRIUS first collects all fragmentation spectra and their precursor information and then searches for features that are associated with those fragmentation spectra"
- [discussion] 10 ppm mass accuracy for de novo formula: "We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm."
