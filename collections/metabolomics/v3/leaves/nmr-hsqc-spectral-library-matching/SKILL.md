---
name: nmr-hsqc-spectral-library-matching
description: Use when you have HSQC mixture spectra (grid spectra or peak lists) and
  a compound spectral library (e.g. 501-metabolite MetaboMiner library), and need
  to detect constituent compounds and estimate concentrations without peak-picking
  the mixture.
when_to_use_negative:
- Input is already a peak list and you need peak-picking-based algorithms like MetaboMiner,
  COLMAR-HSQC, or SMART-Miner as the comparison baseline — this method was designed
  to work without peak-picking the mixture spectrum
- Spectra have severe peak shifts between library and target — sensitivity toward
  peak shifts is a documented limitation and the paper gives no protocol for choosing
  assignment radius or threshold in shifted conditions
- The goal is reliable quantification in highly complex mixtures — the method shows
  diminishing performance for mixtures of increasing complexity and only semiquantitative
  accuracy (up to 34 compounds in the tested library)
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: mcfNMR
  role: 'Runs the minimum cost flow spectral matching: builds the peak network, optimizes
    flow via linear programming, and emits result pickles/CSV/plots'
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: spec2csv
  role: Converts UCSF, Bruker matrix, or peak-list inputs into the mcfNMR target csv.gz,
    optionally binning grid spectra with -b
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear optimizer solving the minimum cost flow assignment of flow from source
    to sink nodes
- name: SciPy
  role: Python interface used to call HiGHS for the eq 8 optimization
- name: MetaboMiner
  role: 'Baseline library source: the HMDB-based 501-compound Wishart library distributed
    with MetaboMiner is loaded as mcfNMR''s library file'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:a4597a83a56ce75a53e414bf9fdc39483360cc948f08ea18521bbd83f86c3777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - COLMAR-HSQC
  - HiGHS
  - MetaboMiner
  - SMART-Miner
  - SciPy
  - mcfNMR
  - spec2csv
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# nmr-hsqc-spectral-library-matching

## Summary

