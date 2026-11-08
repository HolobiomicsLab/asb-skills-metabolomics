---
name: minimum-cost-flow-spectral-fitting
description: Use when you need to identify and quantify compounds in 2D 1H,13C HSQC
  NMR mixture spectra against a compound library, especially when peak-picking the
  mixture is undesirable or peaks overlap/shift due to sample-matrix effects.
when_to_use_negative:
- The compound's peak pattern is absent from the library — reconstruction is incomplete
  because no source node exists for its peaks
- You need exact quantitative concentrations rather than semiquantitative estimates
  (limited quantification accuracy is a stated limitation)
- The sample has severe peak shifts or very high-complexity mixtures where MCF performance
  diminishes and the assignment radius/variant choice cannot compensate
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0593
tools:
- name: mcfNMR
  role: 'Primary implementation: constructs the MCF network, solves optimization,
    and writes result pickle/CSV/graphics'
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: spec2csv
  role: Converts UCSC (.uscf), Bruker (.txt.gz) grid spectra or peak-list CSVs into
    the target CSV format mcfNMR consumes
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear optimizer backend (via SciPy) that computes the optimal flow assignment
    from source to sink nodes
- name: SciPy
  role: Python interface to HiGHS used to solve the minimum cost flow problem
- name: MetaboMiner
  role: Baseline algorithm for F1-score comparison on artificial mixtures and plasma
    samples (N907/N926); its HMDB library is reused
  repo: http://wishart.biology.ualberta.ca/metabominer/
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:e14806725d4d190888de1726785772067c2d524d66fd8349c1ee254dd50402ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - MetaboMiner
  - SciPy
  - mcfNMR
  - spec2csv
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# minimum-cost-flow-spectral-fitting

## Summary

Fits 2D 1H,13C HSQC mixture spectra by solving a minimum cost flow (MCF) problem on a network of library- and target-spectrum peaks, enabling simultaneous compound identification and semiquantitative concentration reconstruction via the Earth Mover's Distance.

## When to use

Use when you need to identify and quantify compounds in 2D 1H,13C HSQC NMR mixture spectra against a compound library, especially when peak-picking the mixture is undesirable or peaks overlap/shift due to sample-matrix effects.

## When NOT to use

- The compound's peak pattern is absent from the library — reconstruction is incomplete because no source node exists for its peaks
- You need exact quantitative concentrations rather than semiquantitative estimates (limited quantification accuracy is a stated limitation)
- The sample has severe peak shifts or very high-complexity mixtures where MCF performance diminishes and the assignment radius/variant choice cannot compensate

## Inputs

- Library CSV with compound names, peak coordinates '1H'/'13C', and optional 'weights' (gzipped CSV accepted)
- Target CSV for the mixture spectrum (grid-derived via spec2csv or a peak list)
- mcfNMR TOML configuration file (lib, target, assignment_radius, detection_threshold; optional isolated_fit, incremental_fit, output_dir)
- Dataset options: MetaboMiner HMDB 501-compound library, OLDB grid spectra (Zenodo 14888536), BMRB extracted spectra

## Outputs

- mcf-results pickle (per-compound flow assignments and predicted concentrations c_k = alpha_k * c_k°)
- results CSV with detections and F1 scores per spectrum/method variant
- Optional plot of matched compounds overlaid on the target spectrum (svg/png/eps per gfxformat)

## How to apply

Build a library CSV (columns 'name', '1H', '13C', optional 'weights') and a target CSV for the mixture spectrum, converting grid spectra with spec2csv (e.g. `-b 2,4` binning for UCSC/Bruker grids). Write a TOML config with obligatory fields `lib`, `target`, `assignment_radius`, and `detection_threshold`, then choose a method variant: single-pass MCF (`incremental_fit=false`) for grid data, the peak-list-based MCF-D, or the multipass incremental fit (`incremental_fit=true` with a radius sequence) which is most robust for quantification (e.g. mean relative error 0.18 at r=0.25 on a 34-compound mixture vs 0.25 for single-pass at r=0.15). The MCF is solved with HiGHS via SciPy, matching library peaks to target peaks within a locally restricted assignment radius to avoid nonlocal redistribution, and jointly respecting maximal target capacity unless `isolated_fit=true`. Set `detection_threshold` by observing how many detections appear in a blank spectrum with the same noise characteristics; assignments above the threshold indicate compound containment.

