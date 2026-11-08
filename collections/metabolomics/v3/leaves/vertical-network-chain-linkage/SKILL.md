---
name: vertical-network-chain-linkage
description: 'Use when building vertical chains from horizontal 13C INADEQUATE networks:
  after horizontal pair screening (DQT/SumXY/SDT) has produced per-spin-system networks
  and shared-carbon linkage is needed.'
when_to_use_negative:
- The peak list has not yet been screened for horizontal (DQT) pairs — vertical linkage
  assumes horizontal networks already exist
- The goal is metabolite identification against the simulated INADEQUATE database
  rather than constructing connectivity networks
- Working with 13C-JRES spectra alone, which provide no double-quantum connectivity
  for network construction
edam_operation: http://edamontology.org/operation_3080
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: PyINETA
  role: Executes the vertical network-chain linkage step by evaluating shared carbons
    between horizontal networks against the CST threshold and writing the resulting
    network_table.csv
  repo: https://github.com/rhilt13/PyINETA.git
- name: Python
  role: Runtime environment in which the PyINETA pipeline (including the vertical
    linkage module) is invoked via run_pyineta.py
- name: NMRbox
  role: Public execution platform where PyINETA, including its network-linkage module,
    is installed and can be run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:b55c61e1cbdc80ace4febc99fe6e9d1b9c56bddaff96d7e30765912d06c6b784
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA.git
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3080
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# vertical-network-chain-linkage

## Summary

Link horizontal INADEQUATE peak networks into vertical chains by identifying networks that share a common carbon whose direct-dimension chemical shift difference falls below the CST threshold, as implemented in PyINETA's network-construction module.

## When to use

Use when building vertical chains from horizontal 13C INADEQUATE networks: after horizontal pair screening (DQT/SumXY/SDT) has produced per-spin-system networks and shared-carbon linkage is needed.

## When NOT to use

- The peak list has not yet been screened for horizontal (DQT) pairs — vertical linkage assumes horizontal networks already exist
- The goal is metabolite identification against the simulated INADEQUATE database rather than constructing connectivity networks
- Working with 13C-JRES spectra alone, which provide no double-quantum connectivity for network construction

## Inputs

- Picked INADEQUATE peak list (CSV of peak chemical shifts and intensities) from the PyINETA picking module
- Horizontal networks grouped per 13C spin system from the horizontal-pairing step
- config.ini containing the CST threshold and other pipeline parameters

## Outputs

- network_table.csv with columns for the two peaks, network labels, and the satisfied criterion (horizontal/vertical)
- Pipeline parameter log recording the CST threshold used for each vertical linkage decision

## How to apply

After horizontal screening has grouped picked INADEQUATE peak pairs into horizontal networks per 13C spin system, evaluate candidate network pairs for vertical linkage via shared carbons. Compute the direct-dimension chemical shift difference for carbons common to both networks and accept the vertical link only when that difference is below the CST threshold configured in the run's config file. PyINETA automatically tracks all pipeline parameters and outputs, so the CST value used for each accepted or rejected link is recorded for transparency. Write the accepted horizontal and vertical connections to network_table.csv, where each row records the two peaks involved, their network labels, and which criterion (horizontal or vertical) was satisfied.

## Related tools

- **PyINETA** (Executes the vertical network-chain linkage step by evaluating shared carbons between horizontal networks against the CST threshold and writing the resulting network_table.csv) — https://github.com/rhilt13/PyINETA.git
- **Python** (Runtime environment in which the PyINETA pipeline (including the vertical linkage module) is invoked via run_pyineta.py)
- **NMRbox** (Public execution platform where PyINETA, including its network-linkage module, is installed and can be run)

## Examples

```
cd <path_to_pyineta_repo>/example/example1/ && python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Every listed peak pair in network_table.csv satisfies at least one of the configured DQT/SumXY/SDT (horizontal) or CST (vertical) thresholds as recorded by PyINETA's parameter tracking
- network_table.csv is a well-formed CSV with columns for the two peaks, network labels, and the satisfied criterion
- Each accepted vertical link corresponds to a carbon whose direct-dimension chemical-shift difference is below the CST threshold recorded in the run parameters
- Running the provided example1 with the default config reproduces the outputs in example1_default_output

## Limitations

- PyINETA is designed for 13C INADEQUATE spectra of 13C-labeled systems; unlabeled or very dilute samples will not yield reliable vertical chains
- Vertical linkage quality depends on the horizontal-network step: poorly grouped horizontal networks propagate into fragmented or incorrect vertical chains

## Evidence

- [methods] PyINETA builds INADEQUATE networks by screening peaks for horizontal pairs using thresholds DQT (double-quantum shift difference), SumXY (direct-dimension sum vs double-quantum mean), and SDT (deviation from the Y = 2X line), and links sequential horizontal networks vertically when a shared carbon has a direct-dimension chemical shift difference below the CST threshold.: [evidence span withheld — anchor status: not_found]
- [abstract] Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics.: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation"
- [readme] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra.: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound"
- [methods] python run_pyineta.py <options>: "python run_pyineta.py <options>"
