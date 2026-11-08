---
name: msms-peak-list-processing
description: Use when LC-MS/MS runs in mzML or mzXML format must be converted into
  cleaned, feature-merged peak lists feeding molecular formula determination or structure
  annotation with SIRIUS 4 / COSMIC.
when_to_use_negative:
- The input LC-MS run contains spectra in profiled mode or only negative ion mode
  — such runs are explicitly discarded and cannot produce valid peak lists for COSMIC
- The task is feature detection or molecular formula de novo determination itself
  rather than preparation of cleaned, merged peak lists
- Structures beyond 700 Da are targeted, since compounds with mass beyond 700 Da were
  discarded to avoid high running time
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS 4
  role: Collects fragmentation spectra and precursor information, searches for associated
    features, and merges spectra per feature via agglomerative cosine-distance clustering
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Downstream workflow consuming the processed peak lists for confidence-scored
    structural annotation
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:23d8b51555b913ee0fa0183374297160894d7cf63cc9924def18f25384aa9c03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# msms-peak-list-processing

## Summary

Preprocess LC-MS/MS spectra into peak lists for structural annotation with SIRIUS 4 and COSMIC: filter raw mzML/mzXML runs, remove degenerate spectra, merge per-feature fragmentation spectra by cosine clustering, and bin peak intensities for similarity and distortion measurements.

## When to use

Use when LC-MS/MS runs in mzML or mzXML format must be converted into cleaned, feature-merged peak lists feeding molecular formula determination or structure annotation with SIRIUS 4 / COSMIC.

## When NOT to use

- The input LC-MS run contains spectra in profiled mode or only negative ion mode — such runs are explicitly discarded and cannot produce valid peak lists for COSMIC
- The task is feature detection or molecular formula de novo determination itself rather than preparation of cleaned, merged peak lists
- Structures beyond 700 Da are targeted, since compounds with mass beyond 700 Da were discarded to avoid high running time

## Inputs

- LC-MS/MS run files in .mzML or .mzXML format
- MassIVE datasets (e.g. MSV000082973, MSV000080553) provided as mzML/mzXML
- Reference tandem MS/MS spectra as peak lists (m/z and intensity pairs)

## Outputs

- Feature-associated, merged fragmentation spectra per precursor ion
- Cleaned peak lists with degenerate spectra (at most one peak) removed
- Binned peak-intensity vectors for cosine-similarity computation
- Distorted spectra with noise-level annotations (e.g. distorted_spectra.mgf)

## How to apply

Start from LC-MS/MS runs in mzML or mzXML format and discard any run containing spectra in profiled mode, since COSMIC/SIRIUS require centroided peak lists. For each run, let SIRIUS collect all fragmentation spectra with their precursor information and search for the associated features (precursor ion, adduct, isotope peaks). Merge fragmentation spectra assigned to the same feature using an agglomerative clustering algorithm based on cosine distance, because annotation rates are consistently better with merged spectra than with single spectra. Remove fragmentation spectra with at most one peak; when preparing evaluation libraries, discard the compound entirely if a spectrum with at most one peak results for any collision energy or noise level, and use only compounds with all three collision energies available. Peak lists are handled as binned peak-intensity vectors when computing cosine similarity, e.g. between distorted and original spectra where the median similarity is tuned to 0.880 (medium noise) or 0.714 (high noise). Molecular formula determination then proceeds de novo from isotope and fragmentation patterns with a 10 ppm mass accuracy.

## Related tools

- **SIRIUS 4** (Collects fragmentation spectra and precursor information, searches for associated features, and merges spectra per feature via agglomerative cosine-distance clustering) — https://github.com/sirius-ms/sirius
- **COSMIC** (Downstream workflow consuming the processed peak lists for confidence-scored structural annotation) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input myrun.mzML --project myproject --merge-frag-spectra
```

## Evaluation signals

- No input run in profiled mode or negative ion mode reaches the merged-spectra stage; 17,414 LC-MS/MS runs remain after discarding such runs
- Every fragmentation spectrum retained for annotation has more than one peak
- For every distorted spectrum, the median cosine similarity to the original spectrum (binned peak-intensity vectors) is 0.880 for medium and 0.714 for high distortion levels
- Fragmentation spectra assigned to the same feature (precursor ion) are merged into a single spectrum per feature
- Compounds used in library evaluation have all three collision energies available and no spectrum with at most one peak at any energy or noise level

## Limitations

- Stereochemistry cannot be established from fragmentation spectra, so only the two-dimensional structure is considered downstream
- Small structure modifications are hard and potentially impossible to tell apart using MS/MS data alone
- Compounds with mass beyond 700 Da must be discarded to keep running time tractable

## Evidence

- [discussion] SIRIUS first collects all fragmentation spectra and their precursor information and then searches for features that are associated with those fragmentation spectra: "SIRIUS first collects all fragmentation spectra and their precursor information and then searches for features that are associated with those fragmentation spectra"
- [discussion] Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance.: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] Only files with extensions '.mzML' or '.mzXML' were considered, and LC–MS runs containing spectra in profiled mode were discarded.: [evidence span withheld — anchor status: not_found]
- [discussion] LC–MS/MS runs containing spectra in profiled or negative ion mode were discarded, leading to 17,414 LC–MS/MS runs being processed: "LC–MS/MS runs containing spectra in profiled or negative ion mode were discarded, leading to 17,414 LC–MS/MS runs being processed"
- [discussion] Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra.: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] we removed fragmentation spectra with, at most, one peak: "we removed fragmentation spectra with, at most, one peak"
- [discussion] we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
- [discussion] Compounds with mass beyond 700 Da were discarded to avoid high running time.: "Compounds with mass beyond 700 Da were discarded to avoid high running time."
- [discussion] We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm.: "We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm."
- [supplementary] For every distorted spectrum, compute the cosine similarity against its original spectrum (binned peak-intensity vectors) and take the median per distortion level.: [evidence span withheld — anchor status: not_found]
