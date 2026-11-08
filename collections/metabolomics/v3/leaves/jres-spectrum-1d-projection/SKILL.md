---
name: jres-spectrum-1d-projection
description: Use when a 2D 13C-JRES spectrum must be reduced to a 1D projection so
  its peaks can be picked and annotated with compound identities inferred from INADEQUATE
  networks in a PyINETA run.
when_to_use_negative:
- The run is INADEQUATE-only and no complementary 13C-JRES spectrum was acquired —
  use the standard load/pick/cluster/find/match steps instead
- No INADEQUATE networks have been established in preceding PyINETA steps, so there
  is no compound information to transfer to JRES peaks
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Performs the JRES 1D projection, 1D peak picking, matching to INADEQUATE networks,
    and compound-label transfer in the integration step
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime environment for the PyINETA package (run_pyineta.py entry point)
- name: NMRbox
  role: Hosted platform where PyINETA (including the JRES integration step) is installed
    and publicly available
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:e0030c85f70c3035ba60e8bd0841c710ae6c5645746391a1a5e1a50b20c80fd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# jres-spectrum-1d-projection

## Summary

An optional PyINETA integration step that reads a raw 2D 13C-JRES NMR spectrum, projects it to a 1D spectrum, picks peaks, and transfers compound annotations from previously matched INADEQUATE networks onto the JRES peaks.

## When to use

Use when a 2D 13C-JRES spectrum must be reduced to a 1D projection so its peaks can be picked and annotated with compound identities inferred from INADEQUATE networks in a PyINETA run.

## When NOT to use

- The run is INADEQUATE-only and no complementary 13C-JRES spectrum was acquired — use the standard load/pick/cluster/find/match steps instead
- No INADEQUATE networks have been established in preceding PyINETA steps, so there is no compound information to transfer to JRES peaks

## Inputs

- Raw 2D 13C-JRES NMR spectrum file
- INADEQUATE compound networks produced by preceding PyINETA steps (pick, cluster, find, match)
- config.ini with Peak_Width_1D and Intensity_threshold_1D parameters

## Outputs

- 1D projection of the 13C-JRES spectrum
- Picked 1D peak list from the projection
- jres_compound_mapping.csv — peak-to-compound mapping table

## How to apply

Run the optional integration stage of the PyINETA pipeline (steps such as pick+/match+ in run_pyineta.py, controlled by a config.ini). First read the raw 13C-JRES spectrum with PyINETA's picking module, then project the 2D JRES spectrum to a 1D spectrum. Pick peaks in the 1D projection using the Peak_Width_1D area parameter, then match those picked 1D peaks against the INADEQUATE compound networks found in the preceding PyINETA steps, applying the Intensity_threshold_1D filter. Transfer the matched INADEQUATE-based compound labels onto the corresponding JRES peaks, leaving peaks with no network match unassigned, and write the peak-to-compound mapping table as jres_compound_mapping.csv. The rationale is that the 13C-JRES experiment is complementary to INADEQUATE, so projecting it to 1D lets JRES peaks inherit compound identities established from the INADEQUATE network matching.

## Related tools

- **PyINETA** (Performs the JRES 1D projection, 1D peak picking, matching to INADEQUATE networks, and compound-label transfer in the integration step) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime environment for the PyINETA package (run_pyineta.py entry point))
- **NMRbox** (Hosted platform where PyINETA (including the JRES integration step) is installed and publicly available)

## Examples

```
python run_pyineta.py -c config.ini -s match+ -o example1_output
```

## Evaluation signals

- Every picked JRES peak is either assigned a compound label consistent with the INADEQUATE networks matched at the given Intensity_threshold_1D or left unassigned
- The peak-to-compound mapping table is written as jres_compound_mapping.csv with a valid mapping entry for each 1D-projected peak
- The 1D projection's picked peaks correspond to features visible in the original 2D JRES spectrum at the Peak_Width_1D area setting
- Running the provided example reproduces outputs matching example1_default_output

## Limitations

- JRES peaks with no matching INADEQUATE network at the chosen Intensity_threshold_1D remain unassigned
- Annotation quality depends entirely on the compound networks established in the preceding INADEQUATE pick/cluster/find/match steps
- For larger datasets (e.g. example2), input and temporary files exceed 100 Mb and raw files must be requested from the Edison lab

## Evidence

- [other] The integration step reads the raw 13C-JRES spectrum, creates a 1D projection, picks peaks from it, matches those peaks to INADEQUATE networks, and transfers the INADEQUATE-based compound information to the JRES peaks.: [evidence span withheld — anchor status: not_found]
- [abstract] Our evaluation of PyINETA using a model mouse study showed that our pipeline successfully integrated INADEQUATE and 13 C-JRES.: "Our evaluation of PyINETA using a model mouse study showed that our pipeline successfully integrated INADEQUATE and 13 C-JRES."
- [intro] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra.: "filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites"
- [readme] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] Optional: Specify which steps you want to run. Can be one of {all,load,pick,cluster,find,match,plot,summary,singleplot,load+,pick+,cluster+,find+,match+}.: [evidence span withheld — anchor status: external_doc]
