---
name: isotope-tracer-extraction
description: Use when you need to quantify target metabolites and their isotopologues
  from MS-based stable isotope tracer experiments, especially to compare observed
  vs. theoretical isotope patterns.
when_to_use_negative:
- When the analysis goal is untargeted feature detection rather than a predefined
  target metabolite/isotopologue panel — TarMet is explicitly a targeted tool.
- When MS/MS spectral library verification or data-independent acquisition is required
  — the README lists these as planned future capabilities not yet supported.
- When raw spectra have not been converted to EICs/peak lists readable by mzR — the
  workflow presupposes exported chromatograms rather than direct instrument files.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: TarMet
  role: Reactive shiny application that orchestrates the isotopologues extraction
    and quantitative analysis workflow
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Provides the reactive graphical user interface for interactive target definition
    and isotopologue inspection
- name: enviPat
  role: Computes theoretical isotope patterns of target compounds at the MS resolution
    for comparison with observed patterns
- name: airPLS
  role: Removes baselines from all extracted ion chromatograms before peak detection
- name: MassSpecWavelet
  role: Detects peak positions in the baseline-corrected EICs
- name: PAFFT
  role: Aligns retention time shifts across samples so isotopologue peaks can be matched
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:f7093f778fc1eece1791ac654abcd5d54d377fa26af0336311081a76b1961f16
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# isotope-tracer-extraction

## Summary

Extracts targeted metabolites and their M+H isotopologues from LC-MS data by combining EIC baseline removal (airPLS), peak detection (MassSpecWavelet), retention-time alignment (PAFFT), and theoretical isotope pattern matching (enviPat) to compute isotopic similarity for stable isotope tracer analysis.

## When to use

Use when you need to quantify target metabolites and their isotopologues from MS-based stable isotope tracer experiments, especially to compare observed vs. theoretical isotope patterns.

## When NOT to use

- When the analysis goal is untargeted feature detection rather than a predefined target metabolite/isotopologue panel — TarMet is explicitly a targeted tool.
- When MS/MS spectral library verification or data-independent acquisition is required — the README lists these as planned future capabilities not yet supported.
- When raw spectra have not been converted to EICs/peak lists readable by mzR — the workflow presupposes exported chromatograms rather than direct instrument files.

## Inputs

- Raw LC-MS peak lists / extracted ion chromatograms for metabolomics samples (e.g. mzR-readable mzML/mzXML)
- Target metabolite definition table (compounds, expected M+H isotopologues)
- MS instrument resolution for enviPat theoretical pattern generation

## Outputs

- Per-metabolite M+H isotopologue peak groups with aligned retention times
- Theoretical vs. observed isotope pattern pairs (enviPat patterns + extracted abundances)
- Isotopic similarity scores per metabolite/sample
- Quantitative isotope tracer assay results (metabolite and isotopologue abundances)

## How to apply

Define the target metabolites first (section 2.1), then run the isotopologues analysis workflow (section 2.2): remove baselines of all EICs with the airPLS method, detect peak positions with the MassSpecWavelet package, and align retention-time shifts across samples with the PAFFT method. Compute theoretical isotope patterns of each target compound at the MS resolution via the enviPat package, extract M+H isotopologues, and filter candidate peaks with a relative abundance threshold of 0.001. Finally, compute the isotopic similarity between observed and theoretical isotope patterns per metabolite to score confidence and support downstream quantitative analysis (section 2.3). The reactive shiny GUI (runTarMet) exposes these steps interactively, and R-script/command-line mode is available via the TarMetCL vignette for batch processing.

## Related tools

- **TarMet** (Reactive shiny application that orchestrates the isotopologues extraction and quantitative analysis workflow) — https://github.com/hcji/TarMet
- **shiny** (Provides the reactive graphical user interface for interactive target definition and isotopologue inspection)
- **enviPat** (Computes theoretical isotope patterns of target compounds at the MS resolution for comparison with observed patterns)
- **airPLS** (Removes baselines from all extracted ion chromatograms before peak detection)
- **MassSpecWavelet** (Detects peak positions in the baseline-corrected EICs)
- **PAFFT** (Aligns retention time shifts across samples so isotopologue peaks can be matched)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- Verify that the theoretical isotope pattern computed by enviPat matches the instrument resolution stated in the target definition, otherwise similarity scores will be systematically biased.
- Check that candidate peak filtering used the relative abundance threshold of 0.001 — lower-value noise peaks should be excluded before similarity computation.
- Confirm retention-time alignment succeeded: the same metabolite's isotopologue peaks should align across samples after PAFFT correction.
- Benchmark against a reference: on the human blood metabolites dataset TarMet detected 76/82 metabolites, with 56 isotopic similarities > 0.90 and 69 > 0.80 (vs. AssayR's 36 and 61); expect similar ordering on comparable data.
- Ensure per-metabolite isotopic similarity scores are tabulated (Table S2 equivalent) so confidence can be audited per target.

## Limitations

- Isotopic similarity depends on the accuracy of enviPat theoretical patterns, which in turn depend on correct compound formulas and the stated MS resolution.
- PAFFT alignment of retention-time shifts may be imperfect across very heterogeneous samples.
- Planned but unsupported features: data-independent acquisition and MS/MS library verification are listed as future work in the README.

## Evidence
