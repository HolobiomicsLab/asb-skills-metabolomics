---
name: iimn-annotation-propagation
description: Use when processing feature quantification tables (.csv), MS2 spectral
  summaries (.mgf), and IIN edges files from ion identity molecular networking, and
  the goal is to expand spectral library coverage by extracting consensus MS2 spectra
  of unannotated ion species linked to annotated features.
when_to_use_negative:
- The feature table lacks IIN edges (e.g., plain FBMN output without ion identity
  networking) — there is no connectivity to propagate annotations along.
- No feature in the network meets the library-match threshold (cosine >0.9, 4–6 shared
  fragments) or corresponds to an authentic standard — propagation would be ungrounded.
- Features have poor MS2 spectra with fewer than four signals or fewer than three
  signals above 0.1% relative intensity — they are filtered out rather than used as
  library entries.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Feature finding, MS1-based ion identity annotation, and export of the feature
    quantification table (.csv), MS2 spectral summary (.mgf), and IIN edges files
    used for propagation
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Runs the IIMN workflow and spectral library search on the web server; hosts
    the propagated library GNPS-IIMN-PROPOGATED
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative open-source tool for producing aligned feature lists that can
    be formatted for GNPS FBMN/IIN via xcms-gnps-tools
  repo: https://github.com/jorainer/xcms-gnps-tools
- name: MS-DIAL
  role: Alternative feature-finding tool interfaced with the IIMN workflow for generating
    the input feature tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:4db268b39735213a6595b03d1afadea41a4fa4161783b5abf0993ba9acfdf641
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MS-DIAL
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# iimn-annotation-propagation

## Summary

Propagates high-confidence GNPS spectral library annotations (cosine >0.9 with 4–6 shared fragments, or authentic standards) across connected ion identity features within IIMN networks, yielding propagated consensus MS2 library spectra entries (e.g., GNPS-IIMN-PROPOGATED with 2657 entries covering adducts, multimers, and in-source fragments).

## When to use

Use when processing feature quantification tables (.csv), MS2 spectral summaries (.mgf), and IIN edges files from ion identity molecular networking, and the goal is to expand spectral library coverage by extracting consensus MS2 spectra of unannotated ion species linked to annotated features.

## When NOT to use

- The feature table lacks IIN edges (e.g., plain FBMN output without ion identity networking) — there is no connectivity to propagate annotations along.
- No feature in the network meets the library-match threshold (cosine >0.9, 4–6 shared fragments) or corresponds to an authentic standard — propagation would be ungrounded.
- Features have poor MS2 spectra with fewer than four signals or fewer than three signals above 0.1% relative intensity — they are filtered out rather than used as library entries.

## Inputs

- Feature quantification table (.csv) from MZmine/MS-DIAL/XCMS aligned feature lists
- MS2 spectral summary file (.mgf) with consensus MS2 spectra per feature
- Supplementary IIN edges files (.csv, columns ID1 and ID2) from the ion identity networking step
- GNPS spectral library search results (annotation per feature)

## Outputs

- Consensus MS2 spectra of retained ion-identity features as new spectral library entries
- Propagated IIMN spectral library (GNPS-IIMN-PROPOGATED) with broadened ion species coverage
- Merged library consolidated with GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE

## How to apply

Load the IIMN outputs of the datasets of interest: the feature quantification table (.csv), the MS2 spectral summary file (.mgf), and the supplementary IIN edges files (.csv). Identify ion identity networks (IINs) containing at least one feature with a GNPS spectral library match using a minimum cosine similarity of 0.9 and a minimum of 4–6 shared fragment ions (or an authentic standard match). Within each matched IIN, retain all ion identity features that have an MS2 spectrum and at least three signals above 0.1% relative intensity, then extract their consensus MS2 spectra as new library entries. This works because IIMN links ion species of the same compound via MS1 characteristics (m/z tolerance 0.001 or 10 ppm, RT tolerance 0.1 min, feature shape Pearson correlation ≥0.85), so a single annotated feature licenses propagation of its identity to connected [M+H]+, [M+Na]+, [M+NH4]+, multimeric, and in-source fragment species. Merge the extracted entries into the propagated IIMN spectral library (GNPS-IIMN-PROPOGATED) and consolidate with the manually curated GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE.

## Related tools

- **MZmine** (Feature finding, MS1-based ion identity annotation, and export of the feature quantification table (.csv), MS2 spectral summary (.mgf), and IIN edges files used for propagation) — https://github.com/mzmine/mzmine
- **GNPS** (Runs the IIMN workflow and spectral library search on the web server; hosts the propagated library GNPS-IIMN-PROPOGATED) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative open-source tool for producing aligned feature lists that can be formatted for GNPS FBMN/IIN via xcms-gnps-tools) — https://github.com/jorainer/xcms-gnps-tools
- **MS-DIAL** (Alternative feature-finding tool interfaced with the IIMN workflow for generating the input feature tables)

## Evaluation signals

- Every propagated entry traces to an IIN containing a feature with a GNPS library match at cosine ≥0.9 and ≥4–6 shared fragment ions.
- Each extracted library spectrum has at least three signals above 0.1% relative intensity.
- The propagated library reproduces the reported ion species distribution (e.g., ~24% [M+H]+, 22% multimeric species, 17% [M+Na]+, 15% in-source fragments, 13% [M+NH4]+).
- Reproducing the 24 public datasets plus two NIH 'ACONN' natural products datasets yields the reported 538 new entries; the full IIMN library totals 2657 entries.
- Merged output is consistent with the existing GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE entries without duplicates.

## Limitations

- Specific MZmine, MS-DIAL, or XCMS versions used for the 24 datasets are not stated; variable feature detection and alignment parameters are only summarized in Supplementary Table 4 and are not reproducible from the text alone.
- The special MZmine IIMN build's release status is unclear (whether the two IIMN modules in the Tools tab are part of a released MZmine version).
- Nodes with poor MS2 spectra (fewer than four signals) must be filtered out before propagation; unassigned charge states and isotope peaks were excluded from MS2 acquisition, so some ion species lack spectra to extract.

## Evidence

- [discussion] Library matches propagate across ion species: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [discussion] Extraction threshold for new library spectra: "all contained ion identity features with an MS2 spectrum and at least three signals above 0.1% relative intensity were extracted as new library spectra"
- [discussion] Library merge into existing curated library: "merged into the existing manually created library GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE"
- [abstract] IIMN purpose: connect and collapse ion species: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [discussion] Input file formats for the IIMN workflow: [evidence span withheld — anchor status: not_found]
