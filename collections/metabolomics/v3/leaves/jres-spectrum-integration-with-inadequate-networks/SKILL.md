---
name: jres-spectrum-integration-with-inadequate-networks
description: Use when a deposited 13C-JRES spectrum must be integrated against previously
  constructed INADEQUATE networks (Match output) to determine how many networks have
  complementary JRES peaks, e.g. reproducing a reported count such as 65 of 67 networks
  in a 13C-labeled mouse liver sample.
when_to_use_negative:
- Input spectra are not INADEQUATE spectra — pyINETA is designed for INADEQUATE NMR
  spectra, not generic 1D 1H NMR or unrelated 2D experiments
- 'No upstream INADEQUATE networks exist yet: the integration step requires the Match
  output of constructed peak networks, so the load/pick/cluster/find steps must run
  first (or use -s match+ to resume from matching)'
- The goal is de novo compound discovery rather than validating a reported integration
  result for an existing deposited spectrum
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: PyINETA
  role: Python package that reads and references INADEQUATE spectra, constructs peak
    networks, and runs the INADEQUATE-JRES integration producing the MatchJres output
  repo: https://github.com/rhilt13/PyINETA.git
- name: Python
  role: Runtime used to execute the run_pyineta.py pipeline script with a config file
- name: NMRbox
  role: Public platform on which PyINETA (v2.0.0) is installed and publicly available
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:0343409c83c002f0c5f789e6bac478323eda63eca25ac415b47faeaef9a179c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA.git
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# jres-spectrum-integration-with-inadequate-networks

## Summary

Integrates a complementary 13C-JRES NMR spectrum with upstream INADEQUATE peak networks produced by PyINETA, generating a MatchJres table that flags which constructed networks have corresponding JRES peaks. This is the JRES-integration step of PyINETA's INADEQUATE-JRES annotation pipeline for 13C-enriched metabolomics samples.

## When to use

Use when a deposited 13C-JRES spectrum must be integrated against previously constructed INADEQUATE networks (Match output) to determine how many networks have complementary JRES peaks, e.g. reproducing a reported count such as 65 of 67 networks in a 13C-labeled mouse liver sample.

## When NOT to use

- Input spectra are not INADEQUATE spectra — pyINETA is designed for INADEQUATE NMR spectra, not generic 1D 1H NMR or unrelated 2D experiments
- No upstream INADEQUATE networks exist yet: the integration step requires the Match output of constructed peak networks, so the load/pick/cluster/find steps must run first (or use -s match+ to resume from matching)
- The goal is de novo compound discovery rather than validating a reported integration result for an existing deposited spectrum

## Inputs

- Deposited 13C-JRES NMR spectrum for the sample (e.g. liver sample 30)
- Upstream PyINETA Match output: constructed INADEQUATE networks from the find/match steps
- PyINETA config file (config.ini) with pipeline options and parameters

## Outputs

- MatchJres output table with one row per INADEQUATE network flagging presence of complementary JRES peaks
- Count of networks with JRES peaks (expected 65 of 67 for the liver sample)
- Pipeline parameters and output logs tracked automatically by PyINETA

## How to apply

Load the deposited JRES spectrum for the sample together with the upstream Match output containing the constructed INADEQUATE networks, then run the PyINETA pipeline (run_pyineta.py driven by a config.ini with required -c option) restricted to or resumed from the matching step (steps such as match, match+, or all). PyINETA attaches JRES peak information to each INADEQUATE network, matching complementary peaks against the simulated INADEQUATE database of metabolites it uses for compound identification, and automatically tracks all pipeline parameters and outputs. The run writes a MatchJres output table with one row per constructed network (67 networks in the liver-sample reproduction) indicating whether complementary JRES peaks are present. Count the networks flagged with JRES peaks and compare against the reported integration result — the liver sample yields 65 of 67. Persist the MatchJres table as the single canonical output file in the designated output directory (-o OUTDIR).

## Related tools

- **PyINETA** (Python package that reads and references INADEQUATE spectra, constructs peak networks, and runs the INADEQUATE-JRES integration producing the MatchJres output) — https://github.com/rhilt13/PyINETA.git
- **Python** (Runtime used to execute the run_pyineta.py pipeline script with a config file)
- **NMRbox** (Public platform on which PyINETA (v2.0.0) is installed and publicly available)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- MatchJres table exists in the specified OUTDIR and has the expected per-network rows (67 networks for the liver sample)
- Number of networks flagged with JRES peaks equals the reported value: 65 of 67
- Per-network JRES flags correspond to complementary JRES peaks in the deposited 13C-JRES spectrum regions of each INADEQUATE network
- All pipeline parameters and outputs are tracked by PyINETA, ensuring the transparency and reproducibility of the annotation
- For the bundled example runs, results in example1_output cross-check against the provided example1_default_output folder

## Limitations

- Large-input runs (e.g. example2) exceed 100 Mb for input and temporary files; raw files must be requested from the Edison lab, so the reproduction may be limited to deposited spectra and provided outputs/configs
- Referencing uses only basic shifting, which may be insufficient for spectra requiring more sophisticated alignment
- Peak-network filtering ideally assumes networks come from the same compound; networks that are not compound-pure may degrade JRES integration and matching

## Evidence

- [abstract] conducts database search, and integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment: "conducts database search, and integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment"
- [intro] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound"
- [methods] python run_pyineta.py <options>: "python run_pyineta.py <options>"
- [readme] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
