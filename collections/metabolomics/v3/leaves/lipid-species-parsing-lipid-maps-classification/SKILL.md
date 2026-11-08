---
name: lipid-species-parsing-lipid-maps-classification
description: Use when a lipid abundance matrix lacks structured annotations (class,
  chain composition) and you need to decompose lipid species names before normalization,
  statistical analysis, or plotting.
when_to_use_negative:
- Input is already a normalized, LIPID MAPS-annotated concentration matrix — re-parsing
  adds no information
- Data are non-lipidomics (e.g. metabolite feature tables without lipid species nomenclature),
  where LIPID MAPS classification cannot be applied
- Raw mass-spectrometry files (e.g. mzML) needing peak detection — use LipidSearch
  or LIQUID first; ADViSELipidomics only handles their outputs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: ADViSELipidomics
  role: Shiny app that performs the LIPID MAPS-based parsing of lipid species and
    links the result to sample metadata for downstream analyses
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of lipid identification and quantification output files that the parsing
    step consumes
- name: LIQUID
  role: Alternative source of lipid identification and quantification output files
    consumed by the parsing step
- name: metabolomicsWorkbenchR
  role: Provides do_query() to download Metabolomics Workbench lipidomic experiments
    whose species are then parsed
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:f3d17387bb08bca7b9ae695a1e9c773c40fcd163a954f53333dbebd54f94303e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIQUID
  - LipidSearch
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lipid-species-parsing-lipid-maps-classification

## Summary

Parse lipid species identifiers from lipidomics quantification outputs (LipidSearch, LIQUID, or Metabolomics Workbench experiments) using the LIPID MAPS classification, producing structured per-lipid annotations that feed exploratory and differential analyses in ADViSELipidomics.

## When to use

Use when a lipid abundance matrix lacks structured annotations (class, chain composition) and you need to decompose lipid species names before normalization, statistical analysis, or plotting.

## When NOT to use

- Input is already a normalized, LIPID MAPS-annotated concentration matrix — re-parsing adds no information
- Data are non-lipidomics (e.g. metabolite feature tables without lipid species nomenclature), where LIPID MAPS classification cannot be applied
- Raw mass-spectrometry files (e.g. mzML) needing peak detection — use LipidSearch or LIQUID first; ADViSELipidomics only handles their outputs

## Inputs

- LipidSearch output files (lipid identification and quantification tables, e.g. Case Study 1 archive)
- LIQUID output files (lipid identification and quantification)
- Metabolomics Workbench lipidomic experiment downloaded in real time via metabolomicsWorkbenchR::do_query()
- Target file with sample metadata

## Outputs

- Parsed lipid species table annotated with LIPID MAPS classification (class/category per lipid species)
- Lipid-by-sample abundance matrix joined to sample metadata, ready for normalization and statistical analysis

## How to apply

Load the quantification output (LipidSearch or LIQUID files for local data, or a Metabolomics Workbench experiment retrieved via metabolomicsWorkbenchR's do_query()) into the ADViSELipidomics Shiny app. Run the lipid-species parsing step, which applies the LIPID MAPS classification to each lipid identifier to extract classification information (e.g. lipid class/category) from the species name. Combine the parsed lipid information with the sample metadata from the target file so that downstream steps — exploratory analysis, internal-standard-based normalization to absolute concentrations, design-matrix construction, and differential abundance testing — operate on correctly annotated lipids. Parsing is a prerequisite: the statistical-analysis tab expects lipid species parsed via LIPID MAPS together with sample information before a simple two-group or complex covariate design can be defined.

## Related tools

- **ADViSELipidomics** (Shiny app that performs the LIPID MAPS-based parsing of lipid species and links the result to sample metadata for downstream analyses) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of lipid identification and quantification output files that the parsing step consumes)
- **LIQUID** (Alternative source of lipid identification and quantification output files consumed by the parsing step)
- **metabolomicsWorkbenchR** (Provides do_query() to download Metabolomics Workbench lipidomic experiments whose species are then parsed)

## Examples

```
library(ADViSELipidomics); run_ADViSELipidomics()  # then load LipidSearch outputs and parse lipid species via LIPID MAPS in the Shiny UI
```

## Evaluation signals

- Every lipid species in the matrix resolves to a valid LIPID MAPS classification entry; unparseable identifiers are flagged rather than silently dropped
- Parsed lipid class labels are consistent across replicates of the same species in the abundance matrix
- Downstream normalization (when internal lipid standards are present) yields absolute concentration values per lipid and per sample without missing-class errors
- Case Study 1 (LipidSearch archive from ADViSELipidomics_book) parses without errors and reproduces the paper's exploratory-analysis inputs

## Limitations

- Parsing relies on lipid species nomenclature conforming to LIPID MAPS classification; non-standard identifiers may fail or be misclassified
- The parsing step alone does not normalize data — absolute concentrations require internal lipid standards to be present in the experiment
- Screen resolution can distort the Shiny interface; the README advises adjusting zoom (Ctrl/Cmd +/-) which may complicate locating the parsing tab

## Evidence

- [abstract] Abstract claim that parsing is the first analytical operation on the quantification outputs: "extracts information by parsing lipid species (using LIPID MAPS classification)"
- [intro] Intro detail tying parsing to sample metadata and downstream analyses: "and, together with information available on the samples,
allows performing several exploratory and statistical analyses"
- [readme] README statement of inputs handled and parsing basis: "It copes with the outputs from
LipidSearch and LIQUID for lipid identification and quantification, and
with data available from the Metabolomics Workbench."
- [readme] README on parsing plus normalization dependency on internal standards: "extracts information by parsing lipid species (using LIPID MAPS
classification) and, together with information available on the samples,
allows performing several exploratory and statistical analyses."
- [supplementary] Case Study 1 data files used to exercise the parsing workflow: "If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1"
