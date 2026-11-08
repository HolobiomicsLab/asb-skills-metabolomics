---
name: tandem-ms-spectral-library-search
description: Use when annotating LC–MS/MS query spectra by direct matching to reference
  spectra (e.g., MassBank/GNPS/NIST 2017 or CSI:FingerID training structures), or
  when constructing the spectral library search baseline for benchmarking in silico
  annotation tools at controlled FDR.
when_to_use_negative:
- The query compound's structure has no reference spectrum in the library — in the
  COSMIC benchmark, 49% of hits had no reference spectra in the training data, and
  typically only a small fraction of spectra can be matched by library search; in
  silico search (COSMIC) is needed instead
- The goal is de novo structural annotation of metabolites absent from spectral libraries
  rather than de-replication against known reference spectra
- Query spectra are in profiled mode or lack sufficient fragmentation peaks (spectra
  with at most one peak were removed in the benchmark)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Determines the molecular formula de novo (10 ppm mass accuracy) and computes
    fragmentation trees, allowing library/candidate entries to be restricted to those
    sharing the query's molecular formula
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates under the formula restriction, providing the in
    silico annotation pipeline that the spectral library search baseline is compared
    against
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Confidence-scored in silico structure annotation whose correct-hit counts
    at 5% FDR (1,415 at 20 eV; 1,701 merged) benchmark against this library-search
    baseline (52 and 1)
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:b15e01b4155b9335726df274195588e1db2d0a9fab4ae4a63d692c9c2895bce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID 1.2.0
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tandem-ms-spectral-library-search

## Summary

Annotates tandem MS/MS query spectra by scoring them against a reference spectral library using normalized dot-product cosine similarity (and its square-root-intensity-transformed variant), serving as the classical de-replication baseline against which in silico methods such as COSMIC are benchmarked at FDR 5%.

## When to use

Use when annotating LC–MS/MS query spectra by direct matching to reference spectra (e.g., MassBank/GNPS/NIST 2017 or CSI:FingerID training structures), or when constructing the spectral library search baseline for benchmarking in silico annotation tools at controlled FDR.

## When NOT to use

- The query compound's structure has no reference spectrum in the library — in the COSMIC benchmark, 49% of hits had no reference spectra in the training data, and typically only a small fraction of spectra can be matched by library search; in silico search (COSMIC) is needed instead
- The goal is de novo structural annotation of metabolites absent from spectral libraries rather than de-replication against known reference spectra
- Query spectra are in profiled mode or lack sufficient fragmentation peaks (spectra with at most one peak were removed in the benchmark)

## Inputs

- Independent query tandem MS/MS spectra in mzML/mzXML format (e.g., medium-noise QTOF data)
- Reference spectral library entries from MassBank, GNPS and NIST 2017, or CSI:FingerID training structures from the web API (positive/negative ion mode)
- Precursor information (precursor mass, adduct, isotope peaks) per fragmentation spectrum
- Collision-energy metadata (e.g., 20 eV, 40 eV, merged spectra conditions)

## Outputs

- Ranked library matches per query spectrum with cosine similarity scores (raw and square-root-intensity-transformed variants)
- Correct annotation counts at exact 5% FDR per scoring variant and condition
- Comparison table written to fdr5_correct_hits_comparison.csv (method x condition x correct hits at FDR 5%)

## How to apply

Build a reference library from entries such as the MassBank/GNPS/NIST 2017 combined dataset or the CSI:FingerID training structures retrieved from the web API (predictor=1 for positive, predictor=2 for negative ion mode). Score each query spectrum against the library with normalized dot-product cosine similarity, plus a variant where intensities are square-root transformed before scoring; for fair evaluation use a spectrum-disjoint setup so a query compound's own reference spectrum is excluded from the library during its evaluation. Restrict reference entries to those matching the query's ion mode and, when coupled to SIRIUS/CSI:FingerID, to candidates sharing the molecular formula of the top-scoring de novo formula assignment (10 ppm mass accuracy). Compute correct annotation counts at exact FDR 5% (FDR estimated from average posterior error probabilities of the top-k hits) separately per collision-energy condition (e.g., 20 eV) and for merged spectra, and tabulate method x condition x correct hits. In the COSMIC benchmark on independent medium-noise QTOF data, this baseline yielded 52 correct hits at 20 eV versus 1,415 for COSMIC, and 1 versus 1,701 with merged spectra.

## Related tools

- **SIRIUS 4** (Determines the molecular formula de novo (10 ppm mass accuracy) and computes fragmentation trees, allowing library/candidate entries to be restricted to those sharing the query's molecular formula) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates under the formula restriction, providing the in silico annotation pipeline that the spectral library search baseline is compared against) — https://github.com/sirius-ms/sirius
- **COSMIC** (Confidence-scored in silico structure annotation whose correct-hit counts at 5% FDR (1,415 at 20 eV; 1,701 merged) benchmark against this library-search baseline (52 and 1)) — https://github.com/sirius-ms/sirius

## Examples

```
python spectral_library_search.py --query qtof_msms.mzML --library csi_fingerid_trainingstructures --scoring dot_product --disjoint --fdr 0.05 --out fdr5_correct_hits_comparison.csv
```

## Evaluation signals

- At exact FDR 5%, the baseline must reproduce the reported counts: 52 correct hits for library search versus 1,415 for COSMIC at 20 eV, and 1 versus 1,701 with merged spectra
- Spectrum-disjoint check: for each query compound, its own reference spectrum is absent from the library during its evaluation
- Both scoring variants (normalized dot-product cosine and square-root-intensity-transformed) are computed and reported side by side
- Merged-spectra condition should yield equal or higher annotation counts than individual collision-energy conditions, consistent with the article's observation that merged spectra give better annotation rates
- Output CSV fdr5_correct_hits_comparison.csv has rows for each method x condition cell with integer correct-hit counts

## Limitations

- Typically only a small fraction of query spectra can be matched by spectral library search, since compounds absent from the library cannot be annotated at all
- COSMIC outperformed spectral library search for de-replication even though COSMIC was not designed or optimized for this task, so the baseline is weak for de novo structure annotation
- Accurate FDR estimation from small molecule fragmentation spectra is expected to remain highly challenging; a threshold such as 0.64 is practical guidance only, no guarantee in either direction
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone, and stereochemistry cannot be established from fragmentation spectra

## Evidence

- [discussion] Library composition from public reference databases: "we used a combined dataset from MassBank68, GNPS37 and the NIST 2017 database"
- [discussion] Baseline is outperformed by COSMIC even at its own de-replication game: "COSMIC clearly outperformed spectral library search for de-replication; this is notable, as COSMIC has not been designed or optimized for this task."
- [discussion] Merged spectra give the best annotation rates: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [readme] Reference library entries retrievable via CSI:FingerID web API: "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1 (training structures for positive ion mode)"
- [abstract] Only a small fraction of spectra are matchable by library search: "Untargeted metabolomics experiments rely on spectral libraries for structure annotation, but, typically, only a small fraction of spectra can be matched"
