---
name: fragment-frequency-filtering
description: Use when consolidating multiple deconvolved MS2 spectra (DDA or SWATH-DIA)
  that belong to the same MS1 peak across replicates, and noisy or inconsistently
  detected fragments must be removed before building a consensus spectrum.
when_to_use_negative:
- Input MS2 spectra are not deconvolved or are chimeric — run PerformDDADeconvolution
  or PerformDIADeconvolution first
- Only a single replicate spectrum exists per MS1 feature — replicate frequency thresholds
  cannot be meaningfully applied
- Input is already a consensus spectrum or a feature/annotation table rather than
  raw replicate MS2 spectra
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformSpectrumConsenus and the frequency-based replicate consensus
    workflow with optional database-assisted fragment rescue
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the consensus and filtering functions are
    executed
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:7dfb0047f5d3089ba9e269b4265fb2a1311844ae4418c3e3bc94f4a828b8b85b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# fragment-frequency-filtering

## Summary

Frequency-based fragment filtering in MetaboAnalystR 4.0 reduces noise when merging replicate MS2 spectra of the same MS1 feature into a single consensus spectrum via PerformSpectrumConsenus.

## When to use

Use when consolidating multiple deconvolved MS2 spectra (DDA or SWATH-DIA) that belong to the same MS1 peak across replicates, and noisy or inconsistently detected fragments must be removed before building a consensus spectrum.

## When NOT to use

- Input MS2 spectra are not deconvolved or are chimeric — run PerformDDADeconvolution or PerformDIADeconvolution first
- Only a single replicate spectrum exists per MS1 feature — replicate frequency thresholds cannot be meaningfully applied
- Input is already a consensus spectrum or a feature/annotation table rather than raw replicate MS2 spectra

## Inputs

- Deconvolved MS2 spectra from PerformDDADeconvolution or PerformDIADeconvolution, grouped by MS1 feature across replicates
- User-defined fragment frequency threshold (e.g., 50%)
- Optional: SQLite MS2 reference spectra library providing the reference fragment list (L) per precursor

## Outputs

- One normalized consensus MS2 spectrum per replicate-grouped MS1 feature
- consensus_spectra.msp file for downstream reference-library searching

## How to apply

Group all replicate MS2 spectra corresponding to the same MS1 peak after deconvolution (output of PerformDDADeconvolution or PerformDIADeconvolution). Count each MS2 fragment across replicates and keep a fragment only if its frequency exceeds a user-defined threshold (e.g., 50%); otherwise remove it. Optionally enable the database-assisted consensus rule: for each sub-threshold fragment, search its m/z against the reference fragment list (L) extracted from the SQLite spectra library for that precursor, and retain it only if found in L and observed in more than 2 replicates. Normalize the intensities of the surviving fragments and merge them into one consensus spectrum per MS1 feature, exported with PerformSpectrumConsenus to consensus_spectra.msp. The rationale is to reduce errors and noise arising from replicate-to-replicate variability before downstream reference-library searching via PerformDBSearchingBatch.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformSpectrumConsenus and the frequency-based replicate consensus workflow with optional database-assisted fragment rescue) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the consensus and filtering functions are executed) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
consensus <- PerformSpectrumConsenus(mset, format="msp", ConsensusFragThreshold=0.5); # merges replicate MS2 spectra, filters fragments by 50% frequency, exports consensus_spectra.msp
```

## Evaluation signals

- consensus_spectra.msp exists and parses as a valid msp file with one consensus spectrum per replicate-grouped MS1 feature
- No fragment below the user-defined frequency threshold (e.g., 50%) is present, unless rescued by the database-assisted rule with frequency > 2 replicates and match to the SQLite reference fragment list L
- Remaining fragment intensities are normalized before merging into the consensus spectrum
- Consensus spectra feed PerformDBSearchingBatch and yield compound identifications without excess spurious fragment matches

## Limitations

- The frequency threshold is user-defined (e.g., 50%), so overly aggressive settings may remove true fragments that vary across replicates
- The database-assisted rescue rule (frequency > 2 plus match to reference list L) depends on the quality and coverage of the SQLite reference spectra library; built-in spectra are compiled from various instruments and experimental conditions with potentially varying qualities
- Filtering operates on replicate-grouped spectra of the same MS1 feature; it does not deconvolve chimeric spectra, which is handled upstream by PerformDDADeconvolution or PerformDIADeconvolution

## Evidence

- [methods] The consensus step merges all replicate MS2 spectra of an MS feature into one spectrum, and may optionally exclude fragments detected only once: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [methods] Spectrum consensus function: "The spectrum consensus can be achieved with the function, PerformSpectrumConsenus."
- [methods] Deconvolution upstream: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [discussion] Reference library quality caveat: "compiled from various instruments and experimental conditions, resulting in potentially varying qualities"
- [readme] Pipeline scope from README: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
