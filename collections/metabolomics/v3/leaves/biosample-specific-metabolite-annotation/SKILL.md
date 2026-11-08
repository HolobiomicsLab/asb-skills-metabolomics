---
name: biosample-specific-metabolite-annotation
description: Use when annotating metabolites from 1H NMR spectra of biological samples/biofluids
  via ROI-based identification against HMDB-derived chemical shift and SRRM libraries,
  and when reproducing or validating those per-biosample databases.
when_to_use_negative:
- Automated identification of ROIs is needed — this is beyond the scope of the current
  program; ROIs must be user-defined.
- Absolute quantification of compounds is the goal — the program's primary aim is
  identification, not quantification.
- The reference database lacks NMR assignments for the metabolites of interest — libraries
  can be swapped (HMDB, BMRB), but metabolites without NMR assignments cannot be identified.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Python program that identifies potential metabolites from defined proton NMR
    ROIs using the HMDB as reference platform
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python >=3.9
  role: Runtime environment for ROIAL-NMR and database construction scripts
  repo: https://www.anaconda.com/download/
- name: pandas 2.2.3
  role: Parsing HMDB records and structuring chemical shift / SRRM tables
- name: openpyxl 3.1.5
  role: Exporting the compiled databases to Excel workbooks
- name: XlsxWriter 3.2.2
  role: Writing the per-biosample Excel output files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:374ae42e8ccc7537db01d3f521f2f4aca49b4551126585e80e03448a39dede48
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Python
  - Python >=3.9
  - ROIAL-NMR
  - XlsxWriter
  - XlsxWriter 3.2.2
  - openpyxl
  - openpyxl 3.1.5
  - pandas
  - pandas 2.2.3
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# biosample-specific-metabolite-annotation

## Summary

Build and use program-specific metabolic libraries — 1H chemical shift values and spectral region resonance multiplets (SRRMs) — derived from the HMDB for 891 metabolites, organized by biosample type (serum, saliva, sweat, urine, CSF, tissues), to annotate metabolites within user-defined NMR ROIs.

## When to use

Use when annotating metabolites from 1H NMR spectra of biological samples/biofluids via ROI-based identification against HMDB-derived chemical shift and SRRM libraries, and when reproducing or validating those per-biosample databases.

## When NOT to use

- Automated identification of ROIs is needed — this is beyond the scope of the current program; ROIs must be user-defined.
- Absolute quantification of compounds is the goal — the program's primary aim is identification, not quantification.
- The reference database lacks NMR assignments for the metabolites of interest — libraries can be swapped (HMDB, BMRB), but metabolites without NMR assignments cannot be identified.

## Inputs

- HMDB metabolite records with 1H NMR chemical shift assignments for 891 metabolites
- Biosample type specification (plasma, serum, CSF, feces, saliva, sweat, urine, tissues)
- User-defined ROIs with trends and significance levels
- Disease-specific metabolite list (optional, prepared from HMDB or other reliable sources)

## Outputs

- Chemical shift database (peak values in ppm per metabolite)
- SRRM database (multiplet regions per metabolite)
- Per-biosample-type database tables exported to an Excel workbook (openpyxl/XlsxWriter)
- Metabolite identification results with abbreviations, match ratios, matched regions, and concentration ranges

## How to apply

Download HMDB metabolite records carrying 1H NMR chemical shift assignments and parse them with Python/pandas into two databases: one of chemical shift peak values (ppm) and one of metabolite-specific SRRMs (multiplet regions from J-couplings), organized by tissue/biosample type into separate tables (plasma, serum, CSF, feces, saliva, sweat, urine). Spot-check the lactate entry: it must contain two SRRMs (1.30-1.33 and 4.08-4.13 ppm) with six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm). Export the databases and per-biosample tables to an Excel workbook with openpyxl/XlsxWriter. Identification takes the intersection of user-specified ROI specifications, metabolic NMR information, and sample-specific information, and retains only metabolites with a match ratio ≥ 50% of their SRRMs within the analyzed ROIs, excluding peaks lying beyond the analyzed ROI from the ratio; metabolites with concentration limits below 5 μM are also excluded. Rationale: both chemical shift values and metabolite-specific SRRMs are required for accurate metabolite identification.

## Related tools

- **ROIAL-NMR** (Python program that identifies potential metabolites from defined proton NMR ROIs using the HMDB as reference platform) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python >=3.9** (Runtime environment for ROIAL-NMR and database construction scripts) — https://www.anaconda.com/download/
- **pandas 2.2.3** (Parsing HMDB records and structuring chemical shift / SRRM tables)
- **openpyxl 3.1.5** (Exporting the compiled databases to Excel workbooks)
- **XlsxWriter 3.2.2** (Writing the per-biosample Excel output files)

## Examples

```
git clone https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git; cd ROIAL-NMR; python main.py
```

## Evaluation signals

- Compiled database contains 891 metabolites with NMR assignments from the HMDB NMR metabolite database.
- Lactate entry reproduces two SRRMs (1.30-1.33 and 4.08-4.13 ppm) and six peaks (1.310, 1.324, 4.084, 4.098, 4.111, 4.125 ppm).
- Identified metabolites all carry a match ratio ≥ 50% of their SRRMs within the analyzed ROIs; peaks >50% outside the analyzed ROI are excluded from the ratio.
- Metabolites with concentration limits below 5 μM are excluded from the results.
- Single-population analysis completes in about 5 seconds and multi-group analysis in less than 10 seconds.

## Limitations

- ROIAL-NMR was not built to test the accuracy of the user-defined ROIs; using user-defined NMR spectral ROIs can lead to variable results obtained by different users.
- Automated identification of ROIs and absolute quantification are not supported by the current version.
- Periodic and automatic synchronization of NMR chemical shift databases is planned but not yet implemented, so database updates require manual input.
- HMDB was used as a demonstrating platform; the program can use any metabolite database containing NMR assignments (e.g., BMRB).

## Evidence

- [intro] 891 metabolites from HMDB: "it enables the rapid (within 5 seconds) identification of a broad range of metabolites from ROIs by utilizing chemical shift data for 891 metabolites from the HMDB NMR metabolite database"
- [methods] SRRMs required: "we found that both are required for accurate metabolite identification"
- [methods] match ratio filter: "Only metabolites with a match ratio ≥ 50% of their SRRMs within the analyzed ROIs are kept for further categorization."
- [methods] concentration filter: "The program then excludes metabolites with concentration limits below 5 μM (the lower limit of detection by most NMR instruments)"
- [readme] README sample types: "which are identified from complex biological samples (i.e., human serum, saliva, sweat, urine, CSF, and tissues) using the Human Metabolome Database (HMDB) as a reference platform."
- [discussion] user ROI variability: "using user-defined NMR spectral ROIs can lead to variable results obtained by different users"
