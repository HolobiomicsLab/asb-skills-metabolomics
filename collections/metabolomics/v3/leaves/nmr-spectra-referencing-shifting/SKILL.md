---
name: nmr-spectra-referencing-shifting
description: Reference for aligning chemical shifts of raw INADEQUATE spectra (via
  basic shifting) before PyINETA peak picking, when input spectra have not yet been
  referenced and picked_peaks.csv coordinates must fall on the correct chemical-shift
  axis.
when_to_use_negative:
- The INADEQUATE spectrum is already referenced (e.g. by prior processing in other
  NMR software) — skip re-referencing to avoid double shifts
- The analysis target is a non-INADEQUATE 1D or 2D NMR experiment; PyINETA's shifting
  is built for INADEQUATE spectra
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Python package performing reading and referencing (basic shifting) of INADEQUATE
    spectra as the first pipeline step before peak picking
  repo: https://github.com/rhilt13/PyINETA.git
- name: Python
  role: Runtime for run_pyineta.py, which executes the load/reference step of the
    INETA pipeline
- name: NMRbox
  role: Hosted environment where PyINETA (including its referencing module) is publicly
    installed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:15d23195cb2b18056de627ea7e74b4656a855e15fad1f74dc3f3654c76bc22fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA.git
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nmr-spectra-referencing-shifting

## Summary

Referencing of INADEQUATE NMR spectra by basic shifting using PyINETA, a prerequisite step so that downstream slice-wise peak picking and network filtering operate on correctly aligned chemical-shift coordinates.

## When to use

Reference for: aligning chemical shifts of raw INADEQUATE spectra (via basic shifting) before PyINETA peak picking, when input spectra have not yet been referenced and picked_peaks.csv coordinates must fall on the correct chemical-shift axis.

## When NOT to use

- The INADEQUATE spectrum is already referenced (e.g. by prior processing in other NMR software) — skip re-referencing to avoid double shifts
- The analysis target is a non-INADEQUATE 1D or 2D NMR experiment; PyINETA's shifting is built for INADEQUATE spectra

## Inputs

- Raw INADEQUATE NMR spectrum data file (2D 13C INADEQUATE FID/processed spectrum)
- PyINETA config file (config.ini) specifying load/reference options

## Outputs

- Referenced INADEQUATE spectrum on a corrected chemical-shift axis, ready for the pick step
- Intermediate loaded-spectrum files written to the PyINETA output directory

## How to apply

Load the raw INADEQUATE spectrum data file into PyINETA and apply basic shifting to reference the chemical shifts before any peak picking. In the PyINETA pipeline this referencing is part of the load step (run_pyineta.py supports step options such as 'load' and 'load+'), and it must precede the 'pick' step that sweeps picking levels from PPmax down to PPmin. Rationale: picked-peak coordinates are written to picked_peaks.csv and later filtered into networks of peaks from the same compound and matched against a simulated INADEQUATE database of metabolites, so an unreferenced spectrum would produce systematically shifted chemical-shift coordinates that fail database matching. Referencing is done automatically within PyINETA rather than with external NMR software, keeping the pipeline reproducible and parameter-tracked.

## Related tools

- **PyINETA** (Python package performing reading and referencing (basic shifting) of INADEQUATE spectra as the first pipeline step before peak picking) — https://github.com/rhilt13/PyINETA.git
- **Python** (Runtime for run_pyineta.py, which executes the load/reference step of the INETA pipeline)
- **NMRbox** (Hosted environment where PyINETA (including its referencing module) is publicly installed)

## Examples

```
python ../../run_pyineta.py -c config.ini -s load+ -o example1_output
```

## Evaluation signals

- Spectrum loads without errors and the load step completes before the pick step in run_pyineta.py output
- Chemical-shift axis of the referenced spectrum matches expected 13C chemical-shift ranges for the sample
- Downstream picked_peaks.csv entries have positions within the referenced spectrum's chemical-shift range
- Example1 runs reproduce the provided example1_default_output reference results

## Limitations

- Basic shifting is a simple referencing approach; spectra requiring more sophisticated alignment may need external correction
- Example2's larger input and temporary files exceed 100 Mb and must be obtained from the Edison lab, limiting out-of-the-box validation
- The paper's discussion states no explicit limitations or reproducibility gaps, so edge-case behavior of basic shifting is not documented

## Evidence

- [intro] Referencing via basic shifting precedes peak picking: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] Pipeline CLI with step options: [evidence span withheld — anchor status: not_found]
- [intro] Database matching depends on correct chemical shifts: "which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra."
- [abstract] Parameter tracking ensures reproducibility: "PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
