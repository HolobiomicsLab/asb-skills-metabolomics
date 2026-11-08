---
name: pandas-data-container-curation
description: Use when raw LC-MS data (mzML) have already yielded an extracted data
  matrix and you need a reproducible, scripted sequence of feature-filtering and correction
  steps (Rt filter, blank filter, RSD filter, LOESS batch correction) curated as an
  ordered Pipeline over a DataContainer.
when_to_use_negative:
- Input is already a fully curated feature table that only needs downstream statistical
  analysis — TidyMS is aimed at preprocessing raw/extracted LC-MS data, and its feature-annotation
  capability is lacking.
- The analysis requires feature annotation of detected peaks; the paper notes TidyMS
  does not provide this capability.
- The user needs interactive manual curation or one-off exploratory filtering rather
  than a fixed, reusable Processor pipeline.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: TidyMS
  role: Provides the Pipeline of Processors operating on a Pandas-backed DataContainer
    for LC-MS data curation
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Reads raw mzML LC-MS data into TidyMS MSData objects for feature extraction
- name: Pandas
  role: Stores the DataContainer data matrix and feature/sample descriptors as DataFrames
- name: Numpy
  role: Numerical backend for the matrix computations underlying Filters and Correctors
- name: Scipy
  role: Supports signal-processing routines (CWT peak picking) in the feature-extraction
    step feeding the DataContainer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:a0049442ddd2a82edf0548e72dfbb22247b1cf36a21f1683df85ae42b5d3fb98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - Pandas
  - Scipy
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pandas-data-container-curation

## Summary

TidyMS organizes LC-MS data curation by loading extracted feature data into a Pandas-backed DataContainer and applying a Pipeline of Processor objects (Filters and Correctors) that transform the data matrix for QC procedures in untargeted metabolomics.

## When to use

Use when raw LC-MS data (mzML) have already yielded an extracted data matrix and you need a reproducible, scripted sequence of feature-filtering and correction steps (Rt filter, blank filter, RSD filter, LOESS batch correction) curated as an ordered Pipeline over a DataContainer.

## When NOT to use

- Input is already a fully curated feature table that only needs downstream statistical analysis — TidyMS is aimed at preprocessing raw/extracted LC-MS data, and its feature-annotation capability is lacking.
- The analysis requires feature annotation of detected peaks; the paper notes TidyMS does not provide this capability.
- The user needs interactive manual curation or one-off exploratory filtering rather than a fixed, reusable Processor pipeline.

## Inputs

- Raw LC-MS data files in mzML format
- Sample mapping metadata associating sample types to sample classes (study samples, pooled/intrastudy QC samples, solvent blanks, zero-volume injection blanks, process blanks)
- Curation strategy parameters (retention-time cutoff, blank fold threshold, RSD cutoff, prevalence area threshold, D-ratio cutoff, LOESS correction settings)

## Outputs

- Processed DataContainer with the curated data matrix and associated feature and sample descriptors stored in Pandas DataFrames
- Serialized processed DataContainer file on disk as the task output

## How to apply

Read the raw LC-MS data from the input mzML file with pyopenms into TidyMS MSData objects, then extract spectral features using the CWT-based peak-picking algorithm optimized for chromatographic data together with chromatogram building and centWave-style feature detection. Instantiate a DataContainer whose data matrix and per-feature and per-sample descriptors are Pandas DataFrames, and set its sample-mapping attribute so sample classes (e.g. plasma samples, pooled/intrastudy QC samples, solvent, zero-volume injection, and process blanks) are known to the Processors. Build a Pipeline object storing an ordered list of Processor objects that encapsulate feature-filtering and correction methods — e.g. the Rt filter discarding features with retention time below 90 s (system dead time ≈ 0.8 min), the blank filter zeroing peak areas ≤ 10-fold of the maximum blank peak area, the RSD filter dropping features with RSD > 20% in pooled/intrastudy QC injections, the 100% intraclass prevalence filter with an area threshold of 5, the D-ratio filter (MAD of QC samples relative to MAD of all study samples), and LOESS batch correction restricted to features present in pooled/intrastudy QC samples. Execute the Pipeline on the DataContainer to obtain the curated data matrix with its descriptors preserved, then serialize the processed DataContainer to disk as the task output; the fixed Processor list makes the curation strategy reusable and independent of any run.

## Related tools

- **TidyMS** (Provides the Pipeline of Processors operating on a Pandas-backed DataContainer for LC-MS data curation) — https://github.com/griquelme/tidyms
- **pyopenms** (Reads raw mzML LC-MS data into TidyMS MSData objects for feature extraction)
- **Pandas** (Stores the DataContainer data matrix and feature/sample descriptors as DataFrames)
- **Numpy** (Numerical backend for the matrix computations underlying Filters and Correctors)
- **Scipy** (Supports signal-processing routines (CWT peak picking) in the feature-extraction step feeding the DataContainer)

## Examples

```
from tidyms.data_container import DataContainer; from tidyms.core import Pipeline; container = DataContainer(mzml_data, sample_mapping); pipeline = Pipeline(processors=[RtFilter(90), BlankFilter(10), RsdFilter(20), PrevalenceFilter(5), DratioFilter(0.10), LoessCorrector()]); container = pipeline.execute(container); container.serialize('curated_data.pkl')
```

## Evaluation signals

- The Pipeline object executes its stored list of Processor objects over the DataContainer without manual reordering — the Processor list defines the curation strategy.
- The curated data matrix preserves per-feature and per-sample descriptors in Pandas DataFrames consistent with the object-interaction design of Figure S5 in the paper's UML description.
- The processed DataContainer is serializable to disk as a task output file.
- Filter invariants hold: features with retention time < 90 s are removed; peak areas ≤ 10-fold the maximum blank peak area are set to 0; features with RSD > 20% in pooled/intrastudy QC injections are eliminated; area values below the threshold 5 are zeroed after the 100% intraclass prevalence filter; features with D-ratio ≥ 10% are removed.
- LOESS batch correction is only attempted for features that passed the pooled/intrastudy QC presence filter.

## Limitations

- TidyMS lacks feature annotation capability.
- No software version numbers are stated for TidyMS, Python, or scikit-learn in the article.
- Running time is fast (66 s for 31 raw files, ~2 s per sample for feature detection), but the fixed Pipeline architecture assumes the Processor list matches the experimental design (sample classes, QC types).
- No accession IDs for NIST candidate RM 8231 or SRM 1950 are provided, so reproducing the demo dataset may require additional metadata.

## Evidence

- [methods] TidyMS describes a curation mechanism in which data are processed inside a Pipeline as a combination of Filters and Correctors applied to the tabular data matrix.: [evidence span withheld — anchor status: not_found]
- [abstract] TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows.: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows."
- [results] The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] A 100% intraclass prevalence filter was subsequently applied with a threshold area value of 5, i.e., all area values below this threshold were set to zero.: "A 100% intraclass prevalence filter was subsequently applied with a threshold area value of 5, i.e., all area values below this threshold were set to zero."
