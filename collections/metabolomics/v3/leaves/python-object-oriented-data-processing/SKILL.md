---
name: python-object-oriented-data-processing
description: Use when a Python-based LC–MS preprocessing workflow must be structured
  as reusable, serializable objects rather than ad-hoc scripts — e.g., building a
  curated data matrix from mzML acquisitions with QC-filter steps (Rt, blank, RSD,
  prevalence, D-ratio) that must be re-run reproducibly.
when_to_use_negative:
- If the input is already a curated feature table that only needs downstream statistical
  analysis — this skill is for structuring and executing the LC–MS curation itself,
  not for post-curation modeling; the paper notes scikit-learn is used for state-of-the-art
  model generation after preprocessing.
- If the study requires interactive per-run manual parameter tuning rather than a
  fixed, reusable curation strategy — the Pipeline architecture encapsulates a fixed
  sequence of Processors intended for reproducible, semi-automated or automated execution,
  not ad-hoc interactive editing.
- If the target analysis is not LC–MS untargeted metabolomics QC (e.g., GC–MS or proteomics
  workflows) — the architecture's filters and CWT peak-picking are parameterized for
  chromatographic LC–MS data and the NIST plasma reference-material use case.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Python package providing the object-oriented architecture (DataContainer,
    Pipeline, Processor objects) that encodes LC–MS data-curation steps as reusable,
    run-independent operations.
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Library used by TidyMS to read raw LC–MS mzML files into the MSData objects
    that feed the DataContainer.
- name: Pandas
  role: Data structure layer used by the DataContainer to store the data matrix and
    associated sample and feature descriptors.
- name: Numpy
  role: Numerical backend used in the TidyMS package to implement Processor operations
    (filters and corrections) on the data matrix.
- name: Scipy
  role: Scientific library used by TidyMS (e.g., for signal processing such as CWT
    peak picking) during feature extraction from the raw MSData.
- name: Python
  role: Programming language in which the Pipeline-of-Processors architecture is implemented,
    enabling integration with scikit-learn for downstream modeling.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:1d4ef3ea0b77cfb3e347f78211e02b9257037ffda939b1f08970118c0d1907c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - Pandas
  - Python
  - Scipy
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# python-object-oriented-data-processing

## Summary

This skill is TidyMS's fixed, object-oriented software architecture for LC–MS data curation: raw mzML data are read via pyopenms into a DataContainer that stores the data matrix plus sample and feature descriptors in Pandas DataFrames, and curation is executed as a Pipeline of Processor objects encapsulating filtering and correction operations, so a sequence of Processors defines a reusable, run-independent data-curation strategy.

## When to use

Use when a Python-based LC–MS preprocessing workflow must be structured as reusable, serializable objects rather than ad-hoc scripts — e.g., building a curated data matrix from mzML acquisitions with QC-filter steps (Rt, blank, RSD, prevalence, D-ratio) that must be re-run reproducibly.

## When NOT to use

- If the input is already a curated feature table that only needs downstream statistical analysis — this skill is for structuring and executing the LC–MS curation itself, not for post-curation modeling; the paper notes scikit-learn is used for state-of-the-art model generation after preprocessing.
- If the study requires interactive per-run manual parameter tuning rather than a fixed, reusable curation strategy — the Pipeline architecture encapsulates a fixed sequence of Processors intended for reproducible, semi-automated or automated execution, not ad-hoc interactive editing.
- If the target analysis is not LC–MS untargeted metabolomics QC (e.g., GC–MS or proteomics workflows) — the architecture's filters and CWT peak-picking are parameterized for chromatographic LC–MS data and the NIST plasma reference-material use case.

## Inputs

- Raw LC–MS data files in mzML format (read via pyopenms into TidyMS MSData objects)
- Sample mapping metadata associating sample types to sample classes (study samples, pooled/intrastudy QC, blanks, QC)
- Extracted feature data matrix with per-feature and per-sample descriptors (built from retention time alignment, peak picking, deisotoping, integration, and adduct grouping)
- User-defined Processor list defining the data-curation strategy (e.g., Rt filter, blank filter, RSD filter, prevalence filter, D-ratio filter, LOESS batch correction)

## Outputs

- Processed DataContainer holding the curated data matrix and its sample/feature descriptors after applying the Pipeline's Processor sequence
- Serialized DataContainer file written to disk as the reusable, reproducible output of the data-curation step

## How to apply

