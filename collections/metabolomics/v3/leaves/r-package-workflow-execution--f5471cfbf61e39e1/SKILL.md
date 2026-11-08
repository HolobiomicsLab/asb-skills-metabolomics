---
name: r-package-workflow-execution
description: Use when running an R-based LC/HRMS pipeline such as IDSL.IPA where a
  single workflow function consumes a parameter spreadsheet (xlsx) pointing at raw
  MS1 data and output directories.
when_to_use_negative:
- Input is already a processed peak table or aligned data matrix — post-processing
  steps like adduct identification, statistics filtering, or molecular formula annotation
  (e.g. via IDSL.UFA) belong downstream, not inside this workflow execution skill
- Data are MS/MS fragmentation spectra rather than MS1-only LC/HRMS scans — use IDSL.CSA
  instead of the IPA peak-picking workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: IDSL.IPA
  role: The R package whose IPA_workflow function consumes the parameter spreadsheet
    and runs the full peak-picking pipeline
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime language in which the workflow is invoked (install.packages then IPA_workflow)
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:bdd43cfa4d3fa46cffde1007383e80919739a7729e4a8a13ae15308057028b78
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# r-package-workflow-execution

## Summary

Executes an R package's top-level workflow entry point (e.g. IPA_workflow) by configuring a parameter spreadsheet that supplies file locations, processing-thread counts, and output paths, then calls the workflow function on MS1-level HRMS data in mzXML, mzML, or netCDF format.

## When to use

Use when running an R-based LC/HRMS pipeline such as IDSL.IPA where a single workflow function consumes a parameter spreadsheet (xlsx) pointing at raw MS1 data and output directories.

## When NOT to use

- Input is already a processed peak table or aligned data matrix — post-processing steps like adduct identification, statistics filtering, or molecular formula annotation (e.g. via IDSL.UFA) belong downstream, not inside this workflow execution skill
- Data are MS/MS fragmentation spectra rather than MS1-only LC/HRMS scans — use IDSL.CSA instead of the IPA peak-picking workflow

## Inputs

- Parameter spreadsheet (IPA_parameters.xlsx) with address/thread parameters populated
- MS1-level HRMS data files (mzXML, mzML, or netCDF)
- R environment with the package installed (e.g. install.packages('IDSL.IPA'))

## Outputs

- Processed chromatographic peak data written to the PARAM0010 output location (e.g. peaklist CSVs, aligned peak tables, directories such as 'peaklists' and 'peak_alignment')

## How to apply

Download the package's parameter spreadsheet (IPA_parameters.xlsx), fill the address-type parameters: PARAM0007 = location of MS1-level HRMS data (mzXML/mzML/netCDF), PARAM0010 = output location, PARAM0006 = number of processing threads, then pass the spreadsheet path to the workflow entry point via library(IDSL.IPA); IPA_workflow(<spreadsheet path>). The workflow automatically orchestrates the pipeline's internal stages — 12C/13C isotopologue pairing in each mass spectrum, chromatographic peak detection on extracted ion chromatograms, and retention time correction — so no per-stage invocation is needed. Use multiple processing threads (PARAM0006) when handling larger studies since peak detection and alignment are parallelized. Rationale: the spreadsheet is the documented single input contract for the workflow function, keeping parameterization reproducible and avoiding hand-edited R scripts.

## Related tools

- **IDSL.IPA** (The R package whose IPA_workflow function consumes the parameter spreadsheet and runs the full peak-picking pipeline) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime language in which the workflow is invoked (install.packages then IPA_workflow)) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("path/to/IPA_parameters.xlsx")
```

## Evaluation signals

- The PARAM0007 spreadsheet cell points to an existing directory containing mzXML/mzML/netCDF files; the workflow errors immediately if paths are wrong
- Number of processing threads (PARAM0010 and PARAM0006) are set to integers matching available CPU cores; leaving default '1' on a many-core machine is a misuse signal
- Output directory (PARAM0010) contains generated peaklist and alignment artifacts after run completion; absence indicates the workflow was not actually executed
- Re-running with identical parameters reproduces the same peak counts — e.g. the article's benchmark on 003.d from MTBLS1684 yields 20,253 total peaks
- No manual edits to R scripts are needed; if per-stage functions are being called individually, the parameter-spreadsheet contract is being violated

## Limitations

- The article does not state the IDSL.IPA software version used, so reproducibility across package versions is not guaranteed
- The article notes that low-abundance peaks passing the S/N ratio threshold were manually rejected, and additional rules to automatically filter them are still needed
- IDSL.IPA aligned data matrices require further processing (adduct/fragment identification, statistics filtering, molecular formula annotation) before they are analysis-ready
- Parameter handling in Windows vs Linux environments differs for parallel processing (mentioned in IDSL.UFA README; apply the same care here)

## Evidence

- [readme] PARAM0007 to the location of the MS1-level HRMS data, PARAM0010 to the output location, and PARAM0006 to the number of processing threads: [evidence span withheld — anchor status: not_found]
