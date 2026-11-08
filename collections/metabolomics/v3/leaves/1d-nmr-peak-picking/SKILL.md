---
name: 1d-nmr-peak-picking
description: Use when you have a raw 2D 13C-JRES spectrum and INADEQUATE compound
  networks from prior PyINETA steps, and need picked 1D peaks to anchor compound annotation
  transfer to the JRES dimension.
when_to_use_negative:
- You only have an INADEQUATE spectrum with no complementary 13C-JRES experiment —
  the pipeline's picking on INADEQUATE spectra is a separate step, not this JRES projection
  picking
- A peak-picked 1D spectrum or an existing peak-to-compound mapping table is already
  available, so re-picking would only duplicate work
- You need annotation of compounds with no matched INADEQUATE network — unmatched
  peaks are left unassigned by design
edam_operation: http://edamontology.org/operation_3256
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: Python package whose picking module reads the raw 13C-JRES spectrum, builds
    the 1D projection, and performs peak picking for the INADEQUATE-JRES integration
    step
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for running the PyINETA pipeline via run_pyineta.py with a config
    file
- name: NMRbox
  role: Hosted environment where PyINETA is installed and publicly available for running
    the pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:e9b3e4067dd6d5c8622dbad360769aa47f1facd7acf2379cf5106b9f7180ce82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3256
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# 1D NMR Peak Picking

## Summary

Peak picking on a 1D projection of a 13C-JRES spectrum within PyINETA's INADEQUATE-to-JRES integration step, so that picked 1D peaks can be matched to INADEQUATE compound networks and annotated with compound labels.

## When to use

Use when you have a raw 2D 13C-JRES spectrum and INADEQUATE compound networks from prior PyINETA steps, and need picked 1D peaks to anchor compound annotation transfer to the JRES dimension.

## When NOT to use

- You only have an INADEQUATE spectrum with no complementary 13C-JRES experiment — the pipeline's picking on INADEQUATE spectra is a separate step, not this JRES projection picking
- A peak-picked 1D spectrum or an existing peak-to-compound mapping table is already available, so re-picking would only duplicate work
- You need annotation of compounds with no matched INADEQUATE network — unmatched peaks are left unassigned by design

## Inputs

- Raw 2D 13C-JRES spectrum file (NMR raw data readable by PyINETA's picking module)
- INADEQUATE compound networks from preceding PyINETA steps (cluster/find/match outputs)
- config.ini with PyINETA parameters including Peak_Width_1D and Intensity_threshold_1D

## Outputs

- List of peaks picked from the 1D projection of the 13C-JRES spectrum
- jres_compound_mapping.csv: peak-to-compound mapping table assigning INADEQUATE-derived compound labels to JRES peaks

## How to apply

Read the raw 13C-JRES spectrum file with PyINETA's picking module, then project the 2D JRES spectrum to a 1D spectrum. Pick peaks in the 1D projection using the Peak_Width_1D area parameter, which controls the width of the region treated as a peak. Match the picked 1D peaks to the INADEQUATE compound networks found in the preceding PyINETA steps (load, pick, cluster, find, match) using the Intensity_threshold_1D filter, then transfer the matched compound labels onto the corresponding JRES peaks and write the peak-to-compound mapping as jres_compound_mapping.csv. The choice of Peak_Width_1D and Intensity_threshold_1D governs the sensitivity/specificity trade-off: too permissive settings yield peaks that cannot be consistently assigned to matched networks.

## Related tools

- **PyINETA** (Python package whose picking module reads the raw 13C-JRES spectrum, builds the 1D projection, and performs peak picking for the INADEQUATE-JRES integration step) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for running the PyINETA pipeline via run_pyineta.py with a config file)
- **NMRbox** (Hosted environment where PyINETA is installed and publicly available for running the pipeline)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Every picked JRES peak is assigned a compound label consistent with the INADEQUATE networks matched at the given Intensity_threshold_1D, with unmatched peaks left unassigned
- The jres_compound_mapping.csv table covers all picked 1D peaks and maps each to a compound or leaves it explicitly unassigned
- Pipeline outputs can be cross-checked against the repository's example1_default_output reference results
- PyINETA's recorded pipeline parameters (Peak_Width_1D, Intensity_threshold_1D) reproduce the annotation, supporting transparency of compound annotation

## Limitations

- Unmatched peaks are left unassigned, so compounds absent from the simulated INADEQUATE database or below the Intensity_threshold_1D filter will not be annotated
- Example 2's input and temporary files exceed 100 Mb and raw files must be requested from the Edison lab, limiting out-of-the-box validation on large datasets
- Results depend on tuning Peak_Width_1D and Intensity_threshold_1D; mis-set parameters yield peaks that cannot be matched to INADEQUATE networks

## Evidence

- [intro] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites"
- [abstract] integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment: "conducts database search, and integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment"
- [readme] You can cross-check the results with the provided output in folder `example1_default_output`.: [evidence span withheld — anchor status: external_doc]
