---
name: baseline-method-performance-comparison
description: Use when introducing a new spectral library-matching or reconstruction
  algorithm and you need a defensible performance comparison on the same target spectra
  (artificial mixtures N925/N987/N988 and plasma N907/N926) against named competing
  tools.
when_to_use_negative:
- The input is already a feature/peak table where a simple cosine or similarity-score
  lookup against a library is the established standard and no reconstruction or competing
  baseline is required.
- You only need concentration quantification without any head-to-head comparison against
  other algorithms — the MCF method's quantification accuracy is limited and the benchmarking
  overhead is unnecessary.
- The target spectra contain compounds absent from the library (complete reconstruction
  is impossible when the mixture contains unknown peak patterns)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: mcfNMR
  role: Implements the minimum-cost-flow reconstruction whose four setups (A-D) are
    benchmarked; provides the classifier, result pickles and CSV outputs.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: MetaboMiner
  role: Published baseline algorithm whose F1 scores on the same 501-compound library
    / mixtures are loaded for comparison.
- name: COLMAR-HSQC
  role: Published baseline algorithm included in the head-to-head F1 comparison.
- name: SMART-Miner
  role: Published baseline algorithm included in the head-to-head F1 comparison.
- name: SciPy
  role: Provides the linear optimizer interface (to HiGHS) that solves the minimum
    cost flow; SciPy is cited in the methods as the numerical backend.
- name: spec2csv
  role: Converts grid spectra (.uscf, Bruker matrices) and peak lists into mcfNMR
    target .csv.gz files so that baseline and new methods operate on identical inputs.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:3e81a5caefa3730bc19be5b90a1fb4d912619ce0b2c3f61a6304b7088ff89c17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - COLMAR-HSQC
  - MetaboMiner
  - SMART-Miner
  - SciPy
  - mcfNMR
  - spec2csv
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# baseline-method-performance-comparison

## Summary

Benchmark a new compound-identification method (e.g. mcfNMR minimum-cost-flow fitting) against established baselines (MetaboMiner, COLMAR-HSQC, SMART-Miner) by computing precision/recall/F1 on shared artificial HSQC mixtures and a natural plasma sample, and reporting the relative F1 margin.

## When to use

Use when introducing a new spectral library-matching or reconstruction algorithm and you need a defensible performance comparison on the same target spectra (artificial mixtures N925/N987/N988 and plasma N907/N926) against named competing tools.

## When NOT to use

- The input is already a feature/peak table where a simple cosine or similarity-score lookup against a library is the established standard and no reconstruction or competing baseline is required.
- You only need concentration quantification without any head-to-head comparison against other algorithms — the MCF method's quantification accuracy is limited and the benchmarking overhead is unnecessary.
- The target spectra contain compounds absent from the library (complete reconstruction is impossible when the mixture contains unknown peak patterns)

## Inputs

- 2D 1H,13C HSQC NMR target spectra of artificial mixtures (N925, N987, N988) and plasma samples (N907, N926) as grid spectra (.uscf, Bruker .txt.gz) or peak lists (.csv)
- Compound library file with 'name', '1H', '13C', and optional 'weights' columns (MetaboMiner 501-compound HMDB/Wishart library; in-house 34-compound OLDB library)
- Ground-truth constituent lists for the artificial mixtures
- Published benchmark results (F1 scores) of MetaboMiner, COLMAR-HSQC, SMART-Miner on the same mixtures
- TOML configuration files specifying assignment_radius, detection_threshold, isolated_fit, incremental_fit, peak-list targets

## Outputs

- Per-compound detection/containment results stored as pickled mcf-results (.pickle)
- CSV result tables with precision, recall and F1 per method and per mixture
- Relative F1 margin of the new method over baseline tools (expected ~15% on artificial mixtures)
- Optional per-mixture overlaid plot of matched compounds on the target spectrum (svg/png/eps/jpg via `plot=true`)
- Quantitative reconstruction error of compound concentrations (semiquantitative accuracy up to 34-compound mixtures)

## How to apply

Run the new method (here, mcfNMR via `python -m mcfnmr -c <config_file>`) with the four MCF setups (A single-pass, B isolated_fit=true, C incremental_fit with a radius sequence, D peak-list targets) on the MetaboMiner 501-compound library, converted target spectra (grid spectra converted with `spec2csv`), and known constituent lists. For each setup and each of the three artificial mixtures, extract per-compound detections from the pickled mcf-results and compute precision, recall and F1 against the ground-truth mixture composition; then load the published F1 values of MetaboMiner, COLMAR-HSQC and SMART-Miner on the same mixtures and compute the relative margin (expected ~15%). Repeat the comparison on the natural plasma sample (N907/N926) to test robustness on real matrix effects (pH-shifted peaks). Choose optimal `assignment_radius` and `detection_threshold` per setup — the paper notes no definite protocol exists, so parameters are tuned e.g. by observing detections in a blank spectrum with the same noise characteristics.

