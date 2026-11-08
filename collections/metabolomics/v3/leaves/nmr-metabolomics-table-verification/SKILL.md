---
name: nmr-metabolomics-table-verification
description: 'Reference for validating PyINETA INADEQUATE network results against
  a published table: compare constructed-network, matched-network, and compound counts
  to the paper''s reported values.'
when_to_use_negative:
- The input is a 13C-JRES spectrum alone rather than the INADEQUATE spectrum that
  PyINETA's pipeline expects
- The goal is to analyze a spectrum from a different sample or tissue rather than
  the liver Sample ID 30 spectrum used for the reference counts in Table 1 and Supporting
  S11/S13
edam_operation: http://edamontology.org/operation_3795
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Python-based INADEQUATE network analysis platform used to construct networks
    and match them to a simulated INADEQUATE database of metabolites
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runs the pipeline entry point (run_pyineta.py) with options
- name: NMRbox
  role: Public platform where PyINETA is installed and available
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:a53f528e90b4279d8bafa0e97b7725f576caf9e47d2081335cffa2d3d0835f46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3795
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nmr-metabolomics-table-verification

## Summary

This skill covers verifying that a PyINETA run reproduces the paper's reported counts for the liver Sample ID 30 INADEQUATE spectrum — 67 constructed networks, 46 matched, 21 unmatched, and 21 compounds in the Matches table — by comparing pipeline outputs against Table 1 and Supporting pages S11/S13.

## When to use

Reference for validating PyINETA INADEQUATE network results against a published table: compare constructed-network, matched-network, and compound counts to the paper's reported values.

## When NOT to use

- The input is a 13C-JRES spectrum alone rather than the INADEQUATE spectrum that PyINETA's pipeline expects
- The goal is to analyze a spectrum from a different sample or tissue rather than the liver Sample ID 30 spectrum used for the reference counts in Table 1 and Supporting S11/S13

## Inputs

- Deposited liver Sample ID 30 INADEQUATE spectrum file
- PyINETA 2.0.0 package (version 2.0.0)
- Paper's Table 1 and Supporting pages S11 and S13 as reference

## Outputs

- Constructed-network count (67)
- Networks with database match (46) and without matches (21)
- Compounds listed in the Matches table (21)

## How to apply

Obtain the deposited liver Sample ID 30 INADEQUATE spectrum and open it in PyINETA 2.0.0, which automatically reads and references (basic shifting) the INADEQUATE spectra, picks peaks, filters picked peaks into networks, matches networks to a simulated INADEQUATE database of metabolites, and integrates the data with the complementary 13C-JRES experiment. The pipeline keeps track of all parameters and outputs; collect the constructed-network count, the subset of networks with at least one database candidate match, the subset without matches, and the compounds listed in the Matches table. Confirm these equal the reported values of 67 constructed networks, 46 networks with a database entry match, 21 networks without matches, and 21 identified compounds in the Matches table. Finally cross-check the reproduced output against the paper's Table 1 and the Supporting pages S11 (networks) and S13 (Matches table) to verify agreement of the matched and unmatched network listings and the identified compound list.

## Related tools

- **PyINETA** (Python-based INADEQUATE network analysis platform used to construct networks and match them to a simulated INADEQUATE database of metabolites) — https://github.com/edisonomics/PyINETA
- **Python** (Runs the pipeline entry point (run_pyineta.py) with options)
- **NMRbox** (Public platform where PyINETA is installed and available)

## Examples

```
python <path_to_pyineta_repo>/run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Constructed-network count equals 67
- Networks matched to at least one database candidate compound equals 46
- Networks without any database match equals 21
- Compounds listed in the Matches table equals 21
- Reproduced output agrees with the matched and unmatched network listings on Supporting page S11 and the identified compound list on Supporting page S13

## Limitations

- PyINETA keeps track of all pipeline parameters and outputs, ensuring transparency of annotation in metabolomics; no explicit limitations or reproducibility gaps were stated in the paper's discussion.
- The README notes that for example2 the input and temporary files are larger than 100 Mb and must be obtained by contacting the Edison lab; a config file and output files are provided for reference.

## Evidence

- [other] PyINETA constructed 67 INADEQUATE networks from the liver sample, matched 46 networks to at least one database candidate compound (21 unmatched), and identified 21 compounds in the Matches table.: [evidence span withheld — anchor status: not_found]
- [intro] PyINETA is designed to filter picked peaks into networks and match them to a simulated INADEQUATE database to identify metabolites in the query INADEQUATE spectra.: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [readme] The README documents how to run the pipeline and lists all outputs and configuration options.: "pyINETA is a Python package for analyzing INADEQUATE NMR spectra."
- [abstract] PyINETA keeps track of all pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics.: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
