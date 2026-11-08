---
name: theoretical-isotope-pattern-simulation
description: Use when extracting and scoring M+H isotopologues for targeted metabolites
  in LC-MS data — you need a theoretical isotope pattern per compound, calculated
  at the MS resolution, to compare against observed EIC peak intensities.
when_to_use_negative:
- Quantitative analysis of already-identified metabolites where no isotopologue verification
  is needed — the abstract separates this into a distinct '2.3 Quantitative analysis'
  workflow.
- Untargeted metabolomics peak picking without a predefined target compound list —
  the simulation is computed per defined target compound, not per detected feature.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: enviPat
  role: Calculates the theoretical isotope pattern of each target compound at the
    resolution of the MS for comparison against observed isotopologue patterns
- name: TarMet
  role: Hosts the isotopologues analysis workflow that invokes the simulation, extracts
    M+H isotopologues, and computes isotopic similarity against the simulated pattern
  repo: https://github.com/hcji/TarMet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:72d93fe811f433353f65804f0086cf48ed76582ee4c90ba720b6bac7a221e122
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Theoretical isotope pattern simulation

## Summary

Compute theoretical isotope patterns of target metabolites at the instrument's mass resolution using the enviPat package, so that observed isotopologue peaks in LC-MS data can be compared against an expected reference when quantifying targeted metabolites and their isotopologues.

## When to use

Use when extracting and scoring M+H isotopologues for targeted metabolites in LC-MS data — you need a theoretical isotope pattern per compound, calculated at the MS resolution, to compare against observed EIC peak intensities.

## When NOT to use

- Quantitative analysis of already-identified metabolites where no isotopologue verification is needed — the abstract separates this into a distinct '2.3 Quantitative analysis' workflow.
- Untargeted metabolomics peak picking without a predefined target compound list — the simulation is computed per defined target compound, not per detected feature.

## Inputs

- Target metabolite definition list (compound formulas/identities, e.g. the 82 human blood metabolite targets)
- MS instrument resolution parameter for pattern simulation
- Detected and RT-aligned candidate isotopologue peaks from the isotopologues analysis workflow

## Outputs

- Theoretical isotope patterns (relative abundances of M and isotopologue peaks) per target compound at the instrument resolution
- Isotopic similarity scores comparing observed vs. theoretical patterns, per metabolite per sample

## How to apply

For each target metabolite defined in the TarMet target list, calculate the theoretical isotope pattern of the compound based on the resolution of the MS instrument using the enviPat package; this produces the expected relative abundances of the monoisotopic M and isotopologue (e.g. M+H) peaks. Run this in the isotopologues analysis workflow after peak positions have been detected by MassSpecWavelet, EIC baselines removed by airPLS, and retention-time shifts aligned with PAFFT. Once candidate isotopologue peaks are extracted and filtered with a relative abundance threshold of 0.001, compare the observed isotope pattern against the simulated pattern to compute an isotopic similarity score per metabolite. Interpret the score as a confidence measure: in the human blood metabolites validation, 56 of 76 detected metabolites scored > 0.90 and 69 scored > 0.80, tabulated in the Table S2-equivalent per-metabolite output.

## Related tools

- **enviPat** (Calculates the theoretical isotope pattern of each target compound at the resolution of the MS for comparison against observed isotopologue patterns)
- **TarMet** (Hosts the isotopologues analysis workflow that invokes the simulation, extracts M+H isotopologues, and computes isotopic similarity against the simulated pattern) — https://github.com/hcji/TarMet

## Examples

```
library(enviPat); data(isotopes); pattern <- isopattern(isotopes, "C6H12O6", threshold=0.001, abundances=TRUE)
```

## Evaluation signals

- All simulated patterns are calculated from the compound's formula at the stated MS resolution — check that the resolution parameter passed to enviPat matches the instrument used.
- Isotopic similarity scores are in [0, 1]; on the human blood metabolites validation, TarMet detected all 76 metabolites, with 56 at similarity > 0.90 and 69 at > 0.80 — flag metabolites below 0.80 as low-confidence.
- Per-metabolite similarity scores are tabulated (Table S2 equivalent) — output should include a row per target metabolite, not a single global metric.
- Isotopologue candidate peaks must pass the relative abundance threshold of 0.001 before being scored against the simulated pattern.

## Limitations

- Simulation quality depends on the accuracy of the instrument resolution supplied to enviPat; a mismatched resolution yields patterns that unfairly penalize observed peaks in the similarity comparison.
- Metabolites whose observed isotope patterns deviate from the theoretical pattern score lower on isotopic similarity — in the validation, 20 of 76 detected metabolites fell below 0.90 similarity, so a low score does not necessarily mean the metabolite is absent.
- Planned but not yet supported: the future-work list for TarMet (data-independent acquisition, MS/MS library verification) is out of scope for this simulation-based scoring path.

## Evidence

- [results] Isotopologue comparison metric and thresholds: [evidence span withheld — anchor status: not_found]
