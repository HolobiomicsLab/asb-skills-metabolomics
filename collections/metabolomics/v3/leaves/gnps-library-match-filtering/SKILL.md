---
name: gnps-library-match-filtering
description: Use when an aligned feature table with MS2 spectra (.mgf) and IIN edges
  is available and high-confidence GNPS library annotations must be identified and
  propagated to co-eluting ion species (adducts, multimers, in-source fragments) of
  the same compound.
when_to_use_negative:
- Input is raw MS data that has not yet been feature-found and exported to a quantification
  table plus .mgf — run the FBMN/IIMN workflow first.
- Goal is de novo annotation or molecular formula prediction (use SIRIUS/CANOPUS-style
  tools) rather than library-match-based spectral propagation.
- The network lacks ion identity edges (plain molecular networking without IIMN),
  so adducts/in-source fragments of the same compound are not connected and propagation
  would be unfounded.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Runs the spectral library search against GNPS libraries and hosts the propagated
    library (GNPS-IIMN-PROPOGATED) and FBMN/IIMN workflow
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Feature finding and IIMN processing; exports the feature quantification table
    (.csv), MS2 spectral summary (.mgf), and IIN edges files (.csv) consumed by the
    filter
  repo: https://github.com/mzmine/mzmine
- name: XCMS
  role: Alternative feature-finding tool for FBMN/IIMN input; xcms-gnps-tools provides
    formatSpectraForGNPS() for MGF export and adduct annotations for IIN
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:435757f0a75b34111bd6b31a0341656a2eb4bb8783074335c7d1d18a5e727cb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gnps-library-match-filtering

## Summary

Filters GNPS spectral library matches in ion identity molecular networks (IIMN) using cosine similarity and shared-fragment thresholds, then propagates high-confidence matches to connected ion-identity features to build expanded spectral library entries (e.g., GNPS-IIMN-PROPOGATED).

## When to use

Use when an aligned feature table with MS2 spectra (.mgf) and IIN edges is available and high-confidence GNPS library annotations must be identified and propagated to co-eluting ion species (adducts, multimers, in-source fragments) of the same compound.

## When NOT to use

- Input is raw MS data that has not yet been feature-found and exported to a quantification table plus .mgf — run the FBMN/IIMN workflow first.
- Goal is de novo annotation or molecular formula prediction (use SIRIUS/CANOPUS-style tools) rather than library-match-based spectral propagation.
- The network lacks ion identity edges (plain molecular networking without IIMN), so adducts/in-source fragments of the same compound are not connected and propagation would be unfounded.

## Inputs

- Aligned feature quantification table (.csv) from MZmine/MS-DIAL/XCMS
- MS2 spectral summary file (.mgf) with consensus spectra
- Supplementary IIN edges files (.csv) linking ion identity features
- GNPS spectral library search results (annotations per feature)

## Outputs

- Filtered set of high-confidence GNPS library matches (cosine >0.9, ≥4–6 shared fragments)
- Consensus MS2 spectra of retained ion-identity features as new library spectra entries
- Propagated IIMN spectral library (GNPS-IIMN-PROPOGATED) merged with GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE

## How to apply

Load the IIMN outputs of each dataset — the feature quantification table (.csv), the MS2 spectral summary (.mgf), and the supplementary IIN edges files (.csv) — from MZmine (or MS-DIAL/XCMS) processing. Identify ion identity networks containing a feature with a match to the GNPS spectral libraries using a minimum cosine similarity of 0.9 and a minimum of 4–6 shared fragment ions. Within each matched IIN, retain all ion identity features that have an MS2 spectrum and at least three signals above 0.1% relative intensity, then extract their consensus MS2 spectra as new library entries. Merge these into the propagated library (GNPS-IIMN-PROPOGATED) and consolidate with the manually curated GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE. The rationale is that MS1-based ion identity links (e.g., 0.001 m/z or 10 ppm tolerance, RT tolerance 0.1 min) connect spectra of different ion species of the same molecule, so a confident match to one species licenses spectra propagation to the rest.

## Related tools

- **GNPS** (Runs the spectral library search against GNPS libraries and hosts the propagated library (GNPS-IIMN-PROPOGATED) and FBMN/IIMN workflow) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MZmine** (Feature finding and IIMN processing; exports the feature quantification table (.csv), MS2 spectral summary (.mgf), and IIN edges files (.csv) consumed by the filter) — https://github.com/mzmine/mzmine
- **XCMS** (Alternative feature-finding tool for FBMN/IIMN input; xcms-gnps-tools provides formatSpectraForGNPS() for MGF export and adduct annotations for IIN) — https://github.com/jorainer/xcms-gnps-tools

## Evaluation signals

- Every retained library match has cosine similarity ≥ 0.9 and at least 4–6 shared fragment ions
- Each extracted library entry's spectrum has at least three signals above 0.1% relative intensity
- Extracted entries come only from ion identity networks containing at least one confident GNPS library match
- Ion-species coverage of the merged library spans the expected distribution (e.g., ~24% [M+H]+, 22% multimeric species, 17% [M+Na]+, 15% in-source fragments, 13% [M+NH4]+)
- Reproducing the 24 public datasets yields 538 new entries; total propagated IIMN library contains 2657 entries

## Limitations

- Variable feature detection and alignment parameters for the 24 datasets are only summarized in Supplementary Table 4, so exact reproduction from text alone is not possible
- Nodes with poor MS2 spectra (fewer than four signals) must be filtered out; unassigned charge states and isotope peaks are excluded from MS2 acquisition
- The IIN edge file format has no formal schema or validator tool mentioned

## Evidence

- [discussion] Library match thresholds: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [discussion] Spectrum quality filter: "all contained ion identity features with an MS2 spectrum and at least three signals above 0.1% relative intensity were extracted as new library spectra"
- [discussion] Library merge target: "merged into the existing manually created library GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE"
- [results] Propagation result: [evidence span withheld — anchor status: not_found]
- [discussion] Export step for inputs: [evidence span withheld — anchor status: not_found]
- [abstract] Public availability: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
