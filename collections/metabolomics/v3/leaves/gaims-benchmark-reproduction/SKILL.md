---
name: gaims-benchmark-reproduction
description: 'Decision support for validating isotopologue deconvolution implementations:
  when you need to check that a new or modified deconvolution/model-ranking pipeline
  (e.g. moiety_modeling mimicking GAIMS) yields comparable optimization results and
  model rankings to a published reference.'
when_to_use_negative:
- Your goal is the paper's recommended deconvolution/model-selection workflow itself
  — use the default logDifference energy function rather than the absolute-difference
  objective, which was introduced only to match GAIMS and overfits (fails to select
  the expert-derived model by AICc).
- Your metabolite or tracer setup does not match the UDP-GlcNAc 13C single-tracer
  benchmark (e.g. multiple-isotope SIRM datasets), since GAIMS/MAIMS-style comparison
  targets only 13C single-isotope data.
- You need metabolic flux analysis from a curated network (EMU-style) rather than
  moiety-model deconvolution of isotopologue profiles.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: moiety_modeling
  role: Runs the modified absolute-difference optimization, analysis, AICc ranking,
    and plotting of the UDP-GlcNAc models
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing plus genetic algorithm optimizer used to fit
    model parameters to the isotopologue data
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Supplementary optimization methods applied after SAGA-optimize refinement
  repo: github.com/scipy/scipy
- name: matplotlib
  role: Visualization of optimized moiety results (fitted isotopologue fraction curves)
  repo: github.com/matplotlib/matplotlib
- name: numpy
  role: Numerical handling of optimized results during plotting
- name: pickle
  role: Saving Python objects in a JSON serializable form and outputting to a file
    for optimization/analysis results
- name: GAIMS
  role: Reference Perl prototype isotopologue deconvolution tool whose results the
    reproduced pipeline is benchmarked against
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:4d93b42ba7943b4dfe189b19d9de3ab3ae0930265cd1460f76c71e08d5835501
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GAIMS
  - SAGA-optimize
  - matplotlib
  - moiety_modeling
  - numpy
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gaims-benchmark-reproduction

## Summary

Reproduce the published moiety_modeling vs GAIMS benchmark for UDP-GlcNAc 13C isotopologue timecourses by switching the moiety_modeling energy function from logDifference to an absolute-difference objective, re-optimizing with SAGA-optimize, and comparing AICc model rankings and fitted parameters against Additional files 9-11.

## When to use

Decision support for validating isotopologue deconvolution implementations: when you need to check that a new or modified deconvolution/model-ranking pipeline (e.g. moiety_modeling mimicking GAIMS) yields comparable optimization results and model rankings to a published reference.

## When NOT to use

- Your goal is the paper's recommended deconvolution/model-selection workflow itself — use the default logDifference energy function rather than the absolute-difference objective, which was introduced only to match GAIMS and overfits (fails to select the expert-derived model by AICc).
- Your metabolite or tracer setup does not match the UDP-GlcNAc 13C single-tracer benchmark (e.g. multiple-isotope SIRM datasets), since GAIMS/MAIMS-style comparison targets only 13C single-isotope data.
- You need metabolic flux analysis from a curated network (EMU-style) rather than moiety-model deconvolution of isotopologue profiles.

## Inputs

- UDP-GlcNAc moiety model JSON file
- 13C isotopologue timecourse dataset JSON file (LnCaP-LN3 human prostate cancer cells)
- optimization settings JSON file
- GAIMS reference results (Additional files 9-11: rank comparison, parameter graphs, parameter table)

## Outputs

- Optimized moiety model parameter sets with objective scores
- AICc-ranked model comparison for UDP-GlcNAc models
- Fitted isotopologue fraction curves and fitted parameter values (plots via matplotlib)
- Cross-tool comparison vs GAIMS: rank order, fitted curves, and parameter table

## How to apply

Load the UDP-GlcNAc model JSON, dataset JSON, and optimization settings JSON into moiety_modeling, but modify the energy function from the default logDifference to an absolute-difference objective to match GAIMS behavior. Run optimization with SAGA-optimize (combined simulated annealing plus genetic algorithm) supplemented by scipy optimization methods, using the quickstart command with --repetition=100 --split --multiprocess. Then analyze the optimizations, rank the UDP-GlcNAc models with --rankCriteria=AICc, and visualize fitted isotopologue fraction curves with numpy/matplotlib. Compare the reproduced rank order against the Additional file 9 rank comparison, the fitted curves against Additional file 10 parameter graphs, and parameter values against the Additional file 11 table. Expect results that are not identical (implementation and stochastic differences exist) but reasonably comparable; note that with this objective function neither method should select the expert-derived model by AICc because of overfitting.

## Related tools

- **moiety_modeling** (Runs the modified absolute-difference optimization, analysis, AICc ranking, and plotting of the UDP-GlcNAc models) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing plus genetic algorithm optimizer used to fit model parameters to the isotopologue data) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Supplementary optimization methods applied after SAGA-optimize refinement) — github.com/scipy/scipy
- **matplotlib** (Visualization of optimized moiety results (fitted isotopologue fraction curves)) — github.com/matplotlib/matplotlib
- **numpy** (Numerical handling of optimized results during plotting)
- **pickle** (Saving Python objects in a JSON serializable form and outputting to a file for optimization/analysis results)
- **GAIMS** (Reference Perl prototype isotopologue deconvolution tool whose results the reproduced pipeline is benchmarked against)

## Examples

```
python3 -m moiety_modeling modeling --models=udp_glcnac_models.json --datasets=udp_glcnac_dataset.json --optimizations=optSettings.json --repetition=100 --split --multiprocess
```

## Evaluation signals

- Reproduced model rank order by AICc matches the rank comparison reported in Additional file 9
- Fitted isotopologue fraction curves visually match the parameter graphs in Additional file 10
- Optimized parameter values are consistent with the parameter table in Additional file 11 (not identical, but reasonably comparable)
- With the absolute-difference objective, the expert-derived model is NOT selected by AICc by either method — matching this failure is itself the expected result
- Optimization completes over --repetition=100 split runs with multiprocess execution without divergent/degenerate fits

## Limitations

- Results are not identical to GAIMS due to implementation and stochastic differences; only qualitative comparability is expected
- With the absolute-difference objective function, AICc overfits and cannot select the expert-derived model — this objective is unsuitable for final model selection
- The benchmark is specific to 13C single-isotope UDP-GlcNAc data; future work is needed to validate multiple-isotope model optimization and selection

## Evidence

- [methods] compare optimization and ranking with GAIMS under matched objective: [evidence span withheld — anchor status: not_found]
- [methods] quickstart optimization command: "python3 -m moiety_modeling modeling --models=<model_jsonfile> --datasets=<dataset_jsonfile> --optimizations=<optimizationSetting_json> --repetition=100 --split --multiprocess"
- [methods] AICc ranking command: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [results] expected comparability finding: [evidence span withheld — anchor status: not_found]
- [abstract] SAGA-optimize role: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [abstract] scipy supplements optimization: "Additional optimization methods from the Python scipy library are utilized as well"
