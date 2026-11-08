---
name: mztab-feature-report-parsing
description: Use when you have run a SmartPeak (CLI or GUI) workflow over mzML injections
  and exported feature reports, and you need to read those tables in Python to score
  peak integration correctness or extract quantitative features.
when_to_use_negative:
- Input is raw vendor or mzML spectra before the SmartPeak workflow has run — feature
  reports do not exist yet
- Analysis targets untargeted de novo identification rather than scoring integrations
  of features already picked and selected by SmartPeak
- Data are outside the CE-, GC-, LC-MS(/MS), and HPLC targeted/semi-targeted metabolomics,
  lipidomics, and fluxomics scope that SmartPeak supports
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyOpenMS
  role: Python package used to parse and process the exported feature/report files
    for downstream scoring
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak CLI
  role: Command-line equivalent of the SmartPeak GUI that executes the workflow and
    exports featuredb/pivottable reports
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: Docker
  role: Container runtime for the SmartPeak CLI with the data directory mounted as
    a volume
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: SmartPeak
  role: Produces the feature reports being parsed; automates peak detection, integration,
    and QC reporting
  repo: https://github.com/AutoFlowResearch/SmartPeak
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:6d97a29e48f816dd02b347b3a83831653928819571293f048d2573735aa99932
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - pyOpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# mzTab/feature report parsing for SmartPeak outputs

## Summary

Parses SmartPeak-exported feature report files (e.g. FeatureDB.csv, PivotTable.csv) using pyOpenMS and pandas to compute downstream metrics such as the fraction of correctly integrated features in m/z and RT dimensions. It bridges the SmartPeak automated targeted-metabolomics workflow to quantitative validation against reference integrations.

## When to use

Use when you have run a SmartPeak (CLI or GUI) workflow over mzML injections and exported feature reports, and you need to read those tables in Python to score peak integration correctness or extract quantitative features.

## When NOT to use

- Input is raw vendor or mzML spectra before the SmartPeak workflow has run — feature reports do not exist yet
- Analysis targets untargeted de novo identification rather than scoring integrations of features already picked and selected by SmartPeak
- Data are outside the CE-, GC-, LC-MS(/MS), and HPLC targeted/semi-targeted metabolomics, lipidomics, and fluxomics scope that SmartPeak supports

## Inputs

- FeatureDB.csv exported by SmartPeak CLI (--report featuredb)
- PivotTable.csv exported by SmartPeak CLI (--report pivottable)
- SmartPeak session output from a completed workflow over mzML injections (e.g. FIAMS_FullScan_Unknowns with workflow.csv and parameters.csv)
- Manually verifiable reference peak integrations in m/z and RT dimensions

## Outputs

- Pandas/pyOpenMS feature table joined across injections and metavalues
- Fraction of correctly integrated features (target > 0.95 with user-defined optimization methods)
- Per-feature integration correctness labels and failure reasons (e.g. inaccurate peak picking or annotation)

## How to apply

Run the SmartPeak CLI workflow (e.g. LOAD_RAW_DATA through MERGE_INJECTIONS and STORE_FEATURES_SAMPLE_GROUP) in the autoflowresearch/smartpeak-cli Docker container with the data directory mounted, then export reports with the featuredb and pivottable options so FeatureDB.csv and PivotTable.csv are written to --reports-out-dir. Load the exported CSVs with pyOpenMS and/or pandas — the paper states these files can be parsed and processed by the pyOpenMS Python package — and join feature metavalues across injections. Compute the fraction of features whose peaks are correctly integrated in both the m/z and RT dimensions by comparing exported integrations against manually verifiable reference integrations. Judge success by the article's threshold: user-defined optimization methods should yield >95% correctly integrated features (baseline automated workflow: 94.3% of 574 Spike-in features, 89.4% of 5,213 yeast features, 88.3% of 7,094 mouse-brain features, 94.9% of 5,050 serum features).

## Related tools

- **pyOpenMS** (Python package used to parse and process the exported feature/report files for downstream scoring) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak CLI** (Command-line equivalent of the SmartPeak GUI that executes the workflow and exports featuredb/pivottable reports) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **Docker** (Container runtime for the SmartPeak CLI with the data directory mounted as a volume) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **SmartPeak** (Produces the feature reports being parsed; automates peak detection, integration, and QC reporting) — https://github.com/AutoFlowResearch/SmartPeak

## Examples

```
docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash
```

## Evaluation signals

- SmartPeak CLI exits with status 0 and both FeatureDB.csv and PivotTable.csv exist as valid CSV files in --reports-out-dir
- Computed correctly-integrated feature fraction exceeds 0.95 when user-defined optimization methods are applied (per-dataset automated baseline: 94.3%, 89.4%, 88.3%, 94.9%)
- Residual ~5% of non-correctly-integrated features are attributable to factors such as inaccurate peak picking or annotation, not parser errors
- Feature counts in the parsed tables match the expected feature totals (e.g. 574 Spike-in, 5,213 yeast, 7,094 mouse brain, 5,050 serum features)
- All workflow steps in the session complete, including background-interference estimation and filtering steps (ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, FILTER_FEATURES_BACKGROUND_INTERFERENCES)

## Limitations

- The remaining ~5% of features may not be correctly integrated even with user-defined optimization methods, due to factors such as inaccurate peak picking or annotation
- Generic linear methods of vendor tools recover fewer correct integrations than SmartPeak's non-linear Gaussian-Mixtures peak picking, calibration, and accuracy/precision methods
- SmartPeak GUI table views are capped (5,000 feature entries; 9,000 chromatogram points), so very large datasets must be inspected via exported reports instead

## Evidence

- [results] >95% correctly integrated features finding: [evidence span withheld — anchor status: not_found]
- [results] baseline per-dataset integration rates: [evidence span withheld — anchor status: not_found]
