---
name: spectral-peak-network-counting
description: Use when a PyINETA run on an INADEQUATE spectrum has produced its 'networks'
  output and you need an objective count of constructed networks and how many matched
  a database compound to verify the filtering worked.
when_to_use_negative:
- Input is a 1D or J-resolved spectrum rather than an INADEQUATE experiment — network
  counting applies to the INADEQUATE networks produced by the pipeline
- You need compound identity rather than a count — use the Matches table content instead
  of tallying networks
- Spectrum has not yet been peak-picked and filtered by PyINETA; counting raw picked
  peaks is not equivalent to counting networks
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PyINETA
  role: Python package that reads/references the INADEQUATE spectra, picks peaks,
    filters them into networks, and matches networks to a simulated INADEQUATE metabolite
    database; its outputs are what get counted
  repo: https://github.com/rhilt13/PyINETA
- name: Python
  role: Execution environment; pipeline entry point is run_pyineta.py with config.ini
    options
- name: NMRbox
  role: Hosted platform where PyINETA is installed and can be run without local setup
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:dc6ff80cae24caaf45a3767e785c0bb33887cedf550aab88c64fc3d2e4ba0750
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Spectral Peak Network Counting

## Summary

Counts the INADEQUATE peak networks constructed by the PyINETA pipeline and partitions them into matched versus unmatched groups, using the resulting tallies as a quantitative sanity check of the filtering step on a specific 2D 13C-13C NMR spectrum.

## When to use

Use when a PyINETA run on an INADEQUATE spectrum has produced its 'networks' output and you need an objective count of constructed networks and how many matched a database compound to verify the filtering worked.

## When NOT to use

- Input is a 1D or J-resolved spectrum rather than an INADEQUATE experiment — network counting applies to the INADEQUATE networks produced by the pipeline
- You need compound identity rather than a count — use the Matches table content instead of tallying networks
- Spectrum has not yet been peak-picked and filtered by PyINETA; counting raw picked peaks is not equivalent to counting networks

## Inputs

- 2D INADEQUATE NMR spectrum file (deposited, e.g. liver Sample ID 30)
- PyINETA config file (config.ini) with shifting, peak-picking, and filtering parameters
- PyINETA pipeline outputs (networks, matches table)

## Outputs

- Total count of constructed INADEQUATE peak networks
- Count of networks matched to at least one simulated INADEQUATE database metabolite
- Count of unmatched networks
- Count of compounds listed in the Matches table

## How to apply

Run the PyINETA pipeline (e.g. via `python run_pyineta.py -c config.ini`) on the deposited INADEQUATE spectrum; the pipeline automatically reads and references (basic shifting) the spectrum, picks peaks, and filters picked peaks into networks of peaks that ideally come from the same compound. After the 'filter' and 'match' steps, enumerate the constructed networks from the pipeline's tracked outputs, then separately count those with at least one candidate match in the simulated INADEQUATE database of metabolites and those with none. Judge correctness by comparing the tallies to the reported reference: for the liver Sample ID 30 spectrum this is 67 constructed networks, 46 matched to at least one database entry, and 21 unmatched; the same run's Matches table should list 21 identified compounds. Discrepancies indicate a parameter change in shifting/peak-picking/filter settings rather than a counting error, so re-run with the paper's config values before trusting a new count.

## Related tools

- **PyINETA** (Python package that reads/references the INADEQUATE spectra, picks peaks, filters them into networks, and matches networks to a simulated INADEQUATE metabolite database; its outputs are what get counted) — https://github.com/rhilt13/PyINETA
- **Python** (Execution environment; pipeline entry point is run_pyineta.py with config.ini options)
- **NMRbox** (Hosted platform where PyINETA is installed and can be run without local setup)

## Examples

```
python <path_to_pyineta_repo>/run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Constructed network count equals the published value (67 for liver Sample ID 30)
- Matched network count equals published value (46 matched to ≥1 database entry)
- Unmatched network count equals published value (21)
- Matches table lists the published number of identified compounds (21)
- Tally partition satisfies matched + unmatched = constructed (46 + 21 = 67)

## Limitations

- Counts are specific to a given spectrum and parameter set; they are a reproducibility check, not a universal threshold
- PyINETA's network filtering is heuristic — networks are 'ideally' from the same compound, so an unmatched network does not necessarily mean no compound is present
- Example2 in the repository requires contacting the Edison lab for large raw files; counting on such data depends on obtaining them
- No explicit limitations or reproducibility gaps stated in the article's discussion

## Evidence

- [results] PyINETA constructed 67 INADEQUATE networks and matched 46 to database candidates with 21 unmatched: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline reads and references the INADEQUATE spectra, picks peaks, and filters them into networks: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking. It is designed to filter these picked peaks to identify networks of"
- [intro] Networks are matched to a simulated INADEQUATE database of metabolites to identify metabolites in the query spectra: "which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra."
- [readme] CLI entry point for running the PyINETA pipeline with a config file: [evidence span withheld — anchor status: external_doc]
- [abstract] PyINETA keeps track of all the pipeline parameters and outputs: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
