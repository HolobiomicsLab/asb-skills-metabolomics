---
name: inadequate-nmr-pipeline-execution
description: Use when you have INADEQUATE 13C-13C NMR spectra from isotope-labeled
  metabolomics samples (e.g. 13C-labeled amino acid feeding studies) and need automated
  compound annotation plus fragment information for unknowns.
when_to_use_negative:
- Input spectra are proton-only (1D 1H or 13C) rather than INADEQUATE 13C-13C correlation
  experiments — the pipeline's network filtering and database matching depend on INADEQUATE
  connectivity.
- No complementary 13C-JRES data exists and the goal is not to match networks against
  the simulated INADEQUATE metabolite database — the integration step will not add
  value.
- The sample cannot be isotopically labeled; the method relies on 13C labeling for
  the INADEQUATE experiment to provide structural connectivity.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3520
tools:
- name: PyINETA
  role: End-to-end Python package that reads/references INADEQUATE spectra, picks
    peaks, clusters networks, matches to a simulated INADEQUATE metabolite database,
    and integrates with 13C-JRES.
  repo: https://github.com/rhilt13/PyINETA.git
- name: NMRbox
  role: Hosting platform where PyINETA is publicly available for execution.
- name: Python
  role: Runtime environment for executing run_pyineta.py and the PyINETA package.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:b10ce887d782278471ab606d3f843e29b95f97f04364b6b9b1e8d95fea709c71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA.git
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# inadequate-nmr-pipeline-execution

## Summary

Execute the PyINETA end-to-end INADEQUATE NMR metabolomics pipeline: load and reference spectra, pick peaks, cluster them into compound networks, match networks to a simulated INADEQUATE metabolite database, and integrate with a complementary 13C-JRES experiment.

## When to use

Use when you have INADEQUATE 13C-13C NMR spectra from isotope-labeled metabolomics samples (e.g. 13C-labeled amino acid feeding studies) and need automated compound annotation plus fragment information for unknowns.

## When NOT to use

- Input spectra are proton-only (1D 1H or 13C) rather than INADEQUATE 13C-13C correlation experiments — the pipeline's network filtering and database matching depend on INADEQUATE connectivity.
- No complementary 13C-JRES data exists and the goal is not to match networks against the simulated INADEQUATE metabolite database — the integration step will not add value.
- The sample cannot be isotopically labeled; the method relies on 13C labeling for the INADEQUATE experiment to provide structural connectivity.

## Inputs

- INADEQUATE NMR spectra from isotope-labeled metabolomics samples
- Complementary 13C-JRES NMR experiment data for integration
- config.ini file with all options and parameters required for the INETA run

## Outputs

- Peak-picked peak lists from the INADEQUATE spectra
- Filtered networks of peaks (candidate compound fragments)
- Matched metabolite identifications from the simulated INADEQUATE database
- Integrated INADEQUATE-13C-JRES annotation with compound and unknown fragment information
- Pipeline parameter and output tracking logs ensuring annotation transparency
- Figures (if -f yes is set)

## How to apply

Install PyINETA (version 2.0.0) via conda using the provided environment.yml, then activate the pyineta environment. Run the pipeline entry point with 'python run_pyineta.py -c config.ini <other options>', supplying a configuration file containing all options and parameters required for the run; use 'python run_pyineta.py -h' to confirm the option list. The pipeline automatically reads and references (using basic shifting) the INADEQUATE spectra, performs peak picking, filters picked peaks into networks of peaks (ideally coming from the same compound), matches networks to a simulated INADEQUATE database of metabolites, and integrates information with the complementary 13C-JRES experiment. Optionally control which steps execute via the -s/--steps flag (one of all, load, pick, cluster, find, match, plot, summary, singleplot, load+, pick+, cluster+, find+, match+) and toggle figure generation with -f yes/no. PyINETA keeps track of all pipeline parameters and outputs, ensuring transparency of annotation in metabolomics.

## Related tools

- **PyINETA** (End-to-end Python package that reads/references INADEQUATE spectra, picks peaks, clusters networks, matches to a simulated INADEQUATE metabolite database, and integrates with 13C-JRES.) — https://github.com/rhilt13/PyINETA.git
- **NMRbox** (Hosting platform where PyINETA is publicly available for execution.)
- **Python** (Runtime environment for executing run_pyineta.py and the PyINETA package.)

## Examples

```
cd <path_to_pyineta_repo>/example/example1/ && python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- The pipeline outputs a Matches table listing identified metabolites; verify each entry corresponds to a network that matched the simulated INADEQUATE database.
- Confirm that every picked peak was either assigned to a filtered network of peaks or explicitly excluded, demonstrating complete coverage of the peak list.
- Check that the integration with the complementary 13C-JRES experiment produced fragment information for unknown compounds where no database match was found.
- Re-run the pipeline with the same config.ini and confirm outputs (networks, matches, parameters) are reproducible, since PyINETA keeps track of all pipeline parameters and outputs.
- Cross-check a known compound's network against the simulated INADEQUATE database entry to validate the matching threshold behavior.

## Limitations

- The pipeline requires a config file with all options and parameters; missing or incorrect parameters will prevent a run.
- Network filtering assumes picked peaks come 'ideally' from the same compound, so noisy or overlapped peaks can fragment true compound networks or merge unrelated ones.
- Only metabolites present in the simulated INADEQUATE database can be identified; unknown compounds receive fragment information only, not full structural assignments.
- Basic shifting is used for referencing, which may be insufficient for spectra with significant referencing offsets.

## Evidence

- [readme] pyINETA is a Python package for analyzing INADEQUATE NMR spectra.: "pyINETA is a Python package for analyzing INADEQUATE NMR spectra."
- [readme] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites"
- [readme] Required: A config file with all the options and parameters required for the INETA run.: "Required: A config file with all the options and
                        parameters required for the INETA run."
- [abstract] PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics.: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [abstract] PyINETA is an open-source platform that provides structural information of molecules using INADEQUATE: "PyINETA is an open-source platform that provides structural information of molecules using INADEQUATE"
