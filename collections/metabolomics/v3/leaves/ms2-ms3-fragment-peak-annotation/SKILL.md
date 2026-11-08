---
name: ms2-ms3-fragment-peak-annotation
description: Use when a LC-MS lipidomics pipeline has already produced MS2 and MS3+
  spectral matches (e.g. from LipiDex 2 Spectrum Searcher) and fragment-level peak
  assignments are needed to validate or annotate chromatographic features.
when_to_use_negative:
- Input spectra have not yet been assigned spectral matches by Spectrum Searcher —
  the Annotator requires existing MS2/MS3+ matches as its input
- The goal is lipid identification from raw spectra rather than fragment-level annotation
  of already-identified features; use Spectrum Searcher first
- The peak table originates from a tool other than the LipiDex 2 pipeline's CSV format,
  since the Annotator consumes LipiDex 2's inter-module CSV results
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Lipid Spectral Annotator
  role: Core module that performs the MS2/MS3+ fragment peak annotation
  repo: https://github.com/coongroup/LipiDex-2
- name: Spectrum Searcher
  role: Upstream LipiDex 2 module supplying the MS2 and MS3+ spectral matches the
    Annotator consumes
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Parent LC-MS lipidomics pipeline within which the Annotator runs as a stand-alone
    module
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:5437b53411f2a5a5de59f566c66f330943acfef0a01f1cec10637d40a6e72c63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Lipid Spectral Annotator
  - Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# MS2/MS3 fragment peak annotation

## Summary

Annotate fragment peaks on chromatographic lipid features using MS2 and MS3+ spectral matches, as implemented in the LipiDex 2 Lipid Spectral Annotator module. This turns raw Spectrum Searcher spectral matches into per-fragment peak assignments that support downstream purity and quality-control evaluation.

## When to use

Use when a LC-MS lipidomics pipeline has already produced MS2 and MS3+ spectral matches (e.g. from LipiDex 2 Spectrum Searcher) and fragment-level peak assignments are needed to validate or annotate chromatographic features.

## When NOT to use

- Input spectra have not yet been assigned spectral matches by Spectrum Searcher — the Annotator requires existing MS2/MS3+ matches as its input
- The goal is lipid identification from raw spectra rather than fragment-level annotation of already-identified features; use Spectrum Searcher first
- The peak table originates from a tool other than the LipiDex 2 pipeline's CSV format, since the Annotator consumes LipiDex 2's inter-module CSV results

## Inputs

- MS2 spectral matches from LipiDex 2 Spectrum Searcher
- MS3+ spectral matches from LipiDex 2 Spectrum Searcher
- LipiDex 2 pipeline CSV results files

## Outputs

- Annotated fragment peak table (CSV) mapping fragment peaks to matched lipid species
- CSV results files consumable by downstream LipiDex 2 modules (e.g. Degreaser QC)

## How to apply

Take the MS2 and MS3+ spectral matches output by the LipiDex 2 Spectrum Searcher module and pass them to the LipiDex 2 Lipid Spectral Annotator, which annotates the fragment peaks associated with each matched lipid. The Annotator operates on the CSV results files that LipiDex 2 modules pass between pipeline stages; each module is stand-alone, so the annotated fragment table can be exported as a CSV and fed into the Degreaser QC module or other downstream steps. Evaluate the annotations against the spectral match quality metrics (e.g. spectral purity, dot product, reverse dot product thresholds used in the paper's LC-MS processing method) so that only fragment peaks from passing matches are retained for feature annotation.

## Related tools

- **LipiDex 2 Lipid Spectral Annotator** (Core module that performs the MS2/MS3+ fragment peak annotation) — https://github.com/coongroup/LipiDex-2
- **Spectrum Searcher** (Upstream LipiDex 2 module supplying the MS2 and MS3+ spectral matches the Annotator consumes) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Parent LC-MS lipidomics pipeline within which the Annotator runs as a stand-alone module) — https://github.com/coongroup/LipiDex-2

## Examples

```
LipiDex2.exe --module LipidSpectralAnnotator --input spectrum_searcher_matches.csv --output annotated_fragments.csv
```

## Evaluation signals

- Every fragment peak in the output table maps to a lipid species that has a passing MS2 or MS3+ spectral match (meeting the paper's spectral purity and dot-product/reverse-dot-product criteria)
- The annotated output is a CSV file consistent with LipiDex 2's inter-module CSV results format, allowing direct handoff to Degreaser QC
- Fragment annotations are consistent with the fragmentation templates defined in the in-silico spectral library used by Spectrum Searcher
- Downstream Degreaser QC accepts the annotated table without format errors, confirming correct inter-module handoff

## Limitations

- The module requires MS2 and MS3+ spectral matches as input and cannot operate on unannotated raw spectra
- Annotation quality depends on the upstream spectral library (e.g. LipidBlast or custom in-silico libraries generated by the Library Generator) and its MSn tree-based fragmentation logic
- If a required fragment is absent, LipiDex 2's logic downgrades the identification to sum composition or to plasmanyl, so fragment annotations may reflect lower structural resolution than the matched species

## Evidence

- [intro] Annotator uses Spectrum Searcher MS2/MS3+ matches: [evidence span withheld — anchor status: not_found]
