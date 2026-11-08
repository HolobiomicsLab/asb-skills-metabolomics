---
name: ion-species-spectrum-aggregation
description: Use when building or extending MS/MS spectral libraries from feature-based
  molecular networking outputs, specifically to propagate a high-confidence library
  match (cosine >0.9) to other ion species (adducts, multimers, in-source fragments)
  of the same compound.
when_to_use_negative:
- When the goal is de novo spectral annotation of features lacking any library match
  — propagation only works within IINs that contain a feature matched to the GNPS
  spectral libraries at cosine >0.9
- When MS1 ion identity networks have not been built (no IIN edges .csv files) — IIMN
  feature relationships are required to link ion species of the same compound
- When working with raw MS/MS vendor data directly; the workflow expects feature-finding
  tool exports (feature quantification table, .mgf, IIN edges) from MZmine, MS-DIAL,
  or XCMS
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Feature-finding tool used to generate the aligned feature list, feature quantification
    table (.csv), MS2 spectral summary (.mgf), and IIN edges files that feed the aggregation
    workflow
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Hosts the IIMN workflow, provides the spectral library matches (cosine similarity),
    and stores the resulting GNPS-IIMN-PROPOGATED library
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative feature-finding tool interfaced with the IIMN workflow; xcms-gnps-tools
    provides formatSpectraForGNPS() and getFeatureAnnotations() to prepare IIN-compatible
    inputs
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:cd8793245e40a6cebb3705613f6c054263f7a46d688148c2b392e5c5288bfa20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ion-species-spectrum-aggregation

## Summary

Extracts consensus MS2 spectra of ion-identity features inside matched IINs to propagate high-confidence library annotations to connected ion species, yielding the GNPS-IIMN-PROPOGATED spectral library with 2657 entries (538 new entries from 24 public datasets).

## When to use

Use when building or extending MS/MS spectral libraries from feature-based molecular networking outputs, specifically to propagate a high-confidence library match (cosine >0.9) to other ion species (adducts, multimers, in-source fragments) of the same compound.

## When NOT to use

- When the goal is de novo spectral annotation of features lacking any library match — propagation only works within IINs that contain a feature matched to the GNPS spectral libraries at cosine >0.9
- When MS1 ion identity networks have not been built (no IIN edges .csv files) — IIMN feature relationships are required to link ion species of the same compound
- When working with raw MS/MS vendor data directly; the workflow expects feature-finding tool exports (feature quantification table, .mgf, IIN edges) from MZmine, MS-DIAL, or XCMS

## Inputs

- IIMN feature quantification table (.csv)
- MS2 spectral summary file (.mgf)
- Supplementary IIN edges files (.csv)
- GNPS spectral library match results (cosine scores per feature)
- Existing spectral libraries: GNPS-IIMN-PROPOGATED and GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE

## Outputs

- Consensus MS2 library spectra entries for connected ion species (538 new entries from 24 public datasets)
- Merged propagated IIMN spectral library with 2657 entries spanning adducts, multimers, and in-source fragments
- Broadened ion-species coverage report per library entry

## How to apply

Load the IIMN outputs of each dataset: the aligned feature quantification table (.csv), the MS2 spectral summary file (.mgf), and the supplementary IIN edges files (.csv). Identify ion identity networks (IINs) containing at least one feature with a GNPS spectral library match at minimum cosine similarity 0.9 and 4–6 shared fragment ions (or an authentic standard). Within each matched IIN, retain every ion identity feature that has an MS2 spectrum and at least three signals above 0.1% relative intensity, then extract its consensus MS2 spectrum as a new library entry — this captures the broadened ion-species distribution (24% [M+H]+, 22% multimeric species, 17% [M+Na]+, 15% in-source fragments, 13% [M+NH4]+). Merge the extracted entries into the GNPS-IIMN-PROPOGATED library and consolidate with the manually curated GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE. The rationale is that IIMN links ion species of the same molecule via MS1 characteristics, so a confident annotation on one ion species can be propagated to all connected species in the IIN.

## Related tools

- **MZmine** (Feature-finding tool used to generate the aligned feature list, feature quantification table (.csv), MS2 spectral summary (.mgf), and IIN edges files that feed the aggregation workflow) — https://github.com/mzmine/mzmine
- **GNPS** (Hosts the IIMN workflow, provides the spectral library matches (cosine similarity), and stores the resulting GNPS-IIMN-PROPOGATED library) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative feature-finding tool interfaced with the IIMN workflow; xcms-gnps-tools provides formatSpectraForGNPS() and getFeatureAnnotations() to prepare IIN-compatible inputs) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
Upload feature_quantification_table.csv, spectra.mgf and IIN edges .csv to the GNPS FEATURE-BASED-MOLECULAR-NETWORKING workflow (https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params={"workflow":"FEATURE-BASED-MOLECULAR-NETWORKING"}) with cosine 0.9 / 4–6 shared fragments to extract propagated library spectra.
```

## Evaluation signals

- Every extracted library spectrum originates from an IIN containing at least one feature matched to the GNPS spectral libraries with cosine ≥0.9 and 4–6 shared fragment ions
- Each retained ion identity feature has an MS2 spectrum with at least three signals above 0.1% relative intensity
- The merged GNPS-IIMN-PROPOGATED library contains the reported 538 new entries and totals 2657 entries
- Ion-species distribution of new entries spans adducts and in-source fragments (24% [M+H]+, 22% multimeric species, 17% [M+Na]+, 15% in-source fragments, 13% [M+NH4]+)
- Propagated entries consolidate cleanly with GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE without losing the manually curated content

## Limitations

- Variable feature detection and alignment parameters for the 24 datasets are only summarized in Supplementary Table 4 and are not reproducible from the article text alone
- Specific MZmine, MS-DIAL, or XCMS versions used for dataset processing are not stated
- Only ion species connected in an IIN containing a library-matched (or authentic-standard) feature get propagated; ion species in unannotated IINs are missed
- IIN edge format is a generic comma-separated file with ID1/ID2 columns and no formal schema or validator tool

## Evidence

- [discussion] library match threshold: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [discussion] retained feature filter: "all contained ion identity features with an MS2 spectrum and at least three signals above 0.1% relative intensity were extracted as new library spectra"
- [discussion] export outputs: [evidence span withheld — anchor status: not_found]
- [abstract] library availability: "IIMN-based spectral libraries with a broad coverage of ion species are publicly available."
- [readme] xcms helper functions: [evidence span withheld — anchor status: not_found]
