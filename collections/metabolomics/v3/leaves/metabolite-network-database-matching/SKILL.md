---
name: metabolite-network-database-matching
description: Use when the input is a picked-peak set from an INADEQUATE NMR spectrum
  and the goal is compound annotation by clustering peaks into same-compound networks
  and matching them against a simulated INADEQUATE metabolite database.
when_to_use_negative:
- The experiment is not INADEQUATE — pyINETA is designed for analyzing INADEQUATE
  NMR spectra, not generic 1D/2D spectra without an INADEQUATE pulse sequence.
- Only the complementary 13C-JRES data is available without the corresponding INADEQUATE
  spectrum to build networks from.
- Very large raw input/temporary files are needed but you cannot obtain them — the
  README notes input and temporary files can exceed 100 Mb and must be requested from
  the Edison lab.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0154
tools:
- name: PyINETA
  role: Python package that performs INADEQUATE spectrum reading/referencing, peak
    picking, network filtering of picked peaks, and matching networks to a simulated
    INADEQUATE database of metabolites to identify metabolites in the query spectrum
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Public platform where PyINETA is installed and available for running the INADEQUATE-JRES
    analysis pipeline
- name: Python
  role: Language in which pyINETA is implemented; invoked via run_pyineta.py with
    a config file and step options
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:9359414c8ce802a17292b6b3f63d248d590e7f34fbfcc81d6e250bf1952b16bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-network-database-matching

## Summary

Identifies metabolites in 13C INADEQUATE spectra by grouping picked peaks into networks of peaks from the same compound and matching each network to a simulated INADEQUATE database of metabolites, reporting matched vs unmatched networks and the compounds in the Matches table.

## When to use

Use when the input is a picked-peak set from an INADEQUATE NMR spectrum and the goal is compound annotation by clustering peaks into same-compound networks and matching them against a simulated INADEQUATE metabolite database.

## When NOT to use

- The experiment is not INADEQUATE — pyINETA is designed for analyzing INADEQUATE NMR spectra, not generic 1D/2D spectra without an INADEQUATE pulse sequence.
- Only the complementary 13C-JRES data is available without the corresponding INADEQUATE spectrum to build networks from.
- Very large raw input/temporary files are needed but you cannot obtain them — the README notes input and temporary files can exceed 100 Mb and must be requested from the Edison lab.

## Inputs

- 13C INADEQUATE spectrum (deposited, e.g. liver Sample ID 30) after loading, referencing/basic shifting, and peak picking by pyINETA
- Config file with options and parameters for the INETA run (e.g. config.ini)
- Simulated INADEQUATE database of metabolites

## Outputs

- Networks of picked peaks (67 constructed in the reported liver-sample run)
- Subset of networks with a database entry match (46) and without matches (21)
- Matches table listing identified compounds (21 compounds in the reported run)
- Figures / network plots when the -f figure option is enabled
- Automatically tracked pipeline parameters and output records

## How to apply

After pyINETA reads, references (basic shifting), and peak-picks the INADEQUATE spectrum, run the pipeline's clustering and match steps so that picked peaks are filtered into networks of peaks ideally originating from the same compound, then matched to a simulated INADEQUATE database of metabolites. Invoke the steps via `run_pyineta.py` with a config file and the `-s` step selector (e.g. `cluster`, `find`, `match`, or combined forms like `match+`); PyINETA automatically keeps track of all pipeline parameters and outputs to ensure annotation transparency. Judge success by inspecting the Matches table for identified compounds and by counting constructed networks, networks with a database match, and networks without matches — on the deposited liver Sample ID 30 spectrum these counts were 67 constructed networks, 46 with at least one database candidate, 21 without matches, and 21 identified compounds, and results should cross-check against the paper's Table 1 and Supporting pages S11 and S13.

## Related tools

- **PyINETA** (Python package that performs INADEQUATE spectrum reading/referencing, peak picking, network filtering of picked peaks, and matching networks to a simulated INADEQUATE database of metabolites to identify metabolites in the query spectrum) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Public platform where PyINETA is installed and available for running the INADEQUATE-JRES analysis pipeline)
- **Python** (Language in which pyINETA is implemented; invoked via run_pyineta.py with a config file and step options)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Count of constructed networks matches expected value (67 for the liver Sample ID 30 reference run).
- Count of networks with at least one database entry match matches expected value (46), and the unmatched count is complementary (21).
- Number of compounds in the Matches table matches expected value (21) and its content agrees with the paper's Table 1.
- Network and Matches listings cross-check against Supporting pages S11 (networks) and S13 (Matches table).
- Reproduced output folder contents agree with the reference output (README example: example1_default_output).

## Limitations

- Input and temporary files can be larger than 100 Mb; the README instructs users to contact the Edison lab to obtain raw files for the larger example (example2).
- The paper's discussion does not explicitly state limitations or reproducibility gaps for PyINETA.
- A config file with all options and parameters is required; the single-plot step additionally requires specifying a particular network and a specific database metabolite.

## Evidence

- [intro] Filter picked peaks into same-compound networks and match to simulated INADEQUATE database: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [intro] Basic tasks: reading, referencing (basic shifting), peak picking: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] Pipeline invocation with config and step options: "Run the run_pyineta.py script with the required options.

`python <path_to_pyineta_repo>/run_pyineta.py -c config.ini <other options>`"
- [readme] Larger example requires contacting the lab for raw files: [evidence span withheld — anchor status: external_doc]
- [abstract] PyINETA tracks pipeline parameters and outputs for transparency: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
