---
name: msms-spectra-database-searching
description: Use when LC-MS/MS spectra of metabolites or other small molecules must
  be identified at the structure level by searching a molecular structure database
  such as PubChem, and measured spectral-library matching (e.g. GNPS library cosine
  search) is insufficient or unavailable.
when_to_use_negative:
- The goal is de novo molecular formula determination only — SIRIUS isotope pattern
  analysis / fragmentation tree computation alone suffices without CSI:FingerID structure
  search.
- Input spectra are MS1-only (no tandem MS peaks); CSI:FingerID requires MS/MS fragmentation
  information.
- A curated experimental MS/MS spectral library (e.g. GNPS library search) confidently
  matches the spectra, making fingerprint-based database search unnecessary.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4 / SIRIUS CLI
  role: Java-based framework that deduces molecular formulas via isotope pattern analysis
    and fragmentation tree computation, then uploads spectra to CSI:FingerID for structure
    search; version 6.5.4 CLI used for the GNPS cross-validation replication
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Web service integrated into SIRIUS that predicts molecular properties/fingerprints
    from fragmentation trees and searches them in molecular structure databases such
    as PubChem to rank candidate structures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:b27918f6c7ab156199fcba63c4949d850edba1ac1603c0bf7c4bb043d714a004
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  - SIRIUS 4 / SIRIUS CLI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# msms-spectra-database-searching

## Summary

Searches tandem MS/MS spectra of small molecules against molecular structure databases (e.g. PubChem) via SIRIUS + CSI:FingerID, which predicts structural fingerprints from fragmentation trees and ranks candidate structures. Enables structure identification even when no measured MS/MS reference spectra exist in a spectral library.

## When to use

Use when LC-MS/MS spectra of metabolites or other small molecules must be identified at the structure level by searching a molecular structure database such as PubChem, and measured spectral-library matching (e.g. GNPS library cosine search) is insufficient or unavailable.

## When NOT to use

- The goal is de novo molecular formula determination only — SIRIUS isotope pattern analysis / fragmentation tree computation alone suffices without CSI:FingerID structure search.
- Input spectra are MS1-only (no tandem MS peaks); CSI:FingerID requires MS/MS fragmentation information.
- A curated experimental MS/MS spectral library (e.g. GNPS library search) confidently matches the spectra, making fingerprint-based database search unnecessary.

## Inputs

- LC-MS/MS tandem mass spectra of small molecules (peak lists with precursor mass, e.g. from SIRIUS-readable MS2 data)
- Molecular formula candidates produced by SIRIUS isotope pattern and fragmentation tree analysis
- Structure database to search (PubChem)
- Correct structure labels per query spectrum (for evaluating the top-k identification curve)

## Outputs

- Ranked top-k candidate structures per query spectrum with ranks and scores
- Top-k identification curve (k vs. fraction of queries with correct structure in top-k)
- identification_curve.csv containing k values and corresponding correct-identification fractions

## How to apply

First determine molecular formulas for each MS/MS query spectrum using SIRIUS (isotope pattern analysis on high-resolution MS1 peaks plus fragmentation tree computation on the MS/MS peaks). Then submit fragmentation trees and spectra from the SIRIUS command-line tool to the CSI:FingerID web service, which predicts molecular fingerprints (ECFP-based) that are matched against a structure database such as PubChem — note that ECFP fingerprints now reside in the CSI:FingerID database so they no longer need to be computed on the user's side. Retrieve the top-k ranked structure candidates for each query, recording ranks and scores. To evaluate performance, compute the top-k identification curve — for each k, the fraction of query spectra whose correct structure falls within the top-k candidates — and verify that at k=1 the identification rate matches the expected value (40.4% for CSI:FingerID 1.1 in structure-disjoint cross-validation on GNPS under the original CSI:FingerID evaluation setup). Save the curve (k values vs. correct-identification fractions) to identification_curve.csv.

## Related tools

- **SIRIUS 4 / SIRIUS CLI** (Java-based framework that deduces molecular formulas via isotope pattern analysis and fragmentation tree computation, then uploads spectra to CSI:FingerID for structure search; version 6.5.4 CLI used for the GNPS cross-validation replication) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Web service integrated into SIRIUS that predicts molecular properties/fingerprints from fragmentation trees and searches them in molecular structure databases such as PubChem to rank candidate structures)

## Evaluation signals

- At k=1, the fraction of query spectra with the correct structure ranked first matches the expected 40.4% in structure-disjoint cross-validation on GNPS using the original CSI:FingerID evaluation setup.
- The top-k identification curve is monotonically non-decreasing in k, with each k value's fraction computed over the same set of query spectra.
- identification_curve.csv contains the expected schema: one row per k with the k value and the corresponding correct-identification fraction.
- Identification rates are in the reported regime for SIRIUS 4 on challenging metabolomics datasets (more than 70%).
- Every query spectrum has a molecular formula assigned by SIRIUS before being submitted to CSI:FingerID.

## Limitations

- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are hosted by the Böcker group and are for academic research and education use only; commercial use requires a Bright Giant license.
- Identification rates depend on the structure database searched — the replicated 40.4% GNPS figure corresponds to searching in PubChem under the original CSI:FingerID release setup.
- The SIRIUS 6.x isotope pattern analysis was rewritten and described only as 'hopefully more stable than before'; support for in-source fragmentations and adducts is labeled experimental.

## Evidence

- [abstract] CSI:FingerID structure search: "SIRIUS 4 integrates CSI:FingerID for searching in molecular structure databases."
- [methods] spectra upload to CSI:FingerID web service: "Fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID, CANOPUS and MSNovelist web services."
- [readme] SIRIUS scope: LC-MS/MS analysis of small molecules: [evidence span withheld — anchor status: external_doc]
- [discussion] ECFP fingerprints served by CSI:FingerID database: "**ECFP fingerprints** are now also in the CSI:FingerID database and do no longer have to be"
