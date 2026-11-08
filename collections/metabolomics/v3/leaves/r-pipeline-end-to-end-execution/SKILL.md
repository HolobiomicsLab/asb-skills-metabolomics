---
name: r-pipeline-end-to-end-execution
description: Use when processing DDA mzXML files from pooled QC samples together with
  DIA mzXML files from individual biological samples and you need a single R script
  to extract features, rescue DDA-unique features, and annotate metabolites end-to-end.
when_to_use_negative:
- If your data are from a conventional DDA-only workflow with no DIA mzXML files to
  rescue against — the DaDIA.R pipeline is built specifically for hybrid DDA/DIA datasets.
- If the number of DIA mzXML files does not match the sample count set on line 38
  of the script, the run will be mis-specified; fix the sample count first rather
  than forcing execution.
- If you need quantification of metabolites in DDA mode only (e.g., full-scan DDA
  on individual samples), the rescue step has no DIA counterpart and adds no value.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DaDIA.R
  role: The end-to-end R program that drives feature extraction, EIC-based rescue,
    and metabolite annotation from DDA/DIA data.
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Provides the centWave peak-picking function used to extract metabolic features
    from both DDA and DIA mzXML files inside DaDIA.R.
- name: metaMS
  role: Required R dependency (version 1.25.1) for the metabolite annotation portion
    of the pipeline.
- name: R
  role: Runtime environment (version 4.0 or above) in which the DaDIA.R script is
    executed.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:d24c2a50cf2a76b00efae34bc6ef4968a2778731cc0578786d66b07394ca0bfb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-pipeline-end-to-end-execution

## Summary

Executes DaDIA.R, an R-based program that automatically extracts metabolic features and performs metabolite annotation from hybrid DDA/DIA metabolomic data, running end-to-end on mzXML files with XCMS and metaMS as dependencies.

## When to use

Use when processing DDA mzXML files from pooled QC samples together with DIA mzXML files from individual biological samples and you need a single R script to extract features, rescue DDA-unique features, and annotate metabolites end-to-end.

## When NOT to use

- If your data are from a conventional DDA-only workflow with no DIA mzXML files to rescue against — the DaDIA.R pipeline is built specifically for hybrid DDA/DIA datasets.
- If the number of DIA mzXML files does not match the sample count set on line 38 of the script, the run will be mis-specified; fix the sample count first rather than forcing execution.
- If you need quantification of metabolites in DDA mode only (e.g., full-scan DDA on individual samples), the rescue step has no DIA counterpart and adds no value.

## Inputs

- mzXML files of DIA mode runs from individual biological samples
- mzXML files of DDA mode runs from pooled QC samples

## Outputs

- Rescued-feature intensity table containing feature m/z, retention time, and rescued intensity per sample
- Streamlined metabolite annotations with MS2 spectral matches (dot product scores)

## How to apply

Set up an R 4.0+ environment with XCMS development version 3.11.4 or above and metaMS 1.25.1, plus any other packages updated to the newest available version. Confirm that the sample count on line 38 of the script matches the real number of DIA mzXML files before running. Launch the script on the DaDIA mzXML inputs; peak picking is performed by the centWave function in XCMS on both the DDA and DIA files. The script then executes the rescue step, re-determining DDA-unique feature intensities in the DIA raw data by extracted-ion-chromatogram (EIC) matching, and performs metabolite annotation by MS2 spectral matching against MoNA. The full run must finish without errors and emit a rescued-feature intensity table containing feature m/z, retention time, and rescued intensity per sample.

## Related tools

- **DaDIA.R** (The end-to-end R program that drives feature extraction, EIC-based rescue, and metabolite annotation from DDA/DIA data.) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Provides the centWave peak-picking function used to extract metabolic features from both DDA and DIA mzXML files inside DaDIA.R.)
- **metaMS** (Required R dependency (version 1.25.1) for the metabolite annotation portion of the pipeline.)
- **R** (Runtime environment (version 4.0 or above) in which the DaDIA.R script is executed.)

## Examples

```
Rscript DaDIA.R
```

## Evaluation signals

- The DaDIA.R script must run end-to-end without errors on the demo mzXML files.
- The rescued-feature output table must exist and contain the m/z, retention time, and intensity fields per sample.
- As a sanity reference, the paper reports an average of 344 rescued features (about 22% of the 1591 DDA features) in urine RP(+) data; a comparable ratio suggests correct execution.
- Manually inspecting rescued EICs against the paper's benchmark gives a ~71% true-positive rate and a ~17% false-positive rate.
- The sample count on line 38 must equal the actual number of DIA mzXML files before the run starts.

## Limitations

- The number of samples on line 38 must agree with the real number of DIA mzXML files; a mismatch will cause incorrect processing.
- Rescued features carry a substantial false-positive rate (17% by manual EIC inspection) and a 12% undetermined fraction, so rescued intensities need manual verification.
- Requires specific tool versions: R 4.0 or above, XCMS development version 3.11.4 or above, and metaMS 1.25.1; older versions may fail.
- Rescue depends on the pooled-QC DDA run being of sufficient quality — if the DDA data miss a feature, it cannot be rescued.

## Evidence

- [intro] The source code of DaDIA.R and its user protocol are available on Git Hub: [evidence span withheld — anchor status: not_found]
