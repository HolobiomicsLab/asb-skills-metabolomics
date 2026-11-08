---
name: ms2-peak-intensity-normalization
description: Use when replicate-grouped, deconvolved MS2 spectra (e.g., DDA or SWATH-DIA
  output) must be merged into one consensus spectrum per MS1 peak and fragment intensities
  need normalization before MSP export.
when_to_use_negative:
- Input is already a merged/consensus spectrum or a clean feature abundance table
  — normalization for statistical analysis is a separate MetaboAnalystR module
- MS2 spectra come from a single replicate (no replicate group) — the frequency-based
  filtering rationale does not apply
- Only MS1 peak-picking/quantification is needed; this skill belongs to the MS2 deconvolution-to-annotation
  pipeline
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3395
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformSpectrumConsenus, which implements frequency-based fragment
    filtering, intensity normalization, and merging of replicate MS2 spectra into
    consensus_spectra.msp
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the consensus/normalization workflow is executed
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:fe0d1bb5c8a14fd32febf70629264b4be32aaffe2e3d55a3267d046d262fd4a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ms2-peak-intensity-normalization

## Summary

Normalizes the fragment-ion intensities of replicate MS2 spectra within MetaboAnalystR 4.0's spectrum-consensus step, after frequency-based fragment filtering, so that all replicate MS2 spectra of an MS1 feature can be merged into a single consensus spectrum. It reduces noise and errors in downstream MS2-based compound identification.

## When to use

Use when replicate-grouped, deconvolved MS2 spectra (e.g., DDA or SWATH-DIA output) must be merged into one consensus spectrum per MS1 peak and fragment intensities need normalization before MSP export.

## When NOT to use

- Input is already a merged/consensus spectrum or a clean feature abundance table — normalization for statistical analysis is a separate MetaboAnalystR module
- MS2 spectra come from a single replicate (no replicate group) — the frequency-based filtering rationale does not apply
- Only MS1 peak-picking/quantification is needed; this skill belongs to the MS2 deconvolution-to-annotation pipeline

## Inputs

- Deconvolved MS2 spectra from PerformDDADeconvolution or PerformDIADeconvolution, grouped by MS1 feature across replicates
- User-defined fragment frequency threshold (e.g., 50%)
- Optional: reference spectra list (L) extracted from the SQLite MS2 reference library for each precursor (for database-assisted consensus)

## Outputs

- One consensus MS2 spectrum per replicate-grouped MS1 feature with normalized fragment intensities
- consensus_spectra.msp file exported by PerformSpectrumConsenus for downstream database searching

## How to apply

Load the deconvolved MS2 spectra produced by PerformDDADeconvolution or PerformDIADeconvolution and group all spectra belonging to the same MS1 peak across replicates. Summarize fragment occurrences across replicates and apply the user-defined frequency threshold (e.g., 50%): keep a fragment only if its frequency exceeds the threshold, otherwise remove it. Optionally enable the database-assisted consensus rule — search sub-threshold fragment m/z values against the reference spectra list (L) from the SQLite library for that precursor and rescue a fragment only if it is found in L and observed in more than 2 replicates. Then normalize the intensities of all remaining fragments and merge them into a single consensus spectrum per MS1 feature. Export the result with PerformSpectrumConsenus to consensus_spectra.msp for downstream PerformDBSearchingBatch reference-library searching. The rationale is that fragments seen inconsistently across replicates (or only in a few replicates) are error- or noise-prone and would degrade spectral matching.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformSpectrumConsenus, which implements frequency-based fragment filtering, intensity normalization, and merging of replicate MS2 spectra into consensus_spectra.msp) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the consensus/normalization workflow is executed) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
mSet <- PerformSpectrumConsenus(mSet)
```

## Evaluation signals

- consensus_spectra.msp exists and parses as a valid MSP file with one consensus spectrum per replicate-grouped MS1 feature
- No fragment below the 50% frequency threshold is present unless it was rescued by the database-assisted rule (found in reference list L and observed in more than 2 replicates)
- Fragment intensities in each consensus spectrum are normalized (comparable scale across all retained fragments)
- Each consensus spectrum's precursor corresponds to a single MS1 feature from the deconvolution output
- Downstream PerformDBSearchingBatch runs successfully against the exported MSP entries

## Limitations

- The frequency threshold is user-defined (e.g., 50%); an overly strict threshold can discard genuine low-reproducibility fragments, while a lenient one retains noise
- The database-assisted rescue depends on the completeness of the built-in MS2 reference libraries, which were compiled from various instruments and experimental conditions with potentially varying qualities
- The function name is spelled PerformSpectrumConsenus in the package (typo), so searches for 'Consensus' will miss it

## Evidence

- [methods] frequency threshold filtering: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [methods] consensus function: "The spectrum consensus can be achieved with the function, PerformSpectrumConsenus."
- [readme] module scope: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA);"
