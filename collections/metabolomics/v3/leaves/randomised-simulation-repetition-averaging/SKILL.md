---
name: randomised-simulation-repetition-averaging
description: Use when an in-silico simulation involves randomisation — e.g. randomly
  removing compounds from the ORA background set or simulating metabolite misidentification
  — and stable mean-effect estimates with uncertainty are needed.
when_to_use_negative:
- The simulation step is fully deterministic (no randomised component), so a single
  run already defines the result and repetition adds no information
- The analysis is a single-run ORA on one user dataset rather than a sensitivity/robustness
  simulation across varied inputs
- Ground-truth pathway identities are required — the study explicitly lacked a ground-truth
  dataset where enriched pathway identities are known
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3517
tools:
- name: Python 3.8
  role: Language used to implement all randomised ORA simulations
  repo: https://github.com/cwieder/metabolomics-ORA
- name: scipy stats fisher_exact
  role: Right-tailed (hypergeometric) test recomputed after every randomised perturbation
    step in each repetition
- name: Jupyter Notebook
  role: Hosts reproducible_simulations.ipynb, which loops the 100 randomised samplings
    and computes means and SEMs
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same randomised-simulation notebook
    without local installation
- name: pip3
  role: Installs notebook dependencies required to run the simulation repetitions
    locally
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:2bf3cbab13d6b8e61ded038c3a8a254d350f6702fea79f024c6edc45527fba6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - Python 3.8
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.48
n_claims: 25
---

# randomised-simulation-repetition-averaging

## Summary

Repeats stochastic in-silico perturbation simulations of ORA inputs 100 times and reports the mean retained significant-enriched-pathway (SEP) proportion with the standard error of the mean, so that conclusions about parameter sensitivity (background set, misidentification, DA selection) are not artifacts of a single random sampling.

## When to use

Use when an in-silico simulation involves randomisation — e.g. randomly removing compounds from the ORA background set or simulating metabolite misidentification — and stable mean-effect estimates with uncertainty are needed.

## When NOT to use

- The simulation step is fully deterministic (no randomised component), so a single run already defines the result and repetition adds no information
- The analysis is a single-run ORA on one user dataset rather than a sensitivity/robustness simulation across varied inputs
- Ground-truth pathway identities are required — the study explicitly lacked a ground-truth dataset where enriched pathway identities are known

## Inputs

- Processed metabolite abundance matrix with group labels per dataset (e.g. MTBLS135, MTBLS136, Yachida et al. 2019)
- Assay-specific background compound list and DA metabolite list (BH FDR q <= 0.05)
- Pathway compound sets from KEGG, Reactome, and BioCyc, filtered to pathways with >= 3 compounds
- Randomisation parameters: reduction levels (down to 10% of original compound count), removal mode, or misidentification rate f%

## Outputs

- Mean retained SEP proportion (p <= 0.1, no multiple-testing correction) across 100 random samplings
- Standard error of the mean for each dataset, removal mode, and reduction step
- Per-dataset reduction tables/curves written to background_reduction_sweep.csv

## How to apply

For each dataset, removal mode (random removal or non-DA-only removal of compounds from the background set), and reduction step down to 10% of the original compound count, run the randomised simulation 100 times independently. On every run, re-derive the differentially abundant (DA) metabolite list when the background changes (two-tailed Student's t-tests with Benjamini-Hochberg FDR at q <= 0.05) and recompute ORA p-values with the right-tailed scipy.stats.fisher_exact (hypergeometric) function for pathways with at least 3 compounds when k >= 1. Count significantly enriched pathways (SEPs) at the paper's uncorrected threshold of p <= 0.1, then average the SEP proportion across the 100 samplings and compute the standard error of the mean. Record these mean and SEM values per dataset, removal mode, and reduction step (e.g. written to background_reduction_sweep.csv) so that robust trends — such as the SEP proportion declining as the background shrinks — can be distinguished from sampling noise. Misidentification simulations follow the same averaging convention, with f% of compounds swapped for isobaric (±20 ppm) or identical-chemical-formula counterparts.

## Related tools

- **Python 3.8** (Language used to implement all randomised ORA simulations) — https://github.com/cwieder/metabolomics-ORA
- **scipy stats fisher_exact** (Right-tailed (hypergeometric) test recomputed after every randomised perturbation step in each repetition)
- **Jupyter Notebook** (Hosts reproducible_simulations.ipynb, which loops the 100 randomised samplings and computes means and SEMs) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same randomised-simulation notebook without local installation)
- **pip3** (Installs notebook dependencies required to run the simulation repetitions locally)

## Examples

```
cd metabolomics-ORA/src && pip3 install -r requirements.txt && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Each randomised condition is run exactly 100 times; verify the repetition count before trusting any reported mean
- Every reported SEP proportion is accompanied by a standard error of the mean; SEMs should shrink as the background set stays large (e.g. Fuhrer et al. with >240 metabolites remaining at 10%)
- The DA list is re-derived with BH-FDR-adjusted two-tailed t-tests (q <= 0.05) on each reduced set in the random-removal mode, but held fixed in the non-DA-only-removal mode — check the correct mode-dependent behaviour
- Output table background_reduction_sweep.csv must contain one mean/SEM row per dataset, removal mode, and reduction step
- Re-running the notebook should reproduce the qualitative pattern: SEP proportion declines monotonically as the background shrinks toward 10% of the original compound count

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so the 100-run average quantifies stability but not correctness
- The majority of examples used a p <= 0.1 threshold without multiple testing correction to select SEPs, so averaged SEP counts may include false positives
- Defining the ideal assay-specific background set remains an open problem, so removal sweeps can only characterise sensitivity, not prescribe a correct background
- Two datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications feeding the simulations
- Code was tested only with Python 3.8 on MacOS (v11.2.3) with standard hardware (16GB RAM)

## Evidence

- [discussion] 100 repetitions with mean and SEM: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [discussion] SEP counting at uncorrected p <= 0.1: [evidence span withheld — anchor status: not_found]
- [discussion] ORA recomputation via fisher_exact: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] DA metabolite selection threshold: [evidence span withheld — anchor status: not_found]
- [readme] Notebook execution environment: "All code has been tested using Python 3.8 on MacOS (v11.2.3) with standard hardware (16GB RAM)."