First read the raw LC–MS mzML files with pyopenms into TidyMS MSData objects, then extract chromatographic features using the CWT-based peak-picking algorithm (with chromatogram building and centWave-style detection) tuned for LC–MS data. Load the extracted data matrix together with its sample-mapping and feature descriptors into a TidyMS DataContainer, which stores the matrix and associated descriptors as Pandas DataFrames; the sample-mapping attribute associates sample types (study samples, pooled/intrastudy QC, blanks) to sample classes so Processor behavior can depend on class. Build a Pipeline object that stores an ordered list of Processor objects encapsulating the curation steps — e.g., the paper's fixed curation strategy applies an Rt filter (retention time ≥ 90 s, since system dead time was ~0.8 min), a QC-presence filter, a blank filter (area ≤ 10-fold max blank area is set to zero), an RSD filter in pooled QC (RSD ≤ 20%), a 100% intraclass prevalence filter (area threshold 5), and a D-ratio filter (MAD of QC relative to MAD of study samples, D-ratio ≥ 10% removed). Execute the Pipeline over the DataContainer to transform the data matrix, and finally serialize the processed DataContainer to disk so the full curation sequence is reproducible and independent of any single run. The rationale is the paper's claim that Python's general-purpose design, integration with scikit-learn-style processing, and a Pipeline of Processors make curation reusable, automatable, and FAIR-compliant, with concrete running time of ~66 s for 31 raw files (~2 s per sample for feature detection).

## Related tools

- **TidyMS** (Python package providing the object-oriented architecture (DataContainer, Pipeline, Processor objects) that encodes LC–MS data-curation steps as reusable, run-independent operations.) — https://github.com/griquelme/tidyms
- **pyopenms** (Library used by TidyMS to read raw LC–MS mzML files into the MSData objects that feed the DataContainer.)
- **Pandas** (Data structure layer used by the DataContainer to store the data matrix and associated sample and feature descriptors.)
- **Numpy** (Numerical backend used in the TidyMS package to implement Processor operations (filters and corrections) on the data matrix.)
- **Scipy** (Scientific library used by TidyMS (e.g., for signal processing such as CWT peak picking) during feature extraction from the raw MSData.)
- **Python** (Programming language in which the Pipeline-of-Processors architecture is implemented, enabling integration with scikit-learn for downstream modeling.)

## Examples

```
from tidyms.core.data_container import DataContainer; from tidyms.core.pipeline import Pipeline; dc = DataContainer('extracted_matrix.csv', 'sample_mapping.csv'); pipe = Pipeline([RtFilter(rt_min=90), BlankFilter(ratio=10), RsdFilter(0.2), PrevalenceFilter(0.5, 5), DRatioFilter(0.1)]); dc = pipe.execute(dc); dc.serialize('curated_matrix.pkl')
```

## Evaluation signals

- Check that a Pipeline object is constructed that stores an ordered list of Processor objects, and that this list is the single source of truth for the curation strategy (no ad-hoc, un-encapsulated filtering calls outside the Pipeline).
- Verify the DataContainer is instantiated with the extracted data matrix and that its sample-mapping attribute correctly associates sample types to sample classes so Processor behavior (e.g., RSD in pooled QC, blank filtering) is class-dependent.
- Confirm the serialized output is a processed DataContainer file whose per-feature and per-sample descriptors are preserved, so the curated matrix can be reloaded without losing QC metadata.
- Cross-check the applied Processor thresholds against the paper's curation strategy: retention time ≥ 90 s, blank-area ratio ≤ 10-fold, RSD ≤ 20% in pooled/intrastudy QC, area threshold of 5 for 100% intraclass prevalence, and D-ratio ≥ 10% removal.
- Measure reproducibility and runtime: the paper reports ~66 s for 31 raw data files with feature detection ~2 s per sample; a correct application of the Pipeline should complete in the same order of magnitude on equivalent mzML input.

## Limitations

- TidyMS does not provide feature annotation capability (the article marks it ❌ relative to other tools).
- The article does not state software version numbers for TidyMS, Python, or scikit-learn, so exact reproducibility may require pinning dependencies manually.
- The fixed Processor sequence encapsulates the paper's specific curation strategy (Rt ≥ 90 s, RSD ≤ 20%, D-ratio ≥ 10%, etc.); adapting it to a different chromatography or matrix (e.g., non-plasma samples or a different LC method with different dead time) requires re-deriving thresholds.
- No accession IDs are provided for the NIST candidate RM 8231 or SRM 1950 reference materials used, complicating direct data-set reuse without obtaining them separately.

## Evidence

- [supplementary] the curation mechanism in which data are processed inside a Pipeline as a combination of Filters and Correctors applied to the tabular data matrix: [evidence span withheld — anchor status: not_found]
- [supplementary] Instantiate a DataContainer that stores the extracted data matrix and associated feature and sample descriptors in Pandas DataFrames, using the sample-mapping attribute: [evidence span withheld — anchor status: not_found]
- [supplementary] a Pipeline object that stores a list of Processor objects encapsulating feature-filtering methods: [evidence span withheld — anchor status: not_found]
- [abstract] The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows.: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows."
- [results] The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [intro] Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms.: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms."
