---
name: ion-identity-network-construction
description: Use when you have an aligned LC-MS/MS feature list with consensus MS2
  spectra and redundant unconnected nodes (adducts/isotopes of one compound), and
  you want MS1 ion-identity edges to propagate annotations in GNPS FBMN.
when_to_use_negative:
- When you already hold a merged/collapsed feature table per compound with no adduct/isotope
  redundancy — IIN grouping would be redundant
- When the data lack MS1 peak-shape information across samples, since grouping depends
  on RT tolerance (0.1 min), Pearson shape correlation (>=0.85, >=5 points), and overlapping
  intensity (>=50%, >=2 samples)
- When the analysis is purely MS2 spectral-similarity networking (e.g., classical
  GNPS METABOLOMICS-SNETS-V2) without an aligned feature table to upload
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3361
tools:
- name: MZmine
  role: Performs metaCorrelate MS1-based feature grouping, ion identity annotation
    (adducts, isotopes, multimers), and export of the FBMN inputs (quantification
    table, .mgf, IIN edge files); used to process all 24 public datasets
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Web environment where the exported files are submitted to Feature-Based Molecular
    Networking with IIMN enabled, fusing the MS1 IIN layer with the MS2 spectral-similarity
    layer
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: xcms-gnps-tools
  role: Provides formatSpectraForGNPS(), getEdgelist(), and getFeatureAnnotations()
    to format xcms/CAMERA results into the IIN edge and MGF formats expected by FBMN
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:69a502a281033bc83f52a2bcbbf5e7d1c7837729694e5625ae9117ca97b8851b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - xcms-gnps-tools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# ion-identity-network-construction

## Summary

Builds MS1-based ion identity networks (IIN) that group and connect adducts, isotopes, and multimers of the same compound across an aligned LC-MS/MS feature list, so that MS2 spectral library matches can be propagated over IIN edges in GNPS Feature-Based Molecular Networking (FBMN) — raising annotation rates by an average of 35% across 24 public datasets.

## When to use

Use when you have an aligned LC-MS/MS feature list with consensus MS2 spectra and redundant unconnected nodes (adducts/isotopes of one compound), and you want MS1 ion-identity edges to propagate annotations in GNPS FBMN.

## When NOT to use

- When you already hold a merged/collapsed feature table per compound with no adduct/isotope redundancy — IIN grouping would be redundant
- When the data lack MS1 peak-shape information across samples, since grouping depends on RT tolerance (0.1 min), Pearson shape correlation (>=0.85, >=5 points), and overlapping intensity (>=50%, >=2 samples)
- When the analysis is purely MS2 spectral-similarity networking (e.g., classical GNPS METABOLOMICS-SNETS-V2) without an aligned feature table to upload

## Inputs

- Aligned LC-MS/MS feature list with consensus MS2 spectra from MZmine, MS-DIAL, or XCMS
- Raw MS data converted to an open format (e.g., mzML) if needed
- Feature shape/intensity chromatograms across samples for correlation grouping

## Outputs

- Feature quantification table (.csv) for GNPS FBMN upload
- MS2 spectral summary file (.mgf)
- Supplementary IIN edge files (.csv) with generic comma-separated columns ID1 and ID2
- Ion identity molecular network combining MS1 ion-identity and MS2 spectral-similarity layers
- Per-dataset annotation statistics (fraction of features with MS2 library matches, MS1 ion annotations, and propagation gain)

## How to apply

In MZmine, run the metaCorrelate MS1-based feature grouping with a retention time tolerance of 0.1 min, feature shape Pearson correlation >= 0.85 with at least five data points (two on each edge), feature height Pearson correlation >= 0.6 with at least three data points, and a minimum overlapping intensity percentage of 50% in at least two samples. Then build the ion identity network with a maximum m/z tolerance of 0.001 or 10 ppm to annotate and connect ion species (adducts, isotopes, multimers) of the same compound. Export the aligned feature list as a feature quantification table (.csv), an MS2 spectral summary (.mgf), and the supplementary IIN edge files (.csv), and submit these to the GNPS FBMN environment with IIMN enabled. The rationale is that the resulting ion identity molecular networks contain two connectivity layers — MS1 ion-identity links between ion species of one compound and MS2 spectral-similarity links between structurally related compounds — enabling library matches (cosine >= 0.9, 4-6 shared fragments) to be propagated to first-neighboring IIN nodes. For xcms-based processing, the jorainer/xcms-gnps-tools functions formatSpectraForGNPS() and getEdgelist()/getFeatureAnnotations() produce the same GNPS-expected file formats.

## Related tools

- **MZmine** (Performs metaCorrelate MS1-based feature grouping, ion identity annotation (adducts, isotopes, multimers), and export of the FBMN inputs (quantification table, .mgf, IIN edge files); used to process all 24 public datasets) — https://github.com/mzmine/mzmine
- **GNPS** (Web environment where the exported files are submitted to Feature-Based Molecular Networking with IIMN enabled, fusing the MS1 IIN layer with the MS2 spectral-similarity layer) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **xcms-gnps-tools** (Provides formatSpectraForGNPS(), getEdgelist(), and getFeatureAnnotations() to format xcms/CAMERA results into the IIN edge and MGF formats expected by FBMN) — https://github.com/jorainer/xcms-gnps-tools

## Evaluation signals

- Annotation statistics reproduce the headline values: ~6% MS2 spectral library matching, ~12% MS1 ion annotation, mean ~35% gain from propagating library matches over first-neighboring IIN nodes, and a maximum 325% gain on dataset 4
- Exported IIN edge file follows the generic comma-separated format with ID1 and ID2 columns as required for GNPS upload
- Grouping parameters match the published thresholds: RT tolerance 0.1 min, feature shape Pearson correlation >= 0.85 (>=5 data points), feature height Pearson correlation >= 0.6 (>=3 data points), overlapping intensity >= 50% in >=2 samples, and m/z tolerance 0.001 or 10 ppm
- MS2 library matches meet cosine >= 0.9 with 4-6 shared fragment ions
- Resulting network shows increased connectivity — redundant sub-networks of the same compound are collapsed into single ion-identity clusters

## Limitations

- All 24 datasets were processed with variable feature detection and alignment parameters summarized only in Supplementary Table 4, not reproducible from the article text alone
- The MZmine IIMN module is described as a 'special IIMN build' — it is unclear whether it is part of a released MZmine version or a custom build, and no specific MZmine/MS-DIAL/XCMS versions are pinned
- Ions with unassigned charge states and isotope peaks were excluded from MS2 acquisition, so some ion species cannot be connected or annotated via MS2

## Evidence

- [abstract] Peak-shape correlation rationale: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [intro] Two connectivity layers: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity"
- [discussion] Export formats for GNPS: [evidence span withheld — anchor status: not_found]
- [discussion] Grouping thresholds: "a feature shape Pearson correlation greater equals 0.85 with at least five data points and two data points on each edge"
- [discussion] RT tolerance for grouping: "A pair of features were grouped with a retention time tolerance of 0.1 min"
- [readme] xcms tooling for IIN: [evidence span withheld — anchor status: external_doc]
- [readme] GNPS workflow repository: [evidence span withheld — anchor status: external_doc]
