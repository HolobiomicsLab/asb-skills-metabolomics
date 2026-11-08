---
name: ms2-spectrum-quality-filtering
description: Use when building IIMN-based propagated spectral libraries from ion identity
  networks where only high-confident, high-quality MS2 consensus spectra should become
  library entries.
when_to_use_negative:
- When the goal is to match spectra against an existing library rather than curate/expand
  one — use a standard GNPS library search instead.
- When the analysis targets MS1-only feature grouping or ion identity annotation with
  no MS2 consensus spectra to filter.
- When MS2 spectra come from DDA acquisitions lacking the ≥3 signals above 0.1% relative
  intensity threshold; the filter would empty the candidate set.
tools:
- name: MZmine
  role: Exports the aligned feature quantification table, MS2 spectral summary .mgf,
    and IIN edges used as filter input
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Hosts the IIMN workflow and the spectral library being filtered/propagated
    (GNPS-IIMN-PROPOGATED)
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:9b17d154d555032446d85c95a596bf3719080026998e8fcf8ef10072cae48843
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ms2-spectrum-quality-filtering

## Summary

Filters candidate MS2 spectra for inclusion in ion-identity propagated spectral libraries by enforcing a minimum cosine similarity library match (≥0.9 with 4–6 shared fragment ions) plus a minimum of three signals above 0.1% relative intensity, producing clean consensus spectra for GNPS library expansion.

## When to use

Use when building IIMN-based propagated spectral libraries from ion identity networks where only high-confident, high-quality MS2 consensus spectra should become library entries.

## When NOT to use

- When the goal is to match spectra against an existing library rather than curate/expand one — use a standard GNPS library search instead.
- When the analysis targets MS1-only feature grouping or ion identity annotation with no MS2 consensus spectra to filter.
- When MS2 spectra come from DDA acquisitions lacking the ≥3 signals above 0.1% relative intensity threshold; the filter would empty the candidate set.

## Inputs

- Feature quantification table (.csv) from MZmine
- MS2 spectral summary file (.mgf)
- Supplementary IIN edges files (.csv)
- Existing GNPS spectral library (e.g. GNPS-IIMN-PROPOGATED, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE)

## Outputs

- Filtered set of IIN-internal features passing the cosine ≥0.9 / ≥4–6 shared fragment criterion
- Consensus MS2 spectra (≥3 signals above 0.1% relative intensity) exported as new library entries
- Propagated IIMN spectral library (GNPS-IIMN-PROPOGATED) with ~2657 total entries including 538 new propagated entries

## How to apply

Start from an aligned feature quantification table (.csv), the MS2 spectral summary (.mgf), and supplementary IIN edges (.csv) exported from MZmine for each dataset. First restrict to ion identity networks (IINs) containing a feature with a GNPS spectral library match at a minimum cosine similarity of 0.9 and a minimum of 4–6 shared fragment ions (or an authentic standard match). Within each matched IIN, retain only contained ion identity features that carry an MS2 spectrum with at least three signals above 0.1% relative intensity, then extract their consensus MS2 spectra as new library entries. The rationale is that propagating annotations only through high-confidence anchors and sufficiently populated spectra avoids injecting noisy or redundant species into the library. Filtering out nodes with poor MS2 spectra (fewer than four signals) is explicitly part of the pipeline to keep propagated spectra trustworthy.

## Related tools

- **MZmine** (Exports the aligned feature quantification table, MS2 spectral summary .mgf, and IIN edges used as filter input) — https://github.com/mzmine/mzmine
- **GNPS** (Hosts the IIMN workflow and the spectral library being filtered/propagated (GNPS-IIMN-PROPOGATED)) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Evaluation signals

- Every retained library entry traces to an IIN whose anchor feature matched GNPS libraries with cosine ≥0.9 and ≥4–6 shared fragment ions.
- Each exported consensus spectrum contains at least three signals above 0.1% relative intensity.
- Total propagated library size reaches ~2657 entries, with 538 newly propagated entries from the 24 public datasets plus the two NIH 'ACONN' datasets.
- Ion species coverage of the final library approximates the reported distribution: 24% [M + H]+, 22% multimeric, 17% [M + Na]+, 15% in-source fragments, 13% [M + NH4]+.
- No entry survives that originates from a node with fewer than four MS2 signals (poor-spectrum nodes filtered out).

## Limitations

- Requires IIN context — filtering is defined on ion identity networks, not on isolated features.
- Anchor matching depends on cosine similarity and shared fragment count thresholds (4–6 shared ions is a range, not a fixed value), so borderline anchors can change the retained set.
- Specific MZmine, MS-DIAL, or XCMS versions used for the 24 datasets are not stated in the text, hindering exact reproduction.
- Variable feature detection and alignment parameters across the 24 datasets are summarized only in Supplementary Table 4, not in the main text.
- No formal schema or validator tool is described for the IIN edge format (Supplementary Pairs text file).

## Evidence

- [results] IINs that contain a feature with a GNPS library match at cosine ≥0.9 and 4–6 shared fragments define the propagation seed set.: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [results] Consensus spectra admitted to the library must show at least three signals above 0.1% relative intensity.: "all contained ion identity features with an MS2 spectrum and at least three signals above 0.1% relative intensity were extracted as new library spectra"
- [discussion] The required input files for the filtering step are explicitly enumerated.: [evidence span withheld — anchor status: not_found]
- [discussion] Nodes with too few MS2 signals are explicitly excluded before library propagation.: "Filtering out nodes with poor MS2 spectra with less than four signals"
- [results] Filtering is performed within ion identity networks, not on isolated features.: [evidence span withheld — anchor status: not_found]
