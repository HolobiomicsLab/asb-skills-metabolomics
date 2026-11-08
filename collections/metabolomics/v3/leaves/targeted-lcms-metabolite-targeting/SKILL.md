---
name: targeted-lcms-metabolite-targeting
description: Use when LC-MS raw data for targeted metabolites or stable isotope tracers
  (e.g. 13C6-glucose-labelled cellular metabolites) must be quantified into isotopologue
  peak-area tables.
when_to_use_negative:
- Data is already a processed feature table or peak matrix — TarMet works from LC-MS
  raw data and target lists.
- Analysis requires data-independent acquisition (DDA) or MS/MS library verification
  — the README lists these as future, not current, features.
- The goal is untargeted metabolomics — TarMet is designed for targeted analyses,
  not global feature discovery.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: TarMet
  role: shiny GUI application orchestrating the full targeted quantification and isotopologue
    analysis workflow
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: calculates theoretical isotope patterns of target compounds based on MS resolution
- name: airPLS
  role: removes baselines of all the EICs before peak detection
- name: MassSpecWavelet
  role: detects peak positions in the baseline-corrected EICs
- name: PAFFT
  role: aligns retention time shifts across samples before peak integration
- name: shiny
  role: provides the reactive graphical user interface for the TarMet application
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:2f046b58a1daf081d7ad476505d9e2af87132d96cfbc9d7c82e7279a5e3b3486
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# targeted-lcms-metabolite-targeting

## Summary

Quantifies targeted metabolites and their isotopologues from LC-MS raw data using TarMet, a shiny reactive GUI tool that computes theoretical isotope patterns with enviPat, removes baselines with airPLS, detects peaks with MassSpecWavelet, and aligns retention times with PAFFT.

## When to use

Use when LC-MS raw data for targeted metabolites or stable isotope tracers (e.g. 13C6-glucose-labelled cellular metabolites) must be quantified into isotopologue peak-area tables.

## When NOT to use

- Data is already a processed feature table or peak matrix — TarMet works from LC-MS raw data and target lists.
- Analysis requires data-independent acquisition (DDA) or MS/MS library verification — the README lists these as future, not current, features.
- The goal is untargeted metabolomics — TarMet is designed for targeted analyses, not global feature discovery.

## Inputs

- LC-MS raw data (e.g. the MM14 dataset released with XCMS-centWave, the Human blood metabolites dataset, or the Cellular metabolites dataset)
- Target metabolite list with expected isotopologues (e.g. glucose-pathway metabolites with max 13C = 6 for 13C6-glucose tracer)
- MS instrument resolution (used by enviPat to compute theoretical isotope patterns)

## Outputs

- Isotopologue peak-area quantification table (e.g. Table S3 for the cellular metabolites dataset)
- Stacked-bar figure of isotopologue labelling distribution across samples (e.g. Fig. 2)
- Aligned EIC peak positions and integrated areas per metabolite and isotopologue

## How to apply

Load the LC-MS raw data and a target metabolite list into the TarMet shiny application. Calculate theoretical isotope patterns of the target compounds based on the resolution of the MS via the enviPat package, then extract ion chromatograms (EICs) for each metabolite and isotopologue, remove baselines using the airPLS method, and detect peak positions with the MassSpecWavelet package. Align retention-time shifts across samples with the PAFFT method, then integrate peak areas automatically to produce an isotopologue peak-area quantification table. For tracer assays, max 13C must match the tracer (e.g. 6 for 13C6-glucose), and parallel samples should be checked for consistency to confirm the quantification is reliable.

## Related tools

- **TarMet** (shiny GUI application orchestrating the full targeted quantification and isotopologue analysis workflow) — https://github.com/hcji/TarMet
- **enviPat** (calculates theoretical isotope patterns of target compounds based on MS resolution)
- **airPLS** (removes baselines of all the EICs before peak detection)
- **MassSpecWavelet** (detects peak positions in the baseline-corrected EICs)
- **PAFFT** (aligns retention time shifts across samples before peak integration)
- **shiny** (provides the reactive graphical user interface for the TarMet application)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- All isotopologues of each target metabolite are quantified and parallel samples show high consistency.
- For a 13C6-glucose tracer assay, longer labelling (60 min) should yield clearly higher 13C incorporation in early pathway metabolites than shorter labelling (5 min), matching the Fig. 2 stacked-bar pattern.
- The output table corresponds to the published Table S3 for the cellular metabolites dataset.
- Theoretical isotope patterns computed by enviPat match the observed isotopologue envelopes at the MS resolution used.
- Peak positions detected by MassSpecWavelet align across parallel samples after PAFFT alignment.

## Limitations

- TarMet currently does not support data-independent acquisition.
- TarMet currently does not support MS/MS library verification.
- Theoretical isotope pattern accuracy depends on the MS resolution supplied to enviPat.

## Evidence

- [readme] In the future
1. Support data-independent acquisition.
2. Support MS/MS library verification.: "1. Support data-independent acquisition."
