---
name: library-entry-validation-against-vignette
description: Reference for building and quality-checking MetaboAnnotatoR fragment
  library entries from experimental MS/MS spectra (including MassBank .msp records)
  when a trusted reference entry from the package vignettes is available for comparison.
when_to_use_negative:
- When no trusted reference library entry exists for the metabolite — the validation
  step relies on diffing against the vignette example and cannot certify correctness
  without one.
- When the goal is annotating unknown features from an AIF LC-MS dataset rather than
  building/curating the fragment library itself — use the feature-annotation workflow
  instead.
- When the source spectrum is DIA data other than AIF (e.g. SWATH), which MetaboAnnotatoR
  does not currently support.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnnotatoR
  role: R package providing genFragEntry (occurrence scoring of marker peaks from
    experimental MS/MS) and mspToLib (.msp import); its vignettes supply the reference
    library entries used for validation
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment in which MetaboAnnotatoR is installed via devtools and
    the library-entry generation/validation scripts are executed
  repo: https://github.com/gggraca/MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:b5d6d55ce6609a83069f8bca4985f4bb8bbaf6849cf7d19939e98dc398acfb41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# library-entry-validation-against-vignette

## Summary

Generate a MetaboAnnotatoR metabolite fragment library entry from an experimental MS/MS spectrum with genFragEntry (or via mspToLib for .msp records) and validate it by diffing fragment m/z values and occurrence scores against the package's vignette reference entry, e.g. Pantothenic acid [M+H]+ (MassBank MSBNK-RIKEN-PR100295).

## When to use

Reference for building and quality-checking MetaboAnnotatoR fragment library entries from experimental MS/MS spectra (including MassBank .msp records) when a trusted reference entry from the package vignettes is available for comparison.

## When NOT to use

- When no trusted reference library entry exists for the metabolite — the validation step relies on diffing against the vignette example and cannot certify correctness without one.
- When the goal is annotating unknown features from an AIF LC-MS dataset rather than building/curating the fragment library itself — use the feature-annotation workflow instead.
- When the source spectrum is DIA data other than AIF (e.g. SWATH), which MetaboAnnotatoR does not currently support.

## Inputs

- Experimental MS/MS spectrum of a metabolite adduct standard (centroided peaks, e.g. MassBank MSBNK-RIKEN-PR100295 for Pantothenic acid [M+H]+)
- MassBank or similar .msp-format spectral record (for the mspToLib import path)
- MetaboAnnotatoR package vignette example fragment library entry (.csv) as the validation reference

## Outputs

- MetaboAnnotatoR .csv metabolite fragment library entry with per-peak occurrence scores
- Comparison report of generated vs vignette reference entry (matching fragment m/z and occurrence scores)

## How to apply

Load the experimental MS/MS spectrum (or .msp record, e.g. the Pantothenic acid [M]+H MassBank entry MSBNK-RIKEN-PR100295) into R and run MetaboAnnotatoR's genFragEntry on it; genFragEntry attributes occurrence scores to 'marker peaks', i.e. peaks above the mpeaksThres threshold and above the noise level, with metabolite name, adduct, accurate adduct m/z and output filename explicitly defined. Write the scored peak list as a MetaboAnnotatoR .csv library entry; alternatively import the .msp record with mspToLib, which the package supports for external libraries ('by importing external libraries as .txt and .msp files'). Compare the generated .csv entry against the vignette's example entry (the Pantothenic acid [M+H]+ example included with the package) — fragment m/z values and occurrence scores must match; discrepancies indicate incorrect occurrence attribution. Note that the repository README explicitly provides a vignette titled 'Generation of Metabolite fragment database entry from MS/MS experimental spectra', which is the intended reference target for this validation.

## Related tools

- **MetaboAnnotatoR** (R package providing genFragEntry (occurrence scoring of marker peaks from experimental MS/MS) and mspToLib (.msp import); its vignettes supply the reference library entries used for validation) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment in which MetaboAnnotatoR is installed via devtools and the library-entry generation/validation scripts are executed) — https://github.com/gggraca/MetaboAnnotatoR

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); entry <- genFragEntry(spectrum = MSBNK_RIKEN_PR100295, name = "Pantothenic acid", adduct = "[M+H]+", outFile = "pantothenic_acid_MpH.csv")
```

## Evaluation signals

- Generated .csv library entry fragment m/z values match the vignette example entry for the same adduct (e.g. Pantothenic acid [M+H]+) exactly.
- Occurrence scores assigned by genFragEntry match the vignette reference scores for every marker peak.
- Peaks below mpeaksThres or below the noise level are excluded from the entry (only 'marker peaks' are scored).
- The entry's metabolite name, adduct label, and accurate adduct m/z fields are populated and consistent with the source spectrum's record.
- mspToLib path (for .msp input) yields a functionally equivalent entry to the genFragEntry path on the same spectrum.

## Limitations

- Validation is only as good as the reference vignette entry; new metabolites without a curated example cannot be validated this way.
- MetaboAnnotatoR requires raw LC-MS AIF chromatograms in centroid mode; profile-mode data must be transformed first.
- Only AIF is currently supported — other DIA schemes such as SWATH are not validated; Orbitrap-based (higher mass accuracy) annotation is possible but was not tested in the paper.
- Installation can fail with dependency errors (e.g. 'mzR has been built against a different Rcpp version'), typically fixable via Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS="true") or --no-multiarch/--no-lock install options.

## Evidence

- [results] genFragEntry attributes occurrence scores to peaks above mpeaksThres threshold: [evidence span withheld — anchor status: not_found]
- [discussion] .msp import via mspToLib: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [readme] vignette for library entry generation from experimental spectra: [evidence span withheld — anchor status: external_doc]
- [readme] centroid-mode raw data requirement: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [discussion] AIF-only scope (no SWATH): "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH."
- [readme] installation dependency error workaround: [evidence span withheld — anchor status: external_doc]
