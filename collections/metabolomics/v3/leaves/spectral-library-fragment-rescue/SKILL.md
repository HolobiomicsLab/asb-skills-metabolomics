---
name: spectral-library-fragment-rescue
description: Use when building consensus MS2 spectra per MS1 feature across replicates
  and you must decide whether sub-threshold (e.g. <50% frequency) fragments are noise
  or reproducible signal worth keeping.
when_to_use_negative:
- You already have a consensus spectrum or cleaned feature/fragment table and are
  performing compound identification (use PerformDBSearchingBatch instead)
- No reference MS2 spectra library is available for the precursor, since the rescue
  rule cannot be evaluated without the reference fragment list L
- Raw, undeconvolved DDA/DIA spectra — chimeric spectra must first be deconvolved
  before replicate grouping
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformSpectrumConsenus and the frequency-based fragment filtering
    with database-assisted rescue over the SQLite reference library
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the consensus and fragment-rescue workflow
    runs
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:fd3cadd709577fef527612120953c50eaeabf19a0577455956bfb9a2ae09d005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-library-fragment-rescue

## Summary

A database-assisted consensus rule from MetaboAnalystR 4.0 that rescues MS2 fragments failing the replicate-frequency filter by matching their m/z against a SQLite reference spectra library, retaining rescued fragments only when they appear in the reference list and in more than 2 replicates.

## When to use

Use when building consensus MS2 spectra per MS1 feature across replicates and you must decide whether sub-threshold (e.g. <50% frequency) fragments are noise or reproducible signal worth keeping.

## When NOT to use

- You already have a consensus spectrum or cleaned feature/fragment table and are performing compound identification (use PerformDBSearchingBatch instead)
- No reference MS2 spectra library is available for the precursor, since the rescue rule cannot be evaluated without the reference fragment list L
- Raw, undeconvolved DDA/DIA spectra — chimeric spectra must first be deconvolved before replicate grouping

## Inputs

- Deconvolved MS2 spectra from PerformDDADeconvolution or PerformDIADeconvolution, grouped by MS1 feature across replicates
- Fragment frequency table across replicates per MS1 feature
- SQLite reference MS2 spectra library (e.g. built from HMDB, MoNA, MassBank, GNPS, LipidBlast, MINEs msp files) queried for the precursor's reference fragment list L

## Outputs

- Consensus spectrum per MS1 feature with sub-threshold fragments either removed or rescued by the database rule
- consensus_spectra.msp file exported by PerformSpectrumConsenus for downstream reference library searching (PerformDBSearchingBatch)

## How to apply

After deconvolution (PerformDDADeconvolution or PerformDIADeconvolution), group all replicate MS2 spectra of the same MS1 peak and count each fragment's frequency across replicates. Fragments above the user-defined threshold (e.g., 50%) are kept; sub-threshold fragments are removed by default. The rescue step then searches each sub-threshold fragment's m/z against the reference spectra list (L) extracted from the SQLite reference library for that precursor: retain the fragment only if it is found in L AND observed in more than 2 replicates. Rationale: true fragments that are inconsistently sampled across replicates can be recovered using library evidence, while purely stochastic noise is suppressed, reducing errors in the final consensus spectrum exported via PerformSpectrumConsenus to consensus_spectra.msp.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformSpectrumConsenus and the frequency-based fragment filtering with database-assisted rescue over the SQLite reference library) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the consensus and fragment-rescue workflow runs) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
mSet <- PerformSpectrumConsenus(mSet)
```

## Evaluation signals

- consensus_spectra.msp exists and parses as a valid msp file with one consensus spectrum per replicate-grouped MS1 feature
- No fragment below the 50% frequency threshold is present unless it was rescued by the database-assisted rule (found in reference list L and observed in more than 2 replicates)
- Fragments above the 50% replicate-frequency threshold are retained in every consensus spectrum
- Each rescued fragment's m/z is verifiably present in the SQLite reference spectra list L for its precursor
- Rescued fragments occur in more than 2 replicates

## Limitations

- The rescue rule depends on the quality of the built-in MS2 reference spectra library, which is compiled from various instruments and experimental conditions, resulting in potentially varying qualities
- A sub-threshold fragment absent from the reference library L is discarded even if it is real but not yet cataloged
- The 50% frequency threshold is user-defined; inappropriate thresholds shift the noise/rescue trade-off

## Evidence

- [methods] frequency threshold with rescue context: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [methods] consensus function: "The spectrum consensus can be achieved with the function, PerformSpectrumConsenus."
- [discussion] reference library quality limitation: "compiled from various instruments and experimental conditions, resulting in potentially varying qualities"
- [methods] downstream database searching: "The database searching is performed via SQLite query and can be achieved with the function, PerformDBSearchingBatch."