## Related tools

- **mcfNMR** (Implements the minimum-cost-flow reconstruction whose four setups (A-D) are benchmarked; provides the classifier, result pickles and CSV outputs.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **MetaboMiner** (Published baseline algorithm whose F1 scores on the same 501-compound library / mixtures are loaded for comparison.)
- **COLMAR-HSQC** (Published baseline algorithm included in the head-to-head F1 comparison.)
- **SMART-Miner** (Published baseline algorithm included in the head-to-head F1 comparison.)
- **SciPy** (Provides the linear optimizer interface (to HiGHS) that solves the minimum cost flow; SciPy is cited in the methods as the numerical backend.)
- **spec2csv** (Converts grid spectra (.uscf, Bruker matrices) and peak lists into mcfNMR target .csv.gz files so that baseline and new methods operate on identical inputs.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR

## Examples

```
python -m mcfnmr -c config_mcf_c.toml  # after: source scripts/download_demodata.sh 'metabominer'; spec2csv -i n925.uscf -o n925.csv.gz
```

## Evaluation signals

- Best MCF setup must exceed baseline F1 (MetaboMiner 0.65, COLMAR-HSQC 0.64, SMART-Miner 0.68) by the reported relative margin of 15% on the artificial mixtures.
- Setup C (incremental_fit=true with a radius sequence) should reach the highest average F1 (~0.83), followed by A (~0.81), B (~0.80), D (~0.78).
- On the plasma sample all methods should show degraded performance (F1 slightly larger than 0.5), reflecting real matrix effects; MCF-D should outperform there especially on pH-disturbed N907.
- Per-compound detection must reproduce the known constituent counts (21–27 compounds per artificial mixture) within the precision/recall reported.
- Concentration retrieval on the 34-compound library should be semiquantitatively accurate; deviations beyond that indicate a misconfigured assignment_radius or detection_threshold.

## Limitations

- No definite answer or protocol exists for choosing assignment radius r, absorption cost cø, and detection threshold ϑ — parameter choice currently relies on empirical tuning.
- Sensitivity toward peak shifts; matrix and temperature can shift, reshape or erase peaks, degrading performance on natural samples (plasma F1 ~0.5).
- Performance diminishes for mixtures of increasing complexity; quantification accuracy is limited (semiquantitative at best).
- The original MetaboMiner website is unreachable; reference files must be fetched from web.archive.org.
- TextTest-based repository tests must be run twice; some subprocess outputs may desynchronize and are filtered heuristically.

## Evidence

- [discussion] Benchmark design and expected F1 margin: "our method achieved higher F1 scores than all other methods (by a relative margin of 15%)"
- [intro] Baseline algorithms named for comparison: "three other popular algorithms: MetaboMiner,12 COLMAR-HSQC,13 and SMART-Miner.14"
- [discussion] Optimization backend used to produce results: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [methods] Dataset provenance for demodata and benchmarks: "several grid spectra of individual compounds and mixtures provided by GeoMetabolomics @ ICBM, University of Oldenburg ([Dataset link](https://zenodo.org/records/14888536))."
- [readme] README: installation and run entry points: "To install `mcfNMR` for an existing Python environment:

	> pip install mcfNMR

makes the commands ['mcfNMR'](#configuration-file) and ['spec2csv'](#preparing-target-spectra) available."
- [readme] README: running a classification run: "(.mcfnmr_venv)> python -m mcfnmr -c <config_file>"
- [readme] README: target spectra conversion tool: "Currently supported: USCF files (.uscf), Bruker matrices (.txt.gz), and peak lists (.csv) with columns '1H', '13C', and optionally 'weights'."
- [readme] README: demodata download for benchmark reproduction: "The data can be downloaded with

    > cd $MCFNMR_HOME
    > source scripts/download_demodata.sh [<DATASET>]"
- [readme] README: demodata scope (MetaboMiner library, Oldenburg spectra): "the HMDB-based compound library, which was provided by the Wishart lab (and used for [MetaboMiner](http://wishart.biology.ualberta.ca/metabominer/))."
- [readme] README: why web archive is needed for MetaboMiner: "Since the original MetaboMiner website cannot be reached (Feb 19 2025), we download
MetaboMiner files from web.archive.org."
- [discussion] Article: no definite parameter-choice protocol: "To date, we have no definite
answer or protocol to choose these values"
- [discussion] Article: plasma performance ceiling: "They
achieved F1 scores of values slightly larger than 0.5, pointing toward
a potential for further refinements"
- [discussion] Article: limitations of the method: "(i) reliance
on choice of parameters and method variant"
