---
name: gnps-fbmn-job-submission
description: Use when an aligned MS1 feature quantification table with consensus MS2
  spectra and IIN edge files (.csv) is ready and an ion identity molecular network
  must be generated on the GNPS web server.
when_to_use_negative:
- Input is a raw instrument file without prior feature detection and alignment — IIMN
  requires an aligned feature list, not raw mzML alone.
- The dataset lacks MS1 ion identity grouping (no adduct/isotope/multimer annotation),
  so the IIN edge files cannot be produced.
- Analysis targets GC-MS or proteomics data where the feature-based molecular networking
  IIMN workflow is not applicable.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Web server hosting the FEATURE-BASED-MOLECULAR-NETWORKING workflow that runs
    IIMN on uploaded feature tables and edge files
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Exports the aligned feature list as feature quantification table (.csv), consensus
    MS2 .mgf, and IIN edge .csv files for GNPS upload
  repo: https://github.com/mzmine/mzmine
- name: xcms-gnps-tools
  role: R utility functions (formatSpectraForGNPS, getEdgelist, getFeatureAnnotations)
    that format xcms output into the file structure expected by GNPS FBMN/IIN
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:7dfc2f54f7a0c593b55958a5c35c3084050c23f0a66df39848b1706f75298ed4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MZmine
  - xcms-gnps-tools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gnps-fbmn-job-submission

## Summary

Submit an aligned feature list, consensus MS2 spectra (.mgf), and ion identity network edge files to the GNPS feature-based molecular networking (FBMN) environment with IIMN enabled, producing ion identity molecular networks with two layers of node connectivity.

## When to use

Use when an aligned MS1 feature quantification table with consensus MS2 spectra and IIN edge files (.csv) is ready and an ion identity molecular network must be generated on the GNPS web server.

## When NOT to use

- Input is a raw instrument file without prior feature detection and alignment — IIMN requires an aligned feature list, not raw mzML alone.
- The dataset lacks MS1 ion identity grouping (no adduct/isotope/multimer annotation), so the IIN edge files cannot be produced.
- Analysis targets GC-MS or proteomics data where the feature-based molecular networking IIMN workflow is not applicable.

## Inputs

- Aligned MS1 feature quantification table (.csv) from MZmine, MS-DIAL, or XCMS
- MS2 spectral summary file (.mgf) with consensus MS2 spectra per feature
- Supplementary IIN edge files (.csv) with columns ID1 and ID2
- MZML/open-format raw-derived spectra optionally preprocessed with a special MZmine IIMN build

## Outputs

- Ion identity molecular network (IIMN) with two layers of feature connectivity
- Per-feature spectral library matches (cosine ≥ 0.9, 4–6 shared fragments) propagated over IIN neighbors
- Expanded GNPS-IIMN-PROPOGATED spectral library entries

## How to apply

Export the aligned feature list from MZmine as three files: a feature quantification table (.csv), an MS2 spectral summary file (.mgf), and supplementary IIN edge files (.csv). Upload these to the GNPS FBMN workflow (FEATURE-BASED-MOLECULAR-NETWORKING) with IIMN enabled, using the IIN edge format with columns ID1 and ID2 that link features sharing MS1 ion identities (adducts, isotopes, multimers within 0.001 m/z or 10 ppm). Enable the IIMN connectivity layers so that nodes are linked both by MS1 ion-identity relationships and by MS2 spectral similarity. Rationale: ion species of the same molecule often remain unconnected in MS/MS-based molecular networks, and IIMN collapses them by fusing MS1-based ion identity networks with MS2-based spectral similarity, which propagates spectral library matches to neighboring nodes and increases annotation rates.

## Related tools

- **GNPS** (Web server hosting the FEATURE-BASED-MOLECULAR-NETWORKING workflow that runs IIMN on uploaded feature tables and edge files) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MZmine** (Exports the aligned feature list as feature quantification table (.csv), consensus MS2 .mgf, and IIN edge .csv files for GNPS upload) — https://github.com/mzmine/mzmine
- **xcms-gnps-tools** (R utility functions (formatSpectraForGNPS, getEdgelist, getFeatureAnnotations) that format xcms output into the file structure expected by GNPS FBMN/IIN) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
MZmine: Tools > Export > Export for GNPS FBMN (feature table .csv + MS2 .mgf + IIN edges .csv), then upload to https://gnps.ucsd.edu/ProteoSAFe/index.jsp?params=%7B%22workflow%22:%22FEATURE-BASED-MOLECULAR-NETWORKING%22
```

## Evaluation signals

- The uploaded feature quantification table, .mgf, and IIN edge .csv files must share a consistent feature ID space so ID1/ID2 edges resolve to actual nodes.
- The resulting IIMN must contain two distinct connectivity layers: MS1 ion-identity edges (within 0.001 m/z or 10 ppm) and MS2 spectral-similarity edges.
- Annotation statistics must show a mean ~35% gain in library match propagation over IIN neighbors (up to 325% on dataset 4).
- MS2 spectral library matching should land near ~6% and MS1 ion annotation near ~12% of features per dataset.
- Generated network should collapse redundant ion species of the same compound into connected groups rather than disconnected sub-networks.

## Limitations

- The IIN edge text format is a generic comma-separated style with only ID1 and ID2 columns; no formal schema or validator tool is mentioned.
- The special MZmine IIMN build's release status (part of a released version vs. custom build) is unclear.
- Variable feature detection and alignment parameters across the 24 datasets (Supplementary Table 4) are not reproducible from text alone.
- Nodes with poor MS2 spectra (fewer than four signals) should be filtered out before submission.

## Evidence

- [discussion] Export the feature list as a feature quantification table (.csv), an MS2 spectral summary file (.mgf), and a supplementary edges files (IIN files, .csv): [evidence span withheld — anchor status: not_found]
- [discussion] The text format follows a generic comma-separated style with the columns ID1 and ID2: "The text format follows a generic comma-separated style with the columns ID1 and ID2"
- [intro] A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow on the web server: "A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow on the web server"
- [intro] The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
- [readme] formatSpectraForGNPS(): "formatSpectraForGNPS(): format spectra for MGF export in the format expected
  by GNPS."