## Related tools

- **mcfNMR** (Primary implementation: constructs the MCF network, solves optimization, and writes result pickle/CSV/graphics) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **spec2csv** (Converts UCSC (.uscf), Bruker (.txt.gz) grid spectra or peak-list CSVs into the target CSV format mcfNMR consumes) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear optimizer backend (via SciPy) that computes the optimal flow assignment from source to sink nodes)
- **SciPy** (Python interface to HiGHS used to solve the minimum cost flow problem)
- **MetaboMiner** (Baseline algorithm for F1-score comparison on artificial mixtures and plasma samples (N907/N926); its HMDB library is reused) — http://wishart.biology.ualberta.ca/metabominer/

## Examples

```
mcfNMR -c config_basic.toml  # config: lib='lib.csv', target='N907.csv.gz', assignment_radius=0.15, detection_threshold=0.05
```

## Evaluation signals

- Reproduced F1 scores match article values: e.g. MCF on N926 plasma = 81.6 (vs MetaboMiner 82.9) and MCF on N907 = 83.5 (vs 76.2) using the plasma common library
- For quantification, mean relative error magnitudes match reported optima: setup A ≈0.25 at r=0.15, setup B ≈0.30 at r=0.1, setup C (incremental) ≈0.18 at r=0.25 on the 34-compound 3 mM mixture
- Detections in a blank spectrum with identical noise characteristics stay at or below the chosen detection_threshold, validating the threshold
- Results CSV columns (spectrum, method, F1) reproduce the relative ranking: MCF-D improves on pH-disturbed N907 vs MetaboMiner
- Config validation: assignment_radius is a single float for single-pass and a sequence for incremental_fit; mcfNMR caches results in output_dir/mcf-results .pickle + .csv

## Limitations

- No definite protocol exists for choosing assignment radius r, absorption cost cø, and detection threshold ϑ; parameter choice remains method- and data-dependent
- Sensitivity toward peak shifts; diminishing performance for mixtures of increasing complexity; limited quantification accuracy
- Reliability-testing methods (perturbative re-runs, library-subset bootstrapping) were proposed but not fully developed — reconstructions should be treated as tentative
- Some TextTest runs may fail due to unsynchronized subprocess output during linear optimization; tests must run twice due to caching

## Evidence

- [abstract] MCF solves assignment as flow problem: "a minimum cost flow problem is solved on a network consisting of nodes that represent spectral peaks of the library compounds and the mixture"
- [abstract] No peak-picking needed; simultaneous fit: "our approach can be used without peak-picking the mixture spectrum and simultaneously optimizes the fit of all individual compound spectra in a given library"
- [intro] Assignment radius restricts matching locally: "determining whether
a peak of a spectrum X appears in Y requires a locally restricted comparison of intensities. We addressed
this problem by introducing an assignment radius r"
- [discussion] HiGHS via SciPy for optimization: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [abstract] Quantitative accuracy on 34-compound mixture: "our method retrieves individual compound concentrations with at least semiquantitative accuracy for artificial mixtures with up to 34 compounds"
- [readme] Configuration fields: "`lib`: Path to a library-file containing the names, peak-weights, and -coords of the spectra a set of compounds."
- [discussion] Threshold selection via blank: "may thus be obtained by observing the dependency of the number of
detections in a blank spectrum with the same noise characteristics
on the threshold value"
- [readme] spec2csv binning option: "For grid spectra, the option `-b` allows to specify the size of bins for the target spectrum relative to its original 
resolution."
