---
name: metabolomics-pipeline-execution-and-verification
description: Use when reproducing a published NMR metabolomics result from a deposited
  INADEQUATE and/or 13C-JRES spectrum of a labeled sample, where the claim is a concrete
  per-network count (e.g. 65 of 67 networks with JRES peaks) that must be re-derived.
when_to_use_negative:
- When the input is unlabeled (natural-abundance) 1H NMR data rather than 13C INADEQUATE
  spectra — PyINETA is designed for INADEQUATE NMR spectra of labeled systems
- When only a final summary table is available and the deposited raw spectra needed
  for load/pick/cluster/find/match steps are missing (as with example2, where raw
  files exceed 100 Mb and must be requested from the Edison lab)
- When the goal is generic metabolite annotation without needing structural fragment/network
  information from INADEQUATE
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: Python package executing the INADEQUATE reading/referencing, peak picking,
    network filtering, database matching, and INADEQUATE-13C-JRES integration steps
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for run_pyineta.py CLI driver and the pipeline scripts
- name: NMRbox
  role: Hosted environment where PyINETA is installed and publicly available for running
    the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:b453b1893d052be32b1928c65a03c6e3fd66073a06d789b48894019e1f2d6da7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-pipeline-execution-and-verification

## Summary

Execute the PyINETA Python pipeline end-to-end (load, pick, cluster, find, match, and INADEQUATE-13C-JRES integration) on deposited NMR spectra and verify the outputs against the paper's reported counts, e.g. 65 of 67 INADEQUATE networks having corresponding JRES peaks.

## When to use

Use when reproducing a published NMR metabolomics result from a deposited INADEQUATE and/or 13C-JRES spectrum of a labeled sample, where the claim is a concrete per-network count (e.g. 65 of 67 networks with JRES peaks) that must be re-derived.

## When NOT to use

- When the input is unlabeled (natural-abundance) 1H NMR data rather than 13C INADEQUATE spectra — PyINETA is designed for INADEQUATE NMR spectra of labeled systems
- When only a final summary table is available and the deposited raw spectra needed for load/pick/cluster/find/match steps are missing (as with example2, where raw files exceed 100 Mb and must be requested from the Edison lab)
- When the goal is generic metabolite annotation without needing structural fragment/network information from INADEQUATE

## Inputs

- Deposited INADEQUATE NMR spectrum of a 13C-labeled tissue sample (e.g. liver sample 30)
- Deposited complementary 13C-JRES spectrum from the same sample
- config.ini parameter file for run_pyineta.py
- Simulated INADEQUATE database of metabolites (via PyINETA's matching step)

## Outputs

- MatchJres output table listing, per INADEQUATE network, whether complementary JRES peaks are present
- Networks of picked peaks matched to simulated INADEQUATE database metabolites
- Pipeline output directory with per-step results and parameter tracking (e.g. example1_output)

## How to apply

Clone the PyINETA repository and create the conda environment from environment.yml, then drive the pipeline through run_pyineta.py with a config.ini specifying all options and parameters. Run the step sequence via -s (one of {all, load, pick, cluster, find, match, plot, summary, singleplot, load+, pick+, cluster+, find+, match+}; a trailing + runs all subsequent steps), writing outputs to a dedicated directory with -o. Load and reference the deposited INADEQUATE spectrum (basic shifting), peak-pick it, filter picked peaks into networks of peaks from the same compound, and match those networks to a simulated INADEQUATE database of metabolites; for JRES integration, run the matching/integration step on the same sample's deposited JRES spectrum to attach complementary JRES peak information to each INADEQUATE network and emit the MatchJres table. Verify by counting networks flagged with JRES peaks and comparing to the reported value (65 of 67 for liver sample 30), and confirm the MatchJres table has one row per network. PyINETA keeps track of all pipeline parameters and outputs automatically, which supports annotation transparency and reproducibility.

## Related tools

- **PyINETA** (Python package executing the INADEQUATE reading/referencing, peak picking, network filtering, database matching, and INADEQUATE-13C-JRES integration steps) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for run_pyineta.py CLI driver and the pipeline scripts)
- **NMRbox** (Hosted environment where PyINETA is installed and publicly available for running the pipeline)

## Examples

```
cd <path_to_pyineta_repo>/example/example1/ && python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- MatchJres table exists with the expected per-network rows (one row for each of the 67 INADEQUATE networks)
- Count of networks flagged with JRES peaks equals 65 of 67, matching the paper's reported integration result for liver sample 30
- Cross-check pipeline output against the repository's provided reference output (example1_default_output) when running example1
- PyINETA's automatic parameter/output tracking shows each step (load, pick, cluster, find, match) ran with the config.ini parameters
- Matched networks correspond to metabolites present in the query INADEQUATE spectrum per the simulated INADEQUATE database

## Limitations

- Example2's input and temporary files exceed 100 Mb, so raw files are not in the repository and must be requested from the Edison lab
- Network filtering ideally assigns peaks to the same compound, but networks may not map cleanly to single metabolites; matching relies on a simulated INADEQUATE database
- Referencing uses only basic shifting, so spectra with substantial referencing offsets may need manual adjustment

## Evidence

- [readme] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [readme] You can cross-check the results with the provided output in folder `example1_default_output`.: [evidence span withheld — anchor status: external_doc]
- [abstract] Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics.: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [readme] Since the input and temporary files are larger than 100 Mb, please contact the Edison lab for getting the raw files to try out example2, if needed.: [evidence span withheld — anchor status: external_doc]
