---
name: mass-decimal-precision-parameterization
description: Use when solving CSCCP structure-elucidation tasks with csccp-solver-cli
  or G.A.M.E. and a targeted molecular weight must be matched at a chosen decimal
  precision (dec 1–5), balancing runtime cost against mass-spectrum resolution.
when_to_use_negative:
- Input mass spectra are low-resolution, so five decimal digits (D=5) of precision
  would inflate DP runtime tenfold per digit without matching the data's actual accuracy
- Only CPU resources are available — at D=5 the DP algorithm's runtime grows by orders
  of magnitude and becomes impractical without the G.A.M.E. GPU acceleration
- 'The task is generic compound search rather than CSCCP scaffold-extension: no seed
  scaffold database (.cIdx files) with sidechain probabilities is available'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: G.A.M.E.
  role: CUDA-based GPU-accelerated implementation of the DP solver that makes five-decimal-digit
    mass precision tractable; invoked via csccp-solver-cli with the --dec flag
  repo: https://github.com/CMDM-Lab/GAME
- name: PyCuda 2017.1
  role: Python CUDA binding used to implement the GPU parallelization over w values
    in the DP algorithm
  repo: https://github.com/CMDM-Lab/GAME
- name: NVIDIA Tesla K40c
  role: GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the D=5
    benchmark was executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:17b6c6a82d8940b5d53aac25b5acbcaaaa2c9721b77b3d274dd4a0e4e26765fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - CUDA 7.5
  - G.A.M.E.
  - NVIDIA Tesla K40c
  - PyCuda 2017.1
  - Python 2.7
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Mass decimal precision parameterization for DP-based structure elucidation

## Summary

Configures the decimal-digit precision (dec/D) of input molecular weights for dynamic programming (DP)-based chemical structure elucidation, converting mass values to integers by multiplying by 10^D. GPU acceleration via G.A.M.E. makes D=5 (five decimal digits) practical, where the CPU DP runtime grows by orders of magnitude.

## When to use

Use when solving CSCCP structure-elucidation tasks with csccp-solver-cli or G.A.M.E. and a targeted molecular weight must be matched at a chosen decimal precision (dec 1–5), balancing runtime cost against mass-spectrum resolution.

## When NOT to use

- Input mass spectra are low-resolution, so five decimal digits (D=5) of precision would inflate DP runtime tenfold per digit without matching the data's actual accuracy
- Only CPU resources are available — at D=5 the DP algorithm's runtime grows by orders of magnitude and becomes impractical without the G.A.M.E. GPU acceleration
- The task is generic compound search rather than CSCCP scaffold-extension: no seed scaffold database (.cIdx files) with sidechain probabilities is available

## Inputs

- Targeted molecular weight value (parameter -m), e.g. 168.195105 from GAME/examples/MW/
- Seed scaffold files (*.cIdx) from GAME/examples/core_index or GAME/examples/Datasets/scaffolds
- High-resolution mass spectral peak (mass_peak with ppm tolerance, e.g. ppm=5)
- Decimal precision parameter dec/D (1–5) and result count r (typically 3–10)

## Outputs

- Ranked list of top-r candidate molecular structures with probability and weight columns (SMILES strings)
- csccp info block reporting scaffold, mass_peak, mass_peak_min/max, configuration, and number of compounds

## How to apply

Convert all mass values to integers by multiplying by 10^D before DP execution, since the program operates on integer-scaled masses. Choose D by the resolution of the input high-resolution mass spectrum; the README notes dec is generally set to 1–5, and each additional decimal digit increases DP running time roughly tenfold. Run the solver with the --dec flag (e.g. dec=5 with -r 3 and -v idp implemented in C++), supplying seed scaffold files (*.cIdx) via -s and the targeted weight via -m. At D=5 the CPU DP algorithm's runtime grows by orders of magnitude, so use the CUDA-based G.A.M.E. (Python 2.7, CUDA 7.5, PyCuda 2017.1 on an NVIDIA Tesla K40c) to make five-decimal-digit precision tractable. Validate by checking that the returned top-r candidates have weights matching the targeted mass to the requested decimal precision and that runtimes remain practical (paper reports ~20.72 s average G.A.M.E. time at D=5 vs. 324.16 s CPU DP average).

## Related tools

- **G.A.M.E.** (CUDA-based GPU-accelerated implementation of the DP solver that makes five-decimal-digit mass precision tractable; invoked via csccp-solver-cli with the --dec flag) — https://github.com/CMDM-Lab/GAME
- **PyCuda 2017.1** (Python CUDA binding used to implement the GPU parallelization over w values in the DP algorithm) — https://github.com/CMDM-Lab/GAME
- **NVIDIA Tesla K40c** (GPU hardware (compute capability 3.5, 15 SMs of 192 cores) on which the D=5 benchmark was executed)

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- The chosen dec value is in the documented 1–5 range and D is consistent between integer-scaled masses and the solver's --dec flag
- Top-ranked candidates report weights matching the targeted mass_peak to the requested decimal precision (e.g. 1.681951e+02 for -m 168.195105)
- At D=5, G.A.M.E. runtimes remain bounded (~20.72 s average) rather than the orders-of-magnitude CPU DP times (~324.16 s average)
- The csccp info output echoes mass_peak_min and mass_peak_max consistent with the ppm tolerance (e.g. ppm=5)

## Limitations

- Each additional decimal digit of mass increases DP running time roughly tenfold; at D=5 CPU DP execution grows by orders of magnitude and is impractical
- The DP algorithm, though pseudo-polynomial, can still require exponential time in the worst case
- Integer mass scaling requires all mass values be multiplied by 10^D in the program, constraining representable precision to the chosen D

## Evidence

- [abstract] CUDA-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data: "CUDA)-based GPU-accelerated mixture elucidator (G.A.M.E.) that considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [methods] Mass values must be converted to integers by multiplying by 10^D in the program: "all mass values must be converted to integers by multiplying 10D in the program"
- [readme] PS. In general, r is set to 3~10, dec is set to 1~5: "PS. In general, r is set to 3~10, dec is set to 1~5"
- [readme] The example CLI invocation with dec=5 on a targeted weight of 168.195105: "$> csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5"
