---
name: isotope-label-tracing-analysis
description: Use when you have centroided LC/MS data (mzML/mzXML) and preprocessed
  feature tables ('mz' and 'rt' columns) from isotope-tracer-supplemented versus unlabeled
  samples and need to find features with tracer-derived mass shifts.
when_to_use_negative:
- Input data lacks an unlabeled control condition to pair against the tracer-supplemented
  table — Label Finder requires a pairwise labeled/unlabeled comparison
- Feature tables are missing the required 'mz' and 'rt' columns, so isotope shifts
  cannot be computed
- The question is general comparative metabolomics between genotypes/conditions rather
  than tracer-derived isotope enrichment
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Metaboseek
  role: Platform hosting the Label Finder module; loads feature tables, runs pairwise
    isotope-shift detection, and exports the enrichment table as .csv via Save Table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Used through Metaboseek's GUI to detect and align molecular features from
    LC/MS data into feature tables consumed by Label Finder
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used with xcms to load xcms results into Metaboseek as a 'Feature Table' (mzR-based)
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:f0e1353def0775530521fc4a2021c690e9bea827bc7310c8e7dfa4e54a0c53a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - Metaboseek (LabelFinderModule)
  - xcms
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# isotope-label-tracing-analysis

## Summary

Detect isotope-labeled metabolites in LC/MS feature tables using Metaboseek's Label Finder module in pairwise mode, comparing a labeled tracer (e.g. 13C6-leucine) against an unlabeled control to identify label incorporation and trace biosynthetic origins of lipids.

## When to use

Use when you have centroided LC/MS data (mzML/mzXML) and preprocessed feature tables ('mz' and 'rt' columns) from isotope-tracer-supplemented versus unlabeled samples and need to find features with tracer-derived mass shifts.

## When NOT to use

- Input data lacks an unlabeled control condition to pair against the tracer-supplemented table — Label Finder requires a pairwise labeled/unlabeled comparison
- Feature tables are missing the required 'mz' and 'rt' columns, so isotope shifts cannot be computed
- The question is general comparative metabolomics between genotypes/conditions rather than tracer-derived isotope enrichment

## Inputs

- Centroided LC/MS raw data files (.mzML/.mzXML) from isotope-tracer-supplemented samples
- Centroided LC/MS raw data files (.mzML/.mzXML) from unlabeled control samples
- Preprocessed feature tables with required 'mz' and 'rt' columns for both conditions (e.g. from xcms via Metaboseek)

## Outputs

- Isotope-enrichment table (.csv) listing features with tracer-derived mass shifts (e.g. Δ m/z 5.0167 for 13C5)
- Per-feature label incorporation status distinguishing labeled from unlabeled compounds

## How to apply

Load the centroided LC/MS files and preprocessed feature tables (with required 'mz' and 'rt' columns) for the labeled and unlabeled conditions into Metaboseek, then launch the LabelFinderModule in pairwise mode, comparing the labeled table against the unlabeled control. In the 13C6-leucine example, filter the output for features whose mass shift matches the 13C5 isotopologue signature (Δ m/z 5.0167), which reproduces several hundred enriched features including iso-branched fatty acids. The rationale is that each 13C atom incorporated from the tracer shifts the precursor m/z by 1.00335, so a specific Δ m/z signature reveals how many labeled carbons a metabolite derives from the tracer. Inspect individual enriched entries (e.g. hacl-1-enriched 13:1 and 15:1 monounsaturated lipids) to verify negative results — absence of the label shift means the compound does not originate from the traced pathway (here, branched-chain fatty acid metabolism). Export enriched features with Save Table as a .csv isotope-enrichment table for downstream validation.

## Related tools

- **Metaboseek** (Platform hosting the Label Finder module; loads feature tables, runs pairwise isotope-shift detection, and exports the enrichment table as .csv via Save Table) — https://github.com/mjhelf/Metaboseek
- **xcms** (Used through Metaboseek's GUI to detect and align molecular features from LC/MS data into feature tables consumed by Label Finder) — https://github.com/sneumann/xcms
- **MSnbase** (Used with xcms to load xcms results into Metaboseek as a 'Feature Table' (mzR-based)) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); Metaboseek::runMetaboseek()  # then load feature tables and run LabelFinderModule pairwise (13C6-Leu vs unlabeled)
```

## Evaluation signals

- Enrichment table contains several hundred features carrying the Δ m/z 5.0167 (13C5) shift for the 13C6-leucine tracing experiment
- Known 13C5-enriched features such as iso-branched fatty acids are recovered in the Label Finder output
- The hacl-1-enriched 13:1 and 15:1 monounsaturated lipid rows show no detected label shift (expected negative result)
- All input feature tables contain the required 'mz' and 'rt' columns before running Label Finder
- Exported .csv from Save Table is loadable and retains feature m/z, retention time, and enrichment annotations

## Limitations

- Label Finder detects isotope shifts against a paired unlabeled control; without the unlabeled comparison the module cannot be applied
- The negative result (no label incorporation) only indicates the compound does not derive from the traced pathway via the tracer — in the hacl-1 study, 13:1/15:1 lipids did not originate from BCFA metabolism
- Metaboseek annotation generally reaches confidence Level 3 (tentative structure or compound class assignments) for many features; many detected features remain unidentified

## Evidence

- [intro] SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds: "SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds"
- [readme] Load xcms results as Feature Table using xcms and MSnbase: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [full_text] 13C6-Leucine tracing via Label Finder showed no label incorporation in 13:1/15:1 lipids: [evidence span withheld — anchor status: not_found]
- [methods] Metaboseek offers a graphical user interface to set up data analysis with the `xcms` package: "Metaboseek offers a graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [discussion] Metaboseek enables annotation up to confidence Level 3: "facilitates metabolite annotation up to confidence Level 3 for many detected features, enabling tentative structure or compound class assignments"