Identify and quantify compounds in 2D 1H,13C HSQC NMR mixtures by fitting library compound spectra simultaneously to the target spectrum via a minimum cost flow (Earth Mover's Distance) optimization on a peak-network, implemented in mcfNMR.

## When to use

Use when you have HSQC mixture spectra (grid spectra or peak lists) and a compound spectral library (e.g. 501-metabolite MetaboMiner library), and need to detect constituent compounds and estimate concentrations without peak-picking the mixture.

## When NOT to use

- Input is already a peak list and you need peak-picking-based algorithms like MetaboMiner, COLMAR-HSQC, or SMART-Miner as the comparison baseline — this method was designed to work without peak-picking the mixture spectrum
- Spectra have severe peak shifts between library and target — sensitivity toward peak shifts is a documented limitation and the paper gives no protocol for choosing assignment radius or threshold in shifted conditions
- The goal is reliable quantification in highly complex mixtures — the method shows diminishing performance for mixtures of increasing complexity and only semiquantitative accuracy (up to 34 compounds in the tested library)

## Inputs

- Target/mixture spectrum as csv (columns '1H', '13C', optional 'weights'), .csv.gz, UCSF file (.uscf), or Bruker matrix (.txt.gz)
- Compound library csv with columns 'name', '1H', '13C', optional 'weights' (e.g. HMDB-based 501-compound Wishart/MetaboMiner library, BMRB-derived spectra)
- TOML configuration file (template: data/user_templates/config_basic.toml)
- Optional peak-list variant of the mixture (columns '1H', '13C', 'weights') for the MCF-D setup

## Outputs

- mcf-results pickle (.pickle) with per-compound flow assignments and detected compounds
- CSV result table of compound detections/weights
- Optional matched-spectrum plot (svg/png/eps/jpg per gfxformat) showing compounds on top of the target spectrum

## How to apply

Convert each mixture spectrum (N925/N987/N988-style grid data or Bruker matrices) into a mcfNMR target csv.gz using spec2csv, keeping the original grid resolution or specifying bin sizes with -b. Build a library csv with columns 'name', '1H', '13C', and optionally 'weights' (e.g. the Wishart/MetaboMiner 501-compound library or BMRB-derived spectra). Write a TOML config setting 'lib', 'target', 'detection_threshold' (assignment > th indicates containment, chosen by observing detections in a blank spectrum of matching noise), 'assignment_radius' (single float for single-pass, a sequence for incremental_fit), and optional flags 'isolated_fit' (fit each compound independently) or 'incremental_fit' (multipass flow reservation). Run 'python -m mcfnmr -c <config_file>' or 'mcfNMR -c <config_file>' to solve the linear optimization via the SciPy interface to HiGHS, producing a mcf-results pickle, a CSV result table of detected compounds and weights, and optional per-compound matched plots. Choose the variant by benchmark: on the 21–27-compound artificial mixtures the incremental setup reached F1 = 0.83, single-pass F1 = 0.81, isolated optimization F1 = 0.80, and peak-list-based F1 = 0.78.

## Related tools

- **mcfNMR** (Runs the minimum cost flow spectral matching: builds the peak network, optimizes flow via linear programming, and emits result pickles/CSV/plots) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **spec2csv** (Converts UCSF, Bruker matrix, or peak-list inputs into the mcfNMR target csv.gz, optionally binning grid spectra with -b) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear optimizer solving the minimum cost flow assignment of flow from source to sink nodes)
- **SciPy** (Python interface used to call HiGHS for the eq 8 optimization)
- **MetaboMiner** (Baseline library source: the HMDB-based 501-compound Wishart library distributed with MetaboMiner is loaded as mcfNMR's library file)

## Examples

```
python -m mcfnmr -c config_basic.toml
```

## Evaluation signals

- On the MetaboMiner artificial mixtures N925/N987/N988 (21–27 compounds, 501-compound library), reproduce F1 scores: incremental ~0.83, single-pass ~0.81, isolated ~0.80, peak-list ~0.78
- Best setup must outperform MetaboMiner (0.65), COLMAR-HSQC (0.64), SMART-Miner (0.68) by the reported relative margin of ~15%
- Detection threshold should be chosen by observing detections in a blank spectrum with the same noise characteristics — detections in a blank should be near-zero
- For quantitative tests with the 34-compound in-house library, reconstructed concentrations should match at least semiquantitative accuracy
- Output schema check: each run must produce a .pickle mcf-result and a .csv result table; config 'load' should be enabled to avoid recomputation on repeated runs

## Limitations

- No definite protocol exists for choosing the assignment radius r, absorption cost cø, or detection threshold ϑ — values must be tuned per dataset
- Sensitivity toward peak shifts between library and target spectra
- Diminishing performance for mixtures of increasing complexity
- Limited quantification accuracy — concentrations are at best semiquantitative
- Overlapping peaks, peak shape changes, or peaks that disappear (matrix/temperature dependent) impede complete reconstruction; unknown constituents have no library peak pattern to match

## Evidence

- [abstract] The method can be used without peak-picking the mixture spectrum and simultaneously optimizes the fit of all individual compound spectra in a given library.: "our approach can be used without peak-picking the mixture spectrum and simultaneously optimizes the fit of all individual compound spectra in a given library"
- [results] F1 score benchmark on artificial mixtures vs baselines: [evidence span withheld — anchor status: not_found]
- [results] Outperforms MetaboMiner, COLMAR-HSQC, SMART-Miner by relative margin: [evidence span withheld — anchor status: not_found]
- [discussion] Threshold choice via blank spectrum: "may thus be obtained by observing the dependency of the number of detections in a blank spectrum with the same noise characteristics on the threshold value"
- [discussion] Optimization implemented via HiGHS through SciPy: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [readme] Repository installation and config invocation: "python -m mcfnmr -c <config_file>"
- [readme] spec2csv conversion of grid spectra with optional binning: "spec2csv -i input_file [-o output_file -w -b bindims]"
- [readme] Config fields for library, target, radius, threshold: "assignment_radius: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true)"
