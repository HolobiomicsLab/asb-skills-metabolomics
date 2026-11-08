---
name: lipid-annotation-verification
description: Use when LC/MS feature tables contain enriched lipid features with tentative
  acyl-chain annotations whose biosynthetic origin is unknown and isotope-labeled
  precursor (e.g., 13C6-leucine) versus unlabeled control data are available to test
  incorporation.
when_to_use_negative:
- The candidate lipids already have confident Level 1/2 annotations with authentic
  standards, so isotope tracing adds no annotation evidence
- Only unlabeled samples exist — no labeled-supplement feature table is available
  for pairwise Label Finder comparison
- The analysis goal is raw feature detection from profiles rather than testing annotations
  of already-detected features (use the xcms module first)
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Hosts the LabelFinderModule used in pairwise mode to detect isotope-labeled
    features and to export the isotope-enrichment table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Feature detection and alignment of the 13C6-Leu-supplemented and unlabeled
    LC/MS samples into feature tables prior to label tracing
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used to load xcms results into Metaboseek as a Feature Table for subsequent
    isotope-label analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:e58d7e2b8f8ac4f365a09438f8ae50479b64a19cc4729522ffd00fde064f6418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - Metaboseek (LabelFinderModule)
  - xcms
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lipid-annotation-verification

## Summary

Verification of putative lipid annotations by isotope tracing in Metaboseek's Label Finder, testing whether candidate lipids (e.g., odd-chain 13:1 and 15:1 monounsaturated species) derive from a proposed biosynthetic route such as branched-chain fatty acid (BCFA) metabolism via 13C6-leucine incorporation.

## When to use

Use when LC/MS feature tables contain enriched lipid features with tentative acyl-chain annotations whose biosynthetic origin is unknown and isotope-labeled precursor (e.g., 13C6-leucine) versus unlabeled control data are available to test incorporation.

## When NOT to use

- The candidate lipids already have confident Level 1/2 annotations with authentic standards, so isotope tracing adds no annotation evidence
- Only unlabeled samples exist — no labeled-supplement feature table is available for pairwise Label Finder comparison
- The analysis goal is raw feature detection from profiles rather than testing annotations of already-detected features (use the xcms module first)

## Inputs

- Centroided LC/MS raw data files (.mzML/.mzXML)
- Preprocessed molecular feature tables with required 'mz' and 'rt' columns for 13C6-leucine-supplemented samples
- Matching unlabeled leucine-supplemented control feature tables
- List of candidate lipid features of interest (e.g., 13:1 and 15:1 monounsaturated lipids enriched in hacl-1 worms)

## Outputs

- Isotope-enrichment table (.csv) exported via Metaboseek Save Table, listing features with detected label shifts
- Set of verified features carrying the Δ m/z 5.0167 (13C5) isotopologue shift
- Verdict (labeled / not labeled) for each candidate lipid annotation, e.g. no 13C5 incorporation for the hacl-1-enriched 13:1 and 15:1 lipids

## How to apply

Load centroided LC/MS data (.mzML/.mzXML) and preprocessed feature tables (with 'mz' and 'rt' columns) from the 13C6-leucine-supplemented and unlabeled leucine-supplemented samples into Metaboseek. Launch the LabelFinderModule in pairwise mode, comparing the labeled table against the unlabeled control, and filter the output for features carrying the expected isotopologue mass shift (Δ m/z 5.0167 for 13C5 incorporation into leucine-derived BCFA). Inspect the specific candidate lipid rows — here the hacl-1-enriched 13:1 and 15:1 monounsaturated lipids — and record whether the label shift is detected or absent; absence of the shift refutes the BCFA-origin hypothesis. Export the enrichment results via Save Table as a .csv isotope-enrichment table. The rationale is that targeted isotope tracing converts a tentative annotation into a biosynthetic test, since leucine feeds the iso/anteiso branched-chain fatty acid pool in C. elegans.

## Related tools

- **Metaboseek** (Hosts the LabelFinderModule used in pairwise mode to detect isotope-labeled features and to export the isotope-enrichment table) — https://github.com/mjhelf/Metaboseek
- **xcms** (Feature detection and alignment of the 13C6-Leu-supplemented and unlabeled LC/MS samples into feature tables prior to label tracing) — https://github.com/sneumann/xcms
- **MSnbase** (Used to load xcms results into Metaboseek as a Feature Table for subsequent isotope-label analysis)

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); Metaboseek::runMetaboseek()
```

## Evaluation signals

- The exported enrichment table lists several hundred features carrying the Δ m/z 5.0167 (13C5) shift, matching the expected leucine-derived isotopologue signature
- The 13:1 and 15:1 monounsaturated lipid rows show no detected label shift, consistent with the reported negative result for BCFA origin
- Feature tables loaded into Metaboseek contain the required 'mz' and 'rt' columns before running Label Finder
- Positive-control features known to derive from leucine (e.g., iso-branched fatty acids among the 13C5-enriched features) are recovered in the filtered output
- The output is a .csv exported with the Save Table function and contains enrichment/shift annotations for each feature

## Limitations

- Metabolite annotations in this workflow reach at most confidence Level 3 (tentative structure or compound class assignments only), so a negative tracing result refutes one hypothesis but does not fully identify the lipid
- Many detected features remain unidentified, so absence of a label shift leaves the biosynthetic origin unresolved
- The app could not be deployed on shinyapps.io due to the mzR dependency; it must be installed locally, via Docker (mjhelf/metaboseek), or via the hosted shiny-server instance

## Evidence

- [intro] SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds: "SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds"
- [methods] Metaboseek graphical user interface for xcms feature detection: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [readme] README on loading xcms results as feature tables: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] README on install from GitHub: "devtools::install_github("mjhelf/Metaboseek")"
