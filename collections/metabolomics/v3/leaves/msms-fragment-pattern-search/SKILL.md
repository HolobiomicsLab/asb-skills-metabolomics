---
name: msms-fragment-pattern-search
description: Use when you have centroided LC/MS data (mzML) with a feature table and
  MS2 scans, and a known diagnostic product ion (e.g., m/z 333.0592) to screen for
  structurally related features across many spectra.
when_to_use_negative:
- You need confident (Level 1/2) compound identifications — this pattern search supports
  only tentative annotations; the article notes Metaboseek facilitates metabolite
  annotation only up to confidence Level 3.
- You only have MS1 data with no associated MS2 scans — the MS2 Browser linkage and
  pattern search require fragmentation spectra.
- Your goal is de novo formula prediction or spectral library matching rather than
  screening for a known diagnostic fragment motif.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metaboseek
  role: Host platform providing the Load module, MS2 Browser, and Find Patterns (FindPatternsModule)
    GUI for fragment-pattern search over MS2 spectra
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Underlying feature detection and alignment producing the feature table consumed
    by the pattern search
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used to load xcms results into the Metaboseek app as a Feature Table with
    associated MS2 spectra
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:0868ead54c414afc2e58eca09d3a3cf3780e15d3d2e7b461c8c40a27e5603023
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - FindPatternsModule
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# msms-fragment-pattern-search

## Summary

Search MS2 spectra of LC/MS molecular features in Metaboseek's Find Patterns module (FindPatternsModule) for a diagnostic fragment-ion pattern, exemplified by the GLEA-characteristic product ion at m/z 333.0592 plus phosphoglycerol- and glucose-derived fragments, to rapidly flag structurally related metabolites across molecular subnetworks.

## When to use

Use when you have centroided LC/MS data (mzML) with a feature table and MS2 scans, and a known diagnostic product ion (e.g., m/z 333.0592) to screen for structurally related features across many spectra.

## When NOT to use

- You need confident (Level 1/2) compound identifications — this pattern search supports only tentative annotations; the article notes Metaboseek facilitates metabolite annotation only up to confidence Level 3.
- You only have MS1 data with no associated MS2 scans — the MS2 Browser linkage and pattern search require fragmentation spectra.
- Your goal is de novo formula prediction or spectral library matching rather than screening for a known diagnostic fragment motif.

## Inputs

- Centroided LC/MS data files (.mzML or other supported formats)
- Feature table (csv with 'mz' and 'rt' columns)
- Raw MS2 scans associated to features
- Pattern specification: diagnostic product ion m/z values (e.g., 333.0592, 376.1016, 418.1123)

## Outputs

- Feature table with new interactive columns recording pattern-match hits
- Annotated Feature Table saved as .mskFT (with processing history and metadata)
- Exported .csv of the annotated feature table
- Set of identified GLEA-like molecular features (e.g., >100 features across SN2/SN3/SN4)

## How to apply

Load centroided LC/MS files (.mzML) and a feature table (csv with 'mz' and 'rt' columns) into Metaboseek via the Load module, then associate MS2 scans with feature-table entries in the MS2 Browser using the default 5 ppm parent-ion m/z tolerance and 200 s retention time window (both customizable). Run the Find Patterns module (FindPatternsModule) over the MS2 spectra, searching for the diagnostic product ion together with co-occurring fragment ions (for GLEA: m/z 333.0592 plus phosphoglycerol- and glucose-derived fragments, plus beta-hydroxylation ions at m/z 376.1016 and 418.1123 for SN3 variants), applying the noise-level filter that excludes peaks below a set percentage of the maximum peak intensity in a scan. Pattern matches are recorded as hits in new interactive columns of the Feature Table; in the GLEA case this identified more than 100 GLEA-like molecular features across the SN2, SN3, and SN4 subnetworks. Save the annotated table as .mskFT (preserving processing history and metadata) and export a .csv for downstream software.

## Related tools

- **Metaboseek** (Host platform providing the Load module, MS2 Browser, and Find Patterns (FindPatternsModule) GUI for fragment-pattern search over MS2 spectra) — https://github.com/mjhelf/Metaboseek
- **xcms** (Underlying feature detection and alignment producing the feature table consumed by the pattern search) — https://github.com/sneumann/xcms
- **MSnbase** (Used to load xcms results into the Metaboseek app as a Feature Table with associated MS2 spectra) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); library(Metaboseek); mskApp()  # then Load module -> mzML + csv feature table -> MS2 Browser (5 ppm, 200 s) -> Find Patterns: m/z 333.0592 + PG/glucose fragments
```

## Evaluation signals

- More than 100 GLEA-like molecular features are recovered, distributed across the SN2, SN3, and SN4 subnetworks, with hits in new feature-table columns
- Matched spectra contain the GLEA product ion at m/z 333.0592 plus phosphoglycerol- and glucose-derived fragments within the applied peak-filter settings
- SN3 beta-hydroxylation product ions at m/z 376.1016 and m/z 418.1123 are correctly annotated among the matched spectra
- Saved .mskFT file retains processing history and metadata, and the exported .csv opens with the same annotation columns in other software
- MS2-to-feature associations hold at the MS2 Browser defaults (5 ppm parent m/z tolerance, 200 s RT window)

## Limitations

- Annotations remain at confidence Level 3 (tentative structure or compound-class assignments only)
- The 'ignore small fragments' option is an experimental feature excluding peaks below m/z 100
- Spectral-similarity scoring treats matches as 0 if fewer than 'min. peaks' peaks match, so sparse MS2 spectra can miss pattern hits
- Many detected features remain unidentified; diagnostic-ion pattern search does not resolve full structures

## Evidence

- [methods] MS2 Browser default tolerances: "peaks at an intensity below a set percentage of the maximum peak 
intensity in a scan (`Noise level in %`) will be excluded"
- [methods] Feature table loading via MSnbase: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [results] Find Patterns finding >100 GLEA-like features: [evidence span withheld — anchor status: not_found]
- [discussion] Level 3 annotation confidence: "facilitates metabolite annotation up to confidence Level 3 for many detected features, enabling tentative structure or compound class assignments"
- [readme] Metaboseek is an interactive MS data analysis tool built on xcms: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data.
It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package"
