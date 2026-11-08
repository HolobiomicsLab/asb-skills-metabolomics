---
name: contaminant-feature-flagging
description: Use when raw untargeted metabolomics or lipidomics feature-intensity
  tables contain procedural-blank injections alongside real samples, and features
  enriched in blanks must be flagged as background contaminants before signal-drift
  or RSD filtering.
when_to_use_negative:
- When the dataset contains no procedural-blank injections, since the 3x sample-vs-blank
  criterion cannot be computed
- When the goal is to correct batch signal drift or detect out-of-control QCpool observations
  — those are separate QComics steps, not background-contamination flagging
- When features are already known annotated endogenous metabolites with confirmed
  spectra, rather than unfiltered feature tables needing contaminant screening
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: QComics
  role: Provides the sequential QC protocol whose first step ('correct for background
    noise and carryover') defines the 3x sample-vs-procedural-blank flagging criterion
    implemented by this skill
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime for the QComics package; the skill's mean/ratio computation and CSV
    export are implemented as an R script using dplyr and readr
- name: dplyr
  role: Used to rowwise-compute per-feature blank/sample means, ratios, and the <=
    3x flag decision
- name: readr
  role: Writes the per-feature contamination summary to flagged_features.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:2d9a6eadba4343d4ff32272b287b16f1ef5b296b5ce00ed28469639e24057fe3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - QComics
  - R
  - dplyr
  - readr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# contaminant-feature-flagging

## Summary

Flags molecular features as potential contaminants by comparing real-sample intensities against procedural-blank injections, following the QComics background-noise step that flags (rather than deletes) features whose sample mean is not more than three times the procedural-blank mean, and discards only those additionally exhibiting blank RSD > 15%.

## When to use

Use when raw untargeted metabolomics or lipidomics feature-intensity tables contain procedural-blank injections alongside real samples, and features enriched in blanks must be flagged as background contaminants before signal-drift or RSD filtering.

## When NOT to use

- When the dataset contains no procedural-blank injections, since the 3x sample-vs-blank criterion cannot be computed
- When the goal is to correct batch signal drift or detect out-of-control QCpool observations — those are separate QComics steps, not background-contamination flagging
- When features are already known annotated endogenous metabolites with confirmed spectra, rather than unfiltered feature tables needing contaminant screening

## Inputs

- Feature-intensity table (rows = molecular features, columns = samples) containing both procedural-blank injections and real study samples, e.g. a data_df exported from Sciex MultiQuant in txt format
- Column annotation identifying which injections are procedural blanks versus real study samples

## Outputs

- Per-feature summary table with blank mean, real-sample mean, sample/blank ratio, and flagged/retained contaminant decision
- List of features recommended for discard (flagged features that additionally have blank RSD > 15%)
- Reportable QC artifact such as flagged_features.csv

## How to apply

Assemble a feature table (rows = features, columns = samples) that contains both procedural-blank injections and real study samples, ideally exported from Sciex MultiQuant (> v3.0.3) as QComics expects. For each feature compute the mean intensity in the procedural blanks and the mean intensity in the real samples, then apply the QComics-adapted background-contamination criterion: flag the feature as a potential contaminant when its sample mean is not more than three times (<= 3x) its blank mean. Flagging is deliberately conservative — features are marked rather than removed upfront so that decisions can be reported; of the flagged features, only those that also show a blank relative standard deviation above 15% are recommended for actual discard. Assemble a per-feature summary with blank mean, sample mean, the sample/blank ratio, and the flagged/retained decision, and persist it (e.g. flagged_features.csv via readr) so the contamination step is reportable as QComics requires.

## Related tools

- **QComics** (Provides the sequential QC protocol whose first step ('correct for background noise and carryover') defines the 3x sample-vs-procedural-blank flagging criterion implemented by this skill) — https://github.com/ricoderks/QComics
- **R** (Runtime for the QComics package; the skill's mean/ratio computation and CSV export are implemented as an R script using dplyr and readr)
- **dplyr** (Used to rowwise-compute per-feature blank/sample means, ratios, and the <= 3x flag decision)
- **readr** (Writes the per-feature contamination summary to flagged_features.csv)

## Examples

```
Rscript -e 'remotes::install_github("ricoderks/QComics"); QComics::run_app()'
```

## Evaluation signals

- flagged_features.csv exists, parses cleanly, and contains one row per input feature with columns for blank mean, sample mean, sample/blank ratio, and decision
- Every feature with sample/blank ratio <= 3 is marked flagged; every feature with ratio > 3 is marked retained
- The discard subset (flagged features with blank RSD > 15%) is strictly a subset of the flagged set, matching an expert-reviewed reference application of the 3x rule
- The script executes end-to-end without error on the example blank and study-sample table bundled with the QComics R package (github.com/ricoderks/QComics)
- Flagging decisions are recorded (not silently dropped) so the contamination step is auditable and reportable

## Limitations

- The 3x threshold is a QComics-adapted quality criterion, not a universally validated constant; the paper notes that pre-existing QC protocols suffer from 'subjective nature and lack of harmonization between laboratories'
- Flagging is conservative: features are marked rather than removed upfront; actual discard is only recommended for flagged features that also show blank RSD > 15%
- The QComics package is in experimental lifecycle status and expects QCpool samples analysed with Sciex MultiQuant (> v3.0.3) and exported to txt format; datasets in other formats may require preprocessing

## Evidence

- [full_text] QComics flags a feature as a potential contaminant when its mean value in real samples is not more than three times its mean value in procedural blanks: [evidence span withheld — anchor status: not_found]
- [full_text] only those with blank RSD > 15% are recommended for discard: [evidence span withheld — anchor status: not_found]
- [abstract] abstract quote: "we have developed QComics, a robust, easily implementable and reportable method for monitoring and controlling data quality"
- [readme] install quote: "remotes::install_github("ricoderks/QComics")"
- [abstract] workflow quote: "The protocol operates in various sequential steps aimed to (i) correct for background noise and carryover"
