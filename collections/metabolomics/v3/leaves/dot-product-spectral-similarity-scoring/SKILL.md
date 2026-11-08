---
name: dot-product-spectral-similarity-scoring
description: Use when annotating LC-MS/MS or LC-MSn lipid features against an in-silico
  spectral library (e.g. LipiDex 2 Library Generator output) and you need to score
  match quality of MS2 and MS3+ spectra before applying identification thresholds.
when_to_use_negative:
- The input spectra are MS1-only or derived from a non-lipidomics proteomics/metabolomics
  workflow where the LipiDex 2 library and thresholds do not apply
- You need de novo spectral clustering or library-free similarity search rather than
  scoring against a defined in-silico lipid library
- The spectra are already annotated with accepted match scores and you only need downstream
  QC (e.g. Degreaser ECN-vs-RT modeling) rather than re-scoring
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2
  role: Implements Spectrum Searcher spectral matching, computing dot-product and
    reverse dot-product scores of MS2/MS3+ spectra against in-silico library entries
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Library Generator
  role: Generates the in-silico library spectra (with MSn tree-based fragmentation)
    that serve as the comparison reference for dot-product scoring
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Lipid Spectral Annotator module
  role: Consumes passing Spectrum Searcher MS2/MS3+ matches and annotates fragment
    peaks downstream of the scoring step
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:ce1196c02ffd22f71283c6018d3d0be3d58667b2c12efdbe4d744169567a8c0b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Library Generator
  - LipiDex 2 Lipid Spectral Annotator module
  - Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Dot-Product Spectral Similarity Scoring

## Summary

Cosine dot-product and reverse dot-product scoring of experimental MS2/MS3 spectra against in-silico lipid library spectra, as implemented in LipiDex 2's Spectrum Searcher, to quantify spectral match quality for lipid identification.

## When to use

Use when annotating LC-MS/MS or LC-MSn lipid features against an in-silico spectral library (e.g. LipiDex 2 Library Generator output) and you need to score match quality of MS2 and MS3+ spectra before applying identification thresholds.

## When NOT to use

- The input spectra are MS1-only or derived from a non-lipidomics proteomics/metabolomics workflow where the LipiDex 2 library and thresholds do not apply
- You need de novo spectral clustering or library-free similarity search rather than scoring against a defined in-silico lipid library
- The spectra are already annotated with accepted match scores and you only need downstream QC (e.g. Degreaser ECN-vs-RT modeling) rather than re-scoring

## Inputs

- Experimental MS2 and MS3+ spectra extracted from Thermo .raw files (via Thermo RawFileReader) or converted .mgf/.mzXML files
- In-silico lipid spectral library from the LipiDex 2 Library Generator (MSn tree-based fragmentation)
- Chromatographic peak table from Peak Finder / Compound Discoverer 3.3 (.cdResult or .csv)

## Outputs

- Per-match dot product and reverse dot product scores for each MS2 and MS3+ spectral match
- Pass/fail classification of each spectral match against the ≥500 dot-product and ≥700 reverse dot-product thresholds
- Filtered list of high-confidence lipid annotations passing all quality criteria for downstream Degreaser QC and Metaboanalyst export

## How to apply

For each chromatographic feature with MS2 and/or MS3+ spectra, compare the experimental spectrum against the candidate lipid's in-silico library spectrum (generated with the LipiDex 2 Library Generator's MSn tree-based fragmentation logic). Compute the cosine dot product between the two spectra as a forward match score, and compute the reverse dot product to assess how much of the library spectrum is explained by the experimental one — the reverse score guards against sparse or one-peak-forward matches. Apply LipiDex 2's match-quality thresholds for retention of an annotation: minimum dot product of 500 and minimum reverse dot product of 700. Combine these spectral scores with the spectral purity estimate (fraction of fragment ion intensity attributable to the matched lipid versus coeluting isobaric species; require ≥75% purity) so that matches passing all three criteria can be annotated and downstream fragment peaks labeled by the Lipid Spectral Annotator.

## Related tools

- **LipiDex 2** (Implements Spectrum Searcher spectral matching, computing dot-product and reverse dot-product scores of MS2/MS3+ spectra against in-silico library entries) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Library Generator** (Generates the in-silico library spectra (with MSn tree-based fragmentation) that serve as the comparison reference for dot-product scoring) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Lipid Spectral Annotator module** (Consumes passing Spectrum Searcher MS2/MS3+ matches and annotates fragment peaks downstream of the scoring step) — https://github.com/coongroup/LipiDex-2

## Examples

```
run LipiDex2.exe → Spectrum Searcher → set MS2 match thresholds (dot product ≥500, reverse dot product ≥700, spectral purity ≥75%) → output annotated_spectral_matches.csv
```

## Evaluation signals

- Every retained spectral match has dot product ≥500 and reverse dot product ≥700
- Spectral purity of matched features is ≥75% of fragment intensity attributable to the matched lipid versus coeluting isobaric species
- Per-match scores reproduce values in the deposited annotated spectral match table (MSV000091033 reprocessing)
- Matches failing any threshold are flagged for removal and not carried into the annotated feature table
- Result .csv propagates cleanly to the Degreaser QC module and Metaboanalyst export

## Limitations

- Thresholds (500 dot product, 700 reverse dot product, 75% purity) are calibrated for the LipiDex 2 LC-MS processing workflow and Nile rat plasma dataset; may need re-tuning for other matrices or instruments
- Reverse dot product alone can overstate match confidence for spectra with few fragments — must be interpreted jointly with forward dot product and spectral purity
- LipiDex 2 requires .NET 7.0 and Spectrum Searcher expects Thermo .raw support or pre-converted .mgf/.mzXML files
- LipiDex 1 (legacy Java version, .mgf/.mzXML input) is no longer supported and uses different filtering logic

## Evidence

- [results] Annotation thresholds for LC-MS processing workflow: [evidence span withheld — anchor status: not_found]
