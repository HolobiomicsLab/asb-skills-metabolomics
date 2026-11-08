---
name: spin-system-network-construction
description: Use when analyzing picked peaks from an INADEQUATE NMR spectrum and you
  need to reconstruct the 13C spin-system networks of a metabolite before compound
  matching.
when_to_use_negative:
- Input spectra are not INADEQUATE (e.g. single-dimension 13C or 1H spectra) - the
  network logic relies on double-quantum transition structure and is not meaningful
  for non-INADEQUATE peak lists
- Peak picking has not yet been performed - the network step requires the picked peak
  list as input, not raw spectra
- You already have a curated connectivity table and only need compound identification
  - then the downstream matching step is the right entry point, not network construction
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: 'Implements the network-construction (cluster/find) step: screens picked peaks
    with DQT/SumXY/SDT thresholds into horizontal networks and links them vertically
    via the CST threshold'
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Hosted environment in which PyINETA, including the network-construction step,
    is publicly available and can be run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:bb3ca0f11411d19170e94a32b38757fc10dadc9a7fda802bccec05cf9caeef82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spin-system-network-construction

## Summary

PyINETA method for grouping picked INADEQUATE peaks into horizontal carbon networks (via DQT, SumXY, SDT thresholds) and then chaining those networks vertically by shared-carbon CST similarity, reconstructing spin systems from 13C INADEQUATE NMR data.

## When to use

Use when analyzing picked peaks from an INADEQUATE NMR spectrum and you need to reconstruct the 13C spin-system networks of a metabolite before compound matching.

## When NOT to use

- Input spectra are not INADEQUATE (e.g. single-dimension 13C or 1H spectra) - the network logic relies on double-quantum transition structure and is not meaningful for non-INADEQUATE peak lists
- Peak picking has not yet been performed - the network step requires the picked peak list as input, not raw spectra
- You already have a curated connectivity table and only need compound identification - then the downstream matching step is the right entry point, not network construction

## Inputs

- picked peak list CSV (peak chemical shifts and intensities from the PyINETA picking module)
- INADEQUATE NMR spectrum

## Outputs

- network_table.csv listing peak pairs, network labels and the satisfied criterion (horizontal/vertical)
- constructed horizontal and vertical spin-system networks used for downstream matching to the simulated INADEQUATE database

## How to apply

Load the peak list CSV produced by the PyINETA picking module into the PyINETA network-construction step (run_pyineta.py -s cluster). Screen all peak pairs for horizontal connectivity using the DQT threshold on double-quantum shift differences plus SumXY (sum of direct-dimension chemical shifts vs double-quantum mean) and SDT (deviation from the Y = 2X line), retaining pairs whose quadrature transitions and summed shifts satisfy all three thresholds. Group the retained horizontal pairs into horizontal networks representing each 13C spin system. Then link horizontal networks vertically into chains whenever a shared carbon has a direct-dimension chemical shift difference below the CST threshold, and write the network_table.csv output with columns for the two peaks, the network labels and the criterion (horizontal or vertical) that each listed pair satisfied. PyINETA automatically tracks all pipeline parameters and outputs, so thresholds used for every grouping decision remain transparent and reproducible.

## Related tools

- **PyINETA** (Implements the network-construction (cluster/find) step: screens picked peaks with DQT/SumXY/SDT thresholds into horizontal networks and links them vertically via the CST threshold) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Hosted environment in which PyINETA, including the network-construction step, is publicly available and can be run)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- network_table.csv is well-formed and each listed peak pair satisfies at least one configured threshold (DQT/SumXY/SDT or CST) as recorded by PyINETA's parameter tracking
- Horizontal networks each correspond to a single 13C spin system, with all member peak pairs passing DQT, SumXY and SDT criteria
- Vertical links connect only networks that share a carbon below the CST direct-dimension shift difference threshold
- The constructed networks can be matched to the simulated INADEQUATE database to identify the intended metabolite, consistent with the article's pipeline flow
- Threshold values and grouping outputs are logged in PyINETA's automatic pipeline parameter tracking, allowing audit of annotation transparency

## Limitations

- Network construction quality is bounded by the accuracy of the upstream peak picking step; false or missed peaks propagate into the networks
- The skill is specific to INADEQUATE spectra; it cannot be extended to non-INADEQUATE experiments without redefinition
- Large example inputs (>100 Mb, example2) exceed what can be shared directly in the repository, so reproducibility for very large datasets may require contacting the Edison lab

## Evidence

- [other] PyINETA builds INADEQUATE networks by screening peaks for horizontal pairs using thresholds DQT (double-quantum shift difference), SumXY (direct-dimension sum vs double-quantum mean), and SDT (deviation from the Y = 2X line): [evidence span withheld — anchor status: not_found]
- [intro] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra.: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [abstract] PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics.: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [readme] README states the pipeline CLI and step control used to run the network-construction step.: [evidence span withheld — anchor status: external_doc]
