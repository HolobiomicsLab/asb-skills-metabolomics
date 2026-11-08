---
name: molecular-formula-isotopic-carbon-removal
description: Use when processing peak abundance tables with assigned molecular formulas
  from direct-injection FT-ICR MS (e.g. a MetaboDirect-style Report.csv) where 13C
  isotopologue peaks must be removed before normalization and analysis.
when_to_use_negative:
- The input data set has already been filtered by upstream processing (e.g. the S.
  fallax leachate data set, where no peaks were filtered because the original authors
  had pre-filtered it)
- The input is raw FT-ICR MS spectra rather than a formula-assigned peak table — MetaboDirect
  expects data after molecular formula assignment has been performed
- The goal is chromatography-coupled MS processing, where isotope peaks may be handled
  by instrument/vendor software rather than by formula-based filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline whose data pre-processing step performs the 13C isotopic-mass removal
    alongside m/z, error, and sample-presence filters
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Used within the Python-based pipeline to load and filter the peak/formula
    table (Report.csv)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:bf3f377445421b86781514f3d77a6fb8b5a48090c101c1870a9a395e976ccbe8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# molecular-formula-isotopic-carbon-removal

## Summary

A MetaboDirect data pre-processing filter step that removes FT-ICR MS peaks whose assigned molecular formulas contain isotopic 13C, preventing isotopologues from duplicating monoisotopic peaks in downstream chemodiversity and statistical analyses.

## When to use

Use when processing peak abundance tables with assigned molecular formulas from direct-injection FT-ICR MS (e.g. a MetaboDirect-style Report.csv) where 13C isotopologue peaks must be removed before normalization and analysis.

## When NOT to use

- The input data set has already been filtered by upstream processing (e.g. the S. fallax leachate data set, where no peaks were filtered because the original authors had pre-filtered it)
- The input is raw FT-ICR MS spectra rather than a formula-assigned peak table — MetaboDirect expects data after molecular formula assignment has been performed
- The goal is chromatography-coupled MS processing, where isotope peaks may be handled by instrument/vendor software rather than by formula-based filtering

## Inputs

- Peak abundance table with assigned molecular formulas from initial FT-ICR MS processing (e.g. Report.csv)
- Metadata table (metadata.csv) linking samples, used to evaluate the minimum-samples presence threshold
- User-specified m/z range and minimum sample-presence threshold

## Outputs

- Filtered peak table (.csv) with 13C-containing formula peaks removed, retaining error (ppm) and sample-presence criteria
- Peak table with thermodynamic indices (NOSC, GFE, AI_mod, DBE), compound classes, and normalized/unnormalized intensities for downstream MetaboDirect steps

## How to apply

During the MetaboDirect data pre-processing step, load the peak abundance and assigned molecular formula table (e.g. the example Report.csv) with pandas, then drop peaks whose assigned formula contains isotopic 13C in combination with the other pre-processing filters: the user-specified m/z range, formula-assignment error ≤ 0.5 ppm, and presence in at least the user-defined minimum number of samples. The rationale is that 13C-containing formula assignments represent isotopic masses rather than distinct metabolites, so retaining them would double-count peaks and confound thermodynamic index calculations (NOSC, GFE, AI_mod, DBE), compound-class assignment, and intensity normalization. Order matters: isotopic removal happens on the formula-assigned table before the test_normalization (SPANS) companion script selects the normalization method and before chemodiversity or transformation-network analysis. Export the filtered peak table as .csv so the removal is auditable; in the bacterium-phage example data set this step removed approximately 200 masses.

## Related tools

- **MetaboDirect** (Pipeline whose data pre-processing step performs the 13C isotopic-mass removal alongside m/z, error, and sample-presence filters) — https://github.com/Coayala/MetaboDirect
- **pandas** (Used within the Python-based pipeline to load and filter the peak/formula table (Report.csv))

## Examples

```
pip install metabodirect; metabodirect -h
```

## Evaluation signals

- In the bacterium-phage model system data set, approximately 200 masses should be removed by this filter; a substantially different count suggests a mis-specified formula filter
- After filtering, no retained peak should have a formula containing isotopic carbon (13C)
- The filtered peak table .csv exists, is readable, and column counts match the unfiltered input minus removed isotopic peaks
- Co-applied filters are consistent: all retained peaks have formula-assignment error ≤ 0.5 ppm and fall within the user-defined m/z range and minimum-sample threshold

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analysis even after isotopic removal
- Removal quality depends on the accuracy of the upstream molecular formula assignment (MetaboDirect expects formulas assigned before the pipeline; MFAssignR is referenced as an alternative to Formularity for this step)
- If the input data set was already filtered by the original authors, this step removes no peaks and provides no additional value

## Evidence

- [intro] Peaks filtered by m/z values, isotopic presence (13C), error in formula assignment (0.5 ppm), and minimum number of samples (user-defined threshold): [evidence span withheld — anchor status: not_found]
- [results] ~200 masses removed from the bacterium-phage data set due to isotopic carbon formulas: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
- [results] S. fallax data set required no filtering because it was pre-filtered: "No peak was filtered in the S. fallax data set as it was previously filtered by the authors of that study"
- [readme] MetaboDirect expects formula-assigned data and is a command-line FT-ICR MS pipeline: "A comprehensive command-line based pipeline for the analysis of direct injection FT-ICR mass spectrometry data."
