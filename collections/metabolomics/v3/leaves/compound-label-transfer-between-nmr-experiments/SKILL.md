---
name: compound-label-transfer-between-nmr-experiments
description: Use when raw 13C-JRES spectra from 13C-labeled samples must be annotated
  using compound networks already identified from INADEQUATE spectra in preceding
  PyINETA steps.
when_to_use_negative:
- Only 13C-JRES data is available with no INADEQUATE networks from prior PyINETA steps
  — there is no source of compound labels to transfer.
- JRES peaks already carry compound annotations from another source, making network-based
  transfer redundant.
- Samples cannot be 13C-labeled, since the INADEQUATE networks that supply the labels
  rely on 13C-13C connectivity.
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3379
tools:
- name: PyINETA
  role: Python package whose picking/integration module reads the raw JRES spectrum,
    projects it to 1D, picks peaks, and transfers INADEQUATE compound labels
  repo: https://github.com/rhilt13/PyINETA.git
- name: Python
  role: Execution environment for running the run_pyineta.py pipeline script with
    config.ini
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:da5966cc2dccb50049294371373f277b4a102e129ca6a0c2fcd83267411ec054
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA.git
  tools:
  - PyINETA
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# compound-label-transfer-between-nmr-experiments

## Summary

PyINETA's optional integration step transfers compound annotations from INADEQUATE peak networks onto peaks of the complementary 13C-JRES experiment, so structural identities established from 13C-13C connectivity also label J-resolved peaks.

## When to use

Use when raw 13C-JRES spectra from 13C-labeled samples must be annotated using compound networks already identified from INADEQUATE spectra in preceding PyINETA steps.

## When NOT to use

- Only 13C-JRES data is available with no INADEQUATE networks from prior PyINETA steps — there is no source of compound labels to transfer.
- JRES peaks already carry compound annotations from another source, making network-based transfer redundant.
- Samples cannot be 13C-labeled, since the INADEQUATE networks that supply the labels rely on 13C-13C connectivity.

## Inputs

- Raw 13C-JRES spectrum file (2D NMR spectrum from 13C-labeled samples)
- INADEQUATE compound networks identified in preceding PyINETA steps (cluster, find, match outputs)
- PyINETA config file (config.ini) with Peak_Width_1D and Intensity_threshold_1D parameters

## Outputs

- jres_compound_mapping.csv — peak-to-compound mapping table assigning compound labels to JRES peaks
- Picked 1D peaks from the JRES projection

## How to apply

Run PyINETA's integration/picking module on the raw 13C-JRES spectrum and project the 2D JRES spectrum to a 1D spectrum. Pick peaks in the 1D projection using the Peak_Width_1D area parameter, then match the picked 1D peaks to the INADEQUATE compound networks found earlier by PyINETA, applying the Intensity_threshold_1D filter as the match decision point. Transfer the matched INADEQUATE-based compound labels onto the corresponding JRES peaks and write the peak-to-compound mapping table as jres_compound_mapping.csv. This works because INADEQUATE provides the structural networks (filtered peaks matched to a simulated INADEQUATE database of metabolites) while 13C-JRES provides complementary chemical-shift resolution for the same 13C-labeled metabolites.

## Related tools

- **PyINETA** (Python package whose picking/integration module reads the raw JRES spectrum, projects it to 1D, picks peaks, and transfers INADEQUATE compound labels) — https://github.com/rhilt13/PyINETA.git
- **Python** (Execution environment for running the run_pyineta.py pipeline script with config.ini)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Every picked JRES peak is assigned a compound label consistent with the INADEQUATE networks matched at the given Intensity_threshold_1D; unmatched peaks are left unassigned.
- jres_compound_mapping.csv exists and contains the expected peak-to-compound mapping for the run parameters.
- Results match the reference outputs shipped with the repository (e.g. example1_default_output).
- PyINETA's pipeline parameter/output tracking records the Peak_Width_1D and Intensity_threshold_1D values used, ensuring annotation transparency.
- In the model mouse study, transferred labels recapitulated the known tissue-specific enrichment of compounds in liver, spleen, pancreas, muscle, or lung.

## Limitations

- Large example inputs/outputs (example2, >100 Mb) are not shipped with the repository; raw files must be requested from the Edison lab.
- The integration depends on quality of the preceding INADEQUATE network identification; peaks not matching any network at Intensity_threshold_1D remain unassigned.
- INADEQUATE spectra are read/referenced with basic shifting only, so mis-referenced input spectra can propagate errors into the transferred labels.

## Evidence

- [abstract] integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment ( 13 C-JRES): "conducts database search, and integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment"
- [abstract] Our evaluation of PyINETA using a model mouse study showed that our pipeline successfully integrated INADEQUATE and 13 C-JRES.: "Our evaluation of PyINETA using a model mouse study showed that our pipeline successfully integrated INADEQUATE and 13 C-JRES."
- [intro] filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites"
- [readme] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
