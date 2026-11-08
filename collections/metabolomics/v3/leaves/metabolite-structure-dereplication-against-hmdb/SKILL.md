---
name: metabolite-structure-dereplication-against-hmdb
description: Use when a COSMIC/SIRIUS annotation table of structure candidates must
  be reduced to high-confidence novel structures absent from HMDB or spectral libraries,
  or when auditing per-cohort annotation counts.
when_to_use_negative:
- Input spectra have not been processed through SIRIUS 4/CSI:FingerID — there are
  no structure candidates or confidence scores to dereplicate
- Goal is direct spectral library search of raw MS/MS spectra (e.g. GNPS/MassBank/NIST
  matching) rather than counting novel in-silico structure annotations
- Stereochemistry or isomer distinction between closely related structures is required
  — only two-dimensional structures are considered
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: COSMIC
  role: Produces the confidence-scored structure annotations that are filtered at
    0.64 and dereplicated against HMDB
  repo: github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Preprocesses .mzML/.mzXML LC–MS/MS runs, assigns fragmentation spectra to
    features, merges them via agglomerative clustering on cosine distance, and determines
    molecular formulas de novo at 10 ppm
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates with default parameters, restricting candidates
    to the molecular formula of the top-scoring hit
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:174313037decd03e4ebf85a7bee11b11851a6a7d8d958e3239224a7df039b5af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-structure-dereplication-against-hmdb

## Summary

Filter COSMIC in-silico structure annotations to high confidence (score ≥ 0.64), deduplicate to unique structures, and exclude structures present in the HMDB structure database (and COSMIC training data) to count genuinely novel metabolite annotations.

## When to use

Use when a COSMIC/SIRIUS annotation table of structure candidates must be reduced to high-confidence novel structures absent from HMDB or spectral libraries, or when auditing per-cohort annotation counts.

## When NOT to use

- Input spectra have not been processed through SIRIUS 4/CSI:FingerID — there are no structure candidates or confidence scores to dereplicate
- Goal is direct spectral library search of raw MS/MS spectra (e.g. GNPS/MassBank/NIST matching) rather than counting novel in-silico structure annotations
- Stereochemistry or isomer distinction between closely related structures is required — only two-dimensional structures are considered

## Inputs

- COSMIC structure annotation table with confidence scores (per-feature structures from CSI:FingerID runs)
- HMDB structure database (downloaded 8 August 2018, 113,983 compounds)
- List of structures used in COSMIC/CSI:FingerID training data
- LC–MS/MS runs in .mzML or .mzXML format from the ten human MassIVE datasets (MSV000083559, MSV000079651, MSV000080167, MSV000080469, MSV000080533, MSV000080627, MSV000081351, MSV000082261, MSV000082629, MSV000082630) or the 123 Orbitrap MassIVE datasets

## Outputs

- Deduplicated list of high-confidence (score ≥ 0.64) structure annotations absent from HMDB/spectral libraries
- Per-cohort summary table: 436 high-confidence human annotations (315 without reference MS/MS data), 1,715 novel Orbitrap structure annotations
- Reconciliation of discarded-in-training counts (3,530 → 1,715 after removing 1,815 training structures)

## How to apply

Run SIRIUS 4 over the .mzML/.mzXML LC–MS/MS runs (positive ion mode, profiled-mode runs discarded; remove fragmentation spectra with at most one peak), determine molecular formulas de novo at 10 ppm, and rank candidates with CSI:FingerID against the structure database, discarding compounds beyond 700 Da. Map SVM decision values to Platt posterior probabilities, compute the COSMIC confidence score, and filter annotations at the 0.64 threshold, which roughly corresponds to FDR 10%. Deduplicate retained hits to unique structures, then exclude all hits whose structure appears in the HMDB structure database; when counting structures genuinely new to COSMIC, additionally discard annotations whose structure was present in the training data (this turned 3,530 Orbitrap hits into 1,715 novel ones, discarding 1,815). Finally, split human annotations into those without (315) and with (108) MS/MS reference data and compile a per-cohort summary table.

## Related tools

- **COSMIC** (Produces the confidence-scored structure annotations that are filtered at 0.64 and dereplicated against HMDB) — github.com/sirius-ms/sirius
- **SIRIUS 4** (Preprocesses .mzML/.mzXML LC–MS/MS runs, assigns fragmentation spectra to features, merges them via agglomerative clustering on cosine distance, and determines molecular formulas de novo at 10 ppm) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates with default parameters, restricting candidates to the molecular formula of the top-scoring hit) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Every retained annotation carries a confidence score ≥ 0.64
- No retained structure is present in the HMDB structure database; for novel-annotation counting, no retained structure appears in the COSMIC training data
- Annotations are deduplicated to unique structures before counting
- Per-cohort counts match reported values: 436 high-confidence human annotations including 315 without reference MS/MS data, and 1,715 novel Orbitrap annotations (3,530 high-confidence hits minus 1,815 training-data structures)
- Only profiled-free positive-ion-mode runs that survived filtering (17,414 Orbitrap LC–MS/MS runs) contributed to the counts

## Limitations

- The COSMIC confidence score must not be mistaken as the probability that an annotation is correct; it is not calibrated to be such
- The 0.64 threshold roughly corresponds to FDR 10% but is practical guidance only, not a guarantee; accurate FDR estimation for small-molecule fragmentation spectra is expected to remain highly challenging
- Small structure modifications are hard or potentially impossible to tell apart using MS/MS data alone, so distinct structures may be conflated after deduplication
- Compounds with mass beyond 700 Da were discarded to avoid high running time, so very large metabolites are never dereplicated here

## Evidence

- [abstract] Application of COSMIC to 17,400 metabolomics experiments yielded 1,715 novel annotations: "Application of COSMIC to data from 17,400 metabolomics experiments led to 1,715 high-confidence structural annotations that were absent from spectral libraries"
- [discussion] Filter annotations at score threshold 0.64 for high confidence: "We now filter the 24,554 structure annotations for high confidence (score threshold 0.64), resulting in 911 structure annotations."
- [discussion] Exclude HMDB-present structures to count novel annotations: "we excluded all hits with structures in the HMDB structure database"
- [readme] SIRIUS is the software framework analyzing the LC-MS/MS inputs dereplicated here: [evidence span withheld — anchor status: external_doc]
