---
name: threshold-based-peak-pair-filtering
description: Use when you have a PyINETA peak list from a 13C INADEQUATE spectrum
  and need to group peaks into same-compound networks before database matching.
when_to_use_negative:
- Input is a raw 2D INADEQUATE spectrum rather than an already-picked peak list —
  run the PyINETA picking step first.
- You need simulated INADEQUATE database matching or metabolite identification; this
  skill only constructs networks, identification is a later pipeline step.
- The analysis targets unlabeled samples where 13C-13C correlations are absent.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3336
tools:
- name: PyINETA
  role: Python package that implements the DQT/SumXY/SDT/CST threshold screening and
    network construction
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime environment invoked via run_pyineta.py with a config.ini
- name: NMRbox
  role: Public installation platform where PyINETA (and this filtering step) can be
    run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:ba1955359154f5b60cc897610ab2f7c21f4431b24089dd223d71012c6778e7a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# threshold-based-peak-pair-filtering

## Summary

Screens picked INADEQUATE peak pairs with explicit thresholds (DQT, SumXY, SDT, CST) to build horizontal networks of same-compound peaks and link them vertically into full carbon skeletons in PyINETA.

## When to use

Use when you have a PyINETA peak list from a 13C INADEQUATE spectrum and need to group peaks into same-compound networks before database matching.

## When NOT to use

- Input is a raw 2D INADEQUATE spectrum rather than an already-picked peak list — run the PyINETA picking step first.
- You need simulated INADEQUATE database matching or metabolite identification; this skill only constructs networks, identification is a later pipeline step.
- The analysis targets unlabeled samples where 13C-13C correlations are absent.

## Inputs

- PyINETA-picked peak list (CSV of INADEQUATE peak chemical shifts and intensities)
- PyINETA config.ini with threshold parameters (DQT, SumXY, SDT, CST)

## Outputs

- network_table.csv listing peak pairs with horizontal/vertical labels and the satisfied criterion
- Networks of peaks used for downstream simulated-INADEQUATE database matching

## How to apply

After PyINETA peak picking, screen all peak pairs for horizontal (DQT) connectivity: keep pairs whose differences in quadrature transitions and summed direct-dimension shifts satisfy the DQT, SumXY, and SDT thresholds (SDT captures deviation from the Y = 2X double-quantum line). Group retained pairs into horizontal networks per 13C spin system. Then link sequential horizontal networks vertically when a shared carbon's direct-dimension chemical-shift difference is below the CST threshold. PyINETA automatically tracks all pipeline parameters and outputs, so every retained pair's criterion (horizontal vs vertical) is recorded for downstream database matching.

## Related tools

- **PyINETA** (Python package that implements the DQT/SumXY/SDT/CST threshold screening and network construction) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime environment invoked via run_pyineta.py with a config.ini)
- **NMRbox** (Public installation platform where PyINETA (and this filtering step) can be run)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- network_table.csv is a well-formed CSV in which every listed peak pair satisfies at least one of DQT/SumXY/SDT or CST as recorded by PyINETA's parameter tracking.
- Each retained horizontal pair must fall on/near the Y = 2X double-quantum line within the SDT threshold.
- Vertically linked networks share a carbon whose direct-dimension chemical-shift difference is below the CST threshold.
- Resulting networks, when matched against the simulated INADEQUATE metabolite database, identify the expected compounds (verify with the provided example1_default_output).

## Limitations

- Thresholds DQT, SumXY, SDT, and CST must be tuned per dataset; overly permissive settings produce spurious network links.
- example2's raw input files exceed 100 Mb and are not distributed with the repo — contact the Edison lab if needed.
- PyINETA keeps track of all pipeline parameters and outputs, so mis-set thresholds will be visible in the tracking logs but may require rerunning.

## Evidence

- [readme] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound"
- [readme] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [abstract] Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [readme] python <path_to_pyineta_repo>/run_pyineta.py -c config.ini <other options>: [evidence span withheld — anchor status: external_doc]
- [intro] clustering/finding networks of picked peaks to identify metabolites: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
