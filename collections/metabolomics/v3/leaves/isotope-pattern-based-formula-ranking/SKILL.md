---
name: isotope-pattern-based-formula-ranking
description: Use when you have high-resolution LC-MS/MS peak lists for small molecules
  and need to rank molecular formula candidates for the precursor ion, especially
  when MS1 isotope patterns are available or the element alphabet should be restricted.
when_to_use_negative:
- Input spectra are low-resolution and contain no usable isotope patterns, so isotope-pattern
  ranking cannot discriminate candidate decompositions
- The goal is already-determined formulas to be searched as structures in molecular
  databases, not formula ranking from spectra
- You need compound structure identification (CSI:FingerID/CANOPUS/MSNovelist) without
  first establishing the molecular formula
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Performs the isotope pattern analysis and molecular formula decomposition/ranking;
    runnable in command-line mode for batch benchmark datasets
  repo: github.com/sirius-ms/sirius
- name: SIRIUS
  role: Java-based framework hosting the isotope pattern analysis workflow (releases
    downloadable from the repository)
  repo: https://github.com/sirius-ms/sirius/releases
- name: sirius-client-openAPI
  role: OpenAPI SDKs (Python/R/Java) for driving SIRIUS analyses programmatically
  repo: https://github.com/sirius-ms/sirius-client-openAPI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:6834460b51f1ee5ff94b2a64cff42f6e25cfdd663dbe2fd7c07fe67256086511
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SIRIUS
  - SIRIUS 4
  - sirius-client-openAPI
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# isotope-pattern-based-formula-ranking

## Summary

SIRIUS's isotope pattern analysis deduces molecular formulas of small compounds by ranking candidate elemental decompositions of the precursor ion against high-resolution isotope patterns from MS1, restricted to a chosen element alphabet (e.g., CHNOPS, or CHNOPS plus BrClFI for halogen-containing compounds).

## When to use

Use when you have high-resolution LC-MS/MS peak lists for small molecules and need to rank molecular formula candidates for the precursor ion, especially when MS1 isotope patterns are available or the element alphabet should be restricted.

## When NOT to use

- Input spectra are low-resolution and contain no usable isotope patterns, so isotope-pattern ranking cannot discriminate candidate decompositions
- The goal is already-determined formulas to be searched as structures in molecular databases, not formula ranking from spectra
- You need compound structure identification (CSI:FingerID/CANOPUS/MSNovelist) without first establishing the molecular formula

## Inputs

- High-resolution MS1 isotope pattern peak list (masses and abundances) for the precursor ion
- High-resolution MS/MS tandem mass spectrum peak list for the compound
- Precursor mass/neutral mass and adduct/charge information
- Element alphabet restriction (e.g., CHNOPS, or CHNOPS + BrClFI when halogens may be present)

## Outputs

- Ranked list of candidate molecular formulas for the precursor ion with isotope-pattern scores
- Per-compound decomposition results suitable for downstream fragmentation tree computation
- Batch run logs/wall-clock timings when processing benchmark compound sets

## How to apply

Preprocess the high-resolution mass spectra so the instrument output becomes a list of peaks with masses and abundances; SIRIUS then ranks isotope patterns from these MS1 peak lists to decompose the parent ion mass. Restrict the element alphabet to the expected elements — the paper's running-time benchmark uses a CHNOPS alphabet for the MS/MS-only dataset (1,553 compounds) and an extended BrClFI alphabet for the MS1+MS/MS dataset (3,965 compounds) — because the element alphabet drives both correctness and runtime. If MS1 data is available, use the parent peak from MS1 for the decomposition; run in SIRIUS's command-line mode for batch processing of many compounds, recording wall-clock time when comparing versions (e.g., the 332-fold CHNOPS and 231-fold BrClFI speedups of SIRIUS 4 over SIRIUS 3.0). Output is a ranked list of molecular formula candidates, which feeds downstream fragmentation tree computation and CSI:FingerID structure search.

## Related tools

- **SIRIUS 4** (Performs the isotope pattern analysis and molecular formula decomposition/ranking; runnable in command-line mode for batch benchmark datasets) — github.com/sirius-ms/sirius
- **SIRIUS** (Java-based framework hosting the isotope pattern analysis workflow (releases downloadable from the repository)) — https://github.com/sirius-ms/sirius/releases
- **sirius-client-openAPI** (OpenAPI SDKs (Python/R/Java) for driving SIRIUS analyses programmatically) — https://github.com/sirius-ms/sirius-client-openAPI

## Evaluation signals

- Top-ranked formula candidates exactly match the precursor mass under the chosen element alphabet (CHNOPS vs CHNOPS+BrClFI)
- On the 1,553-compound MS/MS-only CHNOPS benchmark, reproduced fold speedup matches the reported 332-fold increase over SIRIUS 3.0
- On the 3,965-compound BrClFI dataset, reproduced processing of the 3,850 lightest compounds matches the reported ~25 min 4 s (SIRIUS 4) vs 4 d 1 h (SIRIUS 3.0), i.e., 231-fold speedup
- When MS1 is supplied, decompositions are anchored on the MS1 parent peak rather than MS/MS-derived precursor guesses
- No crash or missing-formula output attributable to known fixed bugs (e.g., GLPK solver crash, isotope pattern selection bugs)

## Limitations

- The isotope pattern analysis was rewritten and its stability was only claimed as 'hopefully more stable than before', so results should be spot-checked across releases
- Known fixed bugs affected isotope pattern selection and penalizing molecular formulas on intrinsically charged mode; older releases may give wrong rankings
- Without MS1 data, decomposition relies on MS/MS-derived precursor information; when MS1 is available SIRIUS always uses the MS1 parent peak for decomposition
- Element alphabet choice strongly affects runtime — the halogen-inclusive BrClFI search is orders of magnitude more expensive (days in SIRIUS 3.0) than CHNOPS-only

## Evidence

- [readme] SIRIUS deduces molecular formulas by isotope pattern ranking: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] Preprocessed spectra are peak lists: "After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance."
- [discussion] MS1 parent peak drives decomposition: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the"
