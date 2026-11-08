---
name: technical-replicate-peak-filtering
description: Use when processing DIMS SIM-stitched or LC-MS XCMS peak lists where
  each biological sample has been injected as technical replicates and low-confidence
  peaks detected inconsistently across injections must be removed prior to alignment.
when_to_use_negative:
- There are no technical replicate injections in the study design (each sample measured
  once), since the x-out-of-n criterion cannot be evaluated and the filter would be
  inapplicable.
- Input is a single consolidated sample peak table rather than a replicate-resolved
  matrix — Sample Filtering (x-out-of-n biological samples) or Blank Filtering is
  the appropriate check instead.
- The goal is metabolite annotation/identification (e.g. MI-Pack empirical formula
  search) on an already replicate-filtered peak list; re-filtering here would not
  change identifications.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Galaxy-M
  role: Hosts the Replicate Filtering tool as part of the DIMS and LC-MS workflows
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: MI-Pack
  role: Metabolite annotation package executed downstream of replicate-filtered peak
    lists in Galaxy-M
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy
  role: Workflow platform in which Replicate Filtering is chained with SIM-stitching
    and Align Samples steps
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:049f0589000b0060c5fb282e842a8b6d1956a7b88234438b4606a10528b6a608
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# technical-replicate-peak-filtering

## Summary

Replicate Filtering in the Galaxy-M DIMS/LC-MS workflow removes peaks that fail to appear in a user-defined x-out-of-n proportion of technical replicate injections, so that only reproducibly detected peaks survive into sample alignment. It is a reproducibility-based data-cleansing step applied to the replicate peak matrix before Blank/Sample filtering and alignment.

## When to use

Use when processing DIMS SIM-stitched or LC-MS XCMS peak lists where each biological sample has been injected as technical replicates and low-confidence peaks detected inconsistently across injections must be removed prior to alignment.

## When NOT to use

- There are no technical replicate injections in the study design (each sample measured once), since the x-out-of-n criterion cannot be evaluated and the filter would be inapplicable.
- Input is a single consolidated sample peak table rather than a replicate-resolved matrix — Sample Filtering (x-out-of-n biological samples) or Blank Filtering is the appropriate check instead.
- The goal is metabolite annotation/identification (e.g. MI-Pack empirical formula search) on an already replicate-filtered peak list; re-filtering here would not change identifications.

## Inputs

- Galaxy-M replicate peak matrix from DIMS SIM-stitching (m/z peak list per technical replicate) or XCMS-aligned LC-MS feature table
- User-specified x value for the x-out-of-n replicate criterion
- Number of technical replicates per biological sample (n)

## Outputs

- Filtered peak matrix / peak list retaining only peaks present in at least x-out-of-n technical replicates of each sample
- CSV-format peak table suitable for downstream Blank Filtering, Sample Filtering and Align Samples steps

## How to apply

Run Replicate Filtering on the peak-vs-sample matrix produced by the DIMS SIM-stitching chain (Sum Transients, Process Transients, Mass Calibration and SIM-stitching) or by the XCMS LC-MS pipe, where technical replicates of each biological sample are recorded as adjacent columns. For each peak, count the replicates of each sample in which it is detected and remove any peak that fails to appear in at least x-out-of-n technical replicates, with x chosen by the analyst and n being the number of replicates per sample. The rationale is that peaks present in only one replicate of a sample are more likely to be noise or artefacts of transient processing (e.g. Hanning apodization, Fourier transformation, baseline correction, peak picking at an ~3.5:1 signal-to-noise threshold) than true metabolite signals, so removing them increases confidence in downstream sample alignment and statistics. Because x is user-chosen, tune it against the replicate design: stricter (higher) x yields a smaller, more reproducible peak list but discards low-abundance metabolites. In the Galaxy-M workflow this tool sits between SIM-stitching (or XCMS alignment) and downstream filtering/alignment steps such as Blank Filtering, Sample Filtering and the final Align Samples CSV export.

## Related tools

- **Galaxy-M** (Hosts the Replicate Filtering tool as part of the DIMS and LC-MS workflows) — https://github.com/Viant-Metabolomics/Galaxy-M
- **MI-Pack** (Metabolite annotation package executed downstream of replicate-filtered peak lists in Galaxy-M) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy** (Workflow platform in which Replicate Filtering is chained with SIM-stitching and Align Samples steps) — https://github.com/galaxyproject/galaxy

## Examples

```
# In Galaxy-M, run the Replicate Filtering tool on the SIM-stitched peak matrix with user-chosen x (e.g. x=2 of n=3 replicates) before Align Samples export
```

## Evaluation signals

- After filtering, every retained peak should be detected in at least x of the n technical replicates for each sample — recompute replicate counts per peak and confirm none fall below the threshold.
- The filtered output must contain a subset of the input peak rows: m/z values removed by the filter should exactly match rows whose replicate counts are < x.
- Downstream Align Samples output (aligned peak table CSV) should only contain peaks that survived replicate filtering, so any unfiltered m/z appearing there indicates a pipeline wiring error.
- Varying x should monotonically change the retained peak count (higher x ⇒ fewer peaks), which can be used to sanity-check that the parameter is actually being honoured.

## Limitations

- The x threshold is chosen by the user; the article does not prescribe a universal default, so an inappropriate x can either retain noisy peaks or discard genuine low-abundance metabolites.
- No univariate statistics tools are included in the current Galaxy-M implementation, so replicate filtering cannot be followed immediately by per-peak significance testing inside the same workflow.
- The DIMS transient (.dat) files needed upstream of this filter are not included in the MetaboLights accession and must be obtained from the original authors.

## Evidence

- [abstract] Replicate Filtering tool removes peaks not reproducibly detected across technical replicate injections: "Replicate Filtering: filters peaks that fail to appear in at least x-out-of-n technical replicates"
- [abstract] DIMS SIM-stitching precedes replicate filtering, producing m/z peak lists per sample from frequency-domain transients: [evidence span withheld — anchor status: not_found]
- [readme] README documents the SimStitch environment (32-bit WINE, MSFileReader) needed to open Thermo DIMS data feeding the replicate-filter step: "you will likely get errors from the SimStitch tools along the lines of 'Invalid file identifier' - this is because they can't use the MSFileReader tool to open the DIMS data."
