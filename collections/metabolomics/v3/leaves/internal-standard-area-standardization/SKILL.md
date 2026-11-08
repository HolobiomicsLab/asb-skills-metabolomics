---
name: internal-standard-area-standardization
description: Use when raw integrated component areas (Component.Area) from GC-MS samples
  must be normalized against internal or external standards before downstream analyses
  such as NMDS, PCA, or ANOSIM.
when_to_use_negative:
- No internal or external standard was co-analyzed with the sample batch, so no reference
  area exists to standardize against
- The input is not raw component areas but already-normalized abundance data or a
  curated feature table
- Data are MS2, MS3, or ion-mobility MS2 spectra, which uafR's published-data-dependent
  algorithms do not support
tools:
- name: uafR
  role: R package providing spreadOut(), mzExacto(), and standardifyIt() for extracting
    and standardizing GC-MS component areas
  repo: github.com/castratton/uafR
- name: GC-MS
  role: Instrument platform whose peak tables (e.g. from Agilent Unknowns Analysis
    deconvolution) supply the Component.Area values to be standardized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:3c314488e38c943287ad2b2f4b712dc666f8b3c18c9823da244d0f894566374b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GC-MS
  - LOTUS
  - R
  - uafR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# internal-standard-area-standardization

## Summary

Standardizes raw GC-MS component areas (from uafR's mzExacto() output) by expressing each tentative compound's area relative to known internal or external standards, producing a standardized-area table suitable for cross-sample statistical comparison.

## When to use

Use when raw integrated component areas (Component.Area) from GC-MS samples must be normalized against internal or external standards before downstream analyses such as NMDS, PCA, or ANOSIM.

## When NOT to use

- No internal or external standard was co-analyzed with the sample batch, so no reference area exists to standardize against
- The input is not raw component areas but already-normalized abundance data or a curated feature table
- Data are MS2, MS3, or ion-mobility MS2 spectra, which uafR's published-data-dependent algorithms do not support

## Inputs

- GC-MS aggregated component-area table from uafR mzExacto() (compound identities, retention times, areas per sample)
- Identity and raw areas of one or more known internal or external standards measured in each sample or batch

## Outputs

- Standardized-area table (each tentative compound's area expressed relative to the internal/external standard per sample)
- Standardized abundance values ready for downstream PCA, NMDS, or machine learning analyses

## How to apply

After aggregating component areas with mzExacto() on the spreadOut()-prepared hit table (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor), divide or scale each tentative compound's area by the measured area of a known internal or external standard run in the same batch. Use the uafR standardifyIt() function (listed alongside spreadOut() and mzExacto() in the GC-MS processing workflow) to build the standardized-area table. Choose a standard that is present and stable across all 35+ samples in the batch; the rationale is that raw component areas vary with injection volume and instrument drift, so relative quantification removes these biases before statistical analysis. Verify that standard areas are themselves consistent across samples before applying the normalization, and retain the un-standardized table so the normalization is auditable.

## Related tools

- **uafR** (R package providing spreadOut(), mzExacto(), and standardifyIt() for extracting and standardizing GC-MS component areas) — github.com/castratton/uafR
- **GC-MS** (Instrument platform whose peak tables (e.g. from Agilent Unknowns Analysis deconvolution) supply the Component.Area values to be standardized)

## Evaluation signals

- The standard compound appears with a measured area in every sample in the batch; missing standard areas trigger exclusion or re-checking of that sample
- Standardized areas for known compounds remain proportional to injected concentration (analogy: standards validated with R2 0.8273-0.9998 across 1e-5 to 1e3 ng/μl)
- Row count of the standardized table equals the number of compounds retained after mzExacto(decontaminate = TRUE) filtering
- Downstream NMDS/ANOSIM on standardized areas reproduces the significance pattern reported for the manual peak-annotation analysis

## Limitations

- Researchers should still validate that the standardized compound areas make chemical and/or biological sense in their study system; algorithmic standardization does not remove the need for manual review
- Compounds dropped by decontamination (no strong cross-sample match, no PubChem entry, no canonical SMILES) will also be absent from the standardized table, so standardization cannot rescue unvalidated identifications
- uafR has only been validated on GC-MS data from instruments producing Unknowns Analysis-style hit tables; LC-MS or Q-TOF/Orbitrap inputs are untested

## Evidence

- [intro] standardize component areas relative to known standards: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [readme] uafR workflow includes standardifyIt for abundance standardization: [evidence span withheld — anchor status: external_doc]
- [results] standard-area validation supports proportionality: "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
- [discussion] manual validation of compound areas still required: "researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system"
