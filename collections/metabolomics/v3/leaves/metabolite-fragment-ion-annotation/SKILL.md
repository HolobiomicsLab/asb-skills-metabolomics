---
name: metabolite-fragment-ion-annotation
description: Use when a centroided LC/MS/MS feature table contains structurally related
  features (e.g. molecular subnetworks) and a known diagnostic product ion can define
  a compound class across those features.
when_to_use_negative:
- Raw data has not yet been feature-detected — run xcms-based feature detection and
  alignment first to obtain the 'mz'/'rt' feature table
- No diagnostic fragment ion or pattern is known for the compound class of interest
  — pattern matching requires predefined m/z patterns
- MS2 scans are not associated with feature-table entries (e.g. no DDA data or parent-ion
  tolerance mismatch)
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Platform hosting the Find Patterns (FindPatternsModule) and MS2 Browser modules
    used to match diagnostic fragment ions against MS2 spectra and record hits in
    the feature table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Underlying R package via Metaboseek's GUI for feature detection and alignment
    from LC/MS data to build the feature table
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: R package used to load xcms results into Metaboseek as a Feature Table with
    associated MS2 spectra
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: igraph
  role: R package used in Metaboseek for molecular subnetworking of features (SN2/SN3/SN4)
    prior to pattern annotation
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:42717246627c4b6323c6b10baa4be1c24e0428107502b1e0a047f1cc2ec6092c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - FindPatternsModule
  - MSnbase
  - Metaboseek
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# metabolite-fragment-ion-annotation

## Summary

Annotate molecular features in LC/MS/MS data by searching MS2 spectra for diagnostic fragment ions using Metaboseek's Find Patterns module — demonstrated by identifying >100 GLEA-like metabolites via the GLEA product ion at m/z 333.0592 plus phosphoglycerol- and glucose-derived fragments.

## When to use

Use when a centroided LC/MS/MS feature table contains structurally related features (e.g. molecular subnetworks) and a known diagnostic product ion can define a compound class across those features.

## When NOT to use

- Raw data has not yet been feature-detected — run xcms-based feature detection and alignment first to obtain the 'mz'/'rt' feature table
- No diagnostic fragment ion or pattern is known for the compound class of interest — pattern matching requires predefined m/z patterns
- MS2 scans are not associated with feature-table entries (e.g. no DDA data or parent-ion tolerance mismatch)

## Inputs

- Centroided LC/MS/MS raw data files (.mzML or other supported formats)
- Feature table as .csv with 'mz' and 'rt' columns
- Pattern definition: diagnostic product ion m/z values (e.g. 333.0592, 376.1016, 418.1123) plus associated fragment ions
- Noise-level threshold (% of maximum peak intensity in a scan)

## Outputs

- Annotated Feature Table (.mskFT) with new interactive columns recording pattern-match hits
- Exported .csv version of the annotated feature table
- Set of annotated GLEA-like molecular features (>100) across subnetworks SN2, SN3, and SN4

## How to apply

Load centroided LC/MS data (.mzML or other supported formats) and a feature table (csv with 'mz' and 'rt' columns) into Metaboseek, then associate MS2 scans with feature-table entries in the MS2 Browser using the default 5 ppm parent-ion m/z tolerance and 200 s retention time window (both customizable). Run the FindPatternsModule over the MS2 spectra of the features of interest — e.g. the SN2, SN3, and SN4 subnetwork features — searching for the GLEA-characteristic product ion at m/z 333.0592 together with the phosphoglycerol- and glucose-derived fragment ions, applying the noise-level (% of maximum peak intensity) peak filter to exclude low-intensity peaks. Annotate expected class-member product ions (for SN3, the beta-hydroxylation ions at m/z 376.1016 and m/z 418.1123) among the matched spectra to validate the annotation. All pattern matches are recorded as hits in new interactive columns of the Feature Table; save the annotated table as .mskFT (retaining processing history and metadata) and export a .csv for other software. Correct application yields a large, coherent set of class-member features — more than 100 GLEA-like features across SN2/SN3/SN4 in the demonstration.

## Related tools

- **Metaboseek** (Platform hosting the Find Patterns (FindPatternsModule) and MS2 Browser modules used to match diagnostic fragment ions against MS2 spectra and record hits in the feature table) — https://github.com/mjhelf/Metaboseek
- **xcms** (Underlying R package via Metaboseek's GUI for feature detection and alignment from LC/MS data to build the feature table) — https://github.com/sneumann/xcms
- **MSnbase** (R package used to load xcms results into Metaboseek as a Feature Table with associated MS2 spectra) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **igraph** (R package used in Metaboseek for molecular subnetworking of features (SN2/SN3/SN4) prior to pattern annotation) — https://github.com/igraph/rigraph

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- More than 100 GLEA-like molecular features are recorded across SN2, SN3, and SN4, with hits appearing in new interactive columns of the Feature Table
- Matched SN3 spectra additionally show the beta-hydroxylation product ions at m/z 376.1016 and m/z 418.1123
- Each hit spectrum contains the diagnostic GLEA product ion at m/z 333.0592 together with the phosphoglycerol- and glucose-derived fragments within the ppm tolerance
- Annotated feature table is saved as .mskFT retaining processing history and metadata, and a .csv export opens in other software without losing mz/rt columns
- Low-intensity peaks below the set 'Noise level in %' of maximum peak intensity were excluded from spectra before matching

## Limitations

- Spectra similarity is scored as 0 if fewer than 'min. peaks' match between two spectra, so very sparse MS2 spectra may fail pattern matching
- The 'ignore small fragments' option (excluding peaks below m/z 100) is an experimental feature
- Metabolite annotations via this workflow reach at most confidence Level 3 (tentative structure or compound-class assignments only)

## Evidence

- [results] The Find Patterns function enabled rapid identification of more than 100 GLEA-like molecular features distributed across SN2 and two additional subnetworks, SN3 and SN4: [evidence span withheld — anchor status: not_found]
- [methods] Noise level peak filter: "peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded"
- [methods] Feature detection with xcms GUI: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [readme] Metaboseek description: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
