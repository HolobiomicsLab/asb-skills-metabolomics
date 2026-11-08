---
name: lipidsearch-output-preprocessing
description: Use when the input is a LipidSearch (or LIQUID) lipid identification/quantification
  output and the goal is a cleaned, LIPID MAPS-parsed concentration matrix ready for
  exploratory or statistical lipidomics analysis.
when_to_use_negative:
- The data source is a Metabolomics Workbench lipidomic experiment rather than a LipidSearch/LIQUID
  output — ADViSELipidomics ingests those via a different path (metabolomicsWorkbenchR
  do_query()), so the LipidSearch preprocessing route does not apply.
- The experiment lacks internal lipid standards and the goal is absolute per-lipid
  quantification — normalization to absolute concentrations is only performed when
  internal lipid standards are present.
- 'The input is raw mass-spectrometry data needing peak detection/alignment: this
  skill expects vendor-produced identification and quantification tables, not spectra.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: Shiny application whose preprocessing module ingests LipidSearch/LIQUID outputs,
    parses lipid species with LIPID MAPS classification, and performs internal-standard-based
    normalization to absolute concentrations
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Upstream vendor tool whose identification and quantification output files
    are the direct input to the preprocessing module
- name: Shiny
  role: Web framework in which the ADViSELipidomics preprocessing interface is implemented
    and launched
  repo: https://github.com/ShinyFabio/ADViSELipidomics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:f2fd802a4efb27802933146a63323ccfe12db163540c7497b5cfa618798673e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lipidsearch-output-preprocessing

## Summary

Preprocess vendor lipidomics quantification tables (LipidSearch or LIQUID outputs) in the ADViSELipidomics Shiny app: parse lipid species identifiers via LIPID MAPS classification and, when internal lipid standards are present, normalize the data matrix to absolute concentration values per lipid and sample.

## When to use

Use when the input is a LipidSearch (or LIQUID) lipid identification/quantification output and the goal is a cleaned, LIPID MAPS-parsed concentration matrix ready for exploratory or statistical lipidomics analysis.

## When NOT to use

- The data source is a Metabolomics Workbench lipidomic experiment rather than a LipidSearch/LIQUID output — ADViSELipidomics ingests those via a different path (metabolomicsWorkbenchR do_query()), so the LipidSearch preprocessing route does not apply.
- The experiment lacks internal lipid standards and the goal is absolute per-lipid quantification — normalization to absolute concentrations is only performed when internal lipid standards are present.
- The input is raw mass-spectrometry data needing peak detection/alignment: this skill expects vendor-produced identification and quantification tables, not spectra.

## Inputs

- LipidSearch output files for lipid identification and quantification (e.g. Case_Study_1.zip archive contents from the ADViSELipidomics_book data_example repository)
- LIQUID tool output files (alternative supported vendor format)
- Sample metadata / sample information accompanying the experiment
- Internal lipid standard information (required for absolute concentration normalization)

## Outputs

- Parsed lipid species table annotated via LIPID MAPS classification
- Normalized concentration data matrix (absolute values per lipid and sample) when internal standards are present
- Matrix ready for exploratory analysis, differential-abundance testing, and batch effect correction

## How to apply

Install ADViSELipidomics (R > 4.0) with devtools after OS-specific dependency setup (Rtools on Windows; brew imagemagick@6 and cairo on macOS; apt packages such as libcairo2-dev and libmagick++-dev on Ubuntu), then launch the Shiny app with run_ADViSELipidomics(). Load the LipidSearch output files into the preprocessing module; the app parses lipid species identifiers using the LIPID MAPS classification and joins them with sample metadata to enable exploratory and statistical analyses. Decision point: if the experiment includes internal lipid standards, run the internal-standard-based normalization step so the matrix yields normalized (absolute) concentration values per lipid and sample — e.g. recovery-normalized values used in Case Study 1 (monocytes vs iDCs); without internal standards, absolute quantification is not supported and only relative analyses apply. Downstream, the normalized matrix feeds differential-abundance testing (simple designs like two-group comparisons, or complex designs) and batch effect correction. A Dockerized alternative (docker run -p 80:80 shinyfabio/adviselipidomics:latest) provides the same preprocessing pipeline via browser at localhost:80.

## Related tools

- **ADViSELipidomics** (Shiny application whose preprocessing module ingests LipidSearch/LIQUID outputs, parses lipid species with LIPID MAPS classification, and performs internal-standard-based normalization to absolute concentrations) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Upstream vendor tool whose identification and quantification output files are the direct input to the preprocessing module)
- **Shiny** (Web framework in which the ADViSELipidomics preprocessing interface is implemented and launched) — https://github.com/ShinyFabio/ADViSELipidomics

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Reproduce Case Study 1: after normalization on the Case_Study_1.zip LipidSearch files, the normalized concentration table must match the values reported in the article's Supplementary Material per lipid and sample.
- The percentage of recovery is computed per sample during internal-standard normalization; the monocytes-vs-iDCs comparison on the recovered absolute concentrations should reproduce the Case Study 1 results in the Supplementary Material.
- Parsed lipid species identifiers conform to the LIPID MAPS classification (class/subclass/species decomposition extractable for each lipid name).
- The output is a samples × lipids matrix of normalized concentrations with no missing lipid-species annotations after LIPID MAPS parsing.
- Package installation succeeds across the documented platforms (Windows with Rtools, macOS with Homebrew cairo/imagemagick@6, Ubuntu 18.04 with the listed apt libraries), indicating the preprocessing stack is intact.

## Limitations

- Absolute concentration normalization is only available when the experiment includes internal lipid standards; otherwise only relative/exploratory analyses are possible.
- The batch correction step applied downstream is not identified by name in the article's discussion, so the specific correction method cannot be specified from this source.
- Power to detect non-linear drift is low: the 2D pooled-QC MAD pattern is not visible in replicate (n=1 vs n=2) QC samples because variance is very low.
- The Shiny UI rendering depends on monitor resolution/zoom (adjust with Ctrl/Cmd +/−), and the Docker version is tested on Chrome — Firefox may have problems.

## Evidence

- [abstract] Internal-standard normalization to absolute concentrations: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [intro] LipidSearch/LIQUID input handling and LIPID MAPS parsing: "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification"
- [intro] Lipid MAPS-based parsing enabling statistical analyses: "extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses"
- [readme] Case Study 1 test data availability: "If you want to test the software with a LipidSearch output, here we provide the files used in the Case Study 1"
- [methods] Application launch invocation: "library("ADViSELipidomics")
run_ADViSELipidomics()"
