---
name: de-novo-confidence-score-filtering
description: Use when raw confidence-scored de novo peptide sequencing predictions
  (e.g. from Casanovo or Novor on antibody MS/MS spectra) must be filtered before
  assembly, and per-tool precision-score cutoffs are needed.
when_to_use_negative:
- When peptides were identified by database search rather than de novo sequencing
  — filtering targets unvalidated de novo predictions, not scored PSMs against a reference
  database
- 'When no per-tool precision-vs-score calibration is available: thresholds differ
  between Casanovo, Novor, PointNovo etc., so a single generic cutoff cannot be transferred
  between tools'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Casanovo
  role: Deep-learning de novo predictor whose confidence-scored and precursor-mass-filtered
    output is the primary candidate set for filtering before ALPS assembly
- name: Novor
  role: Spectrum-graph-based de novo predictor whose predictions are filtered at its
    own tool-specific confidence threshold
- name: PointNovo
  role: Deep-learning de novo predictor whose output is subject to the same per-tool
    precision >= 50% threshold filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:e8edefa1bd6a3de3719ace2e09e85737454a26b0f68721631567b025022ebec7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - Casanovo
  - Novor
  - PointNovo
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# de-novo-confidence-score-filtering

## Summary

Filters the raw de novo peptide predictions produced by tools such as Casanovo, Novor, pNovo 3, DeepNovo, SMSNet or PointNovo at a tool-specific confidence-score threshold chosen so that amino-acid-level precision does not fall below 50%. The filtered prediction set is the required input for downstream de Bruijn assembly (ALPS) of antibody sequences.

## When to use

Use when raw confidence-scored de novo peptide sequencing predictions (e.g. from Casanovo or Novor on antibody MS/MS spectra) must be filtered before assembly, and per-tool precision-score cutoffs are needed.

## When NOT to use

- When peptides were identified by database search rather than de novo sequencing — filtering targets unvalidated de novo predictions, not scored PSMs against a reference database
- When no per-tool precision-vs-score calibration is available: thresholds differ between Casanovo, Novor, PointNovo etc., so a single generic cutoff cannot be transferred between tools

## Inputs

- Confidence-scored de novo peptide predictions from six tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) over multi-enzymatic antibody MS/MS data sets
- Per-tool confidence-score calibration curve (amino-acid-level precision as a function of score threshold)
- Precursor mass error values for each predicted peptide-spectrum match

## Outputs

- Confidence-filtered peptide prediction list per tool (AA-level precision >= 50% at the chosen threshold)
- Precursor-mass-consistent filtered prediction set ready for ALPS de Bruijn assembly (k-mer size 7)

## How to apply

For each de novo sequencing tool, determine its confidence-score threshold at which amino-acid-level precision falls below 50%, and discard predictions scoring below that tool-specific threshold — cutoffs are NOT shared across tools since Casanovo, Novor, PointNovo etc. use different score scales. In addition, apply a precursor mass filter to the predictions: the article notes that a simple precursor mass filter yields much higher precision (an approach shown by the authors of Casanovo). Feed only the confidence-filtered, mass-consistent peptide set into the de Bruijn assembler ALPS (k-mer size 7) rather than the full prediction list, because unfiltered low-confidence calls introduce wrong k-mers that degrade assembled contigs. Validate the filtering step by checking that subsequent light-chain assembly still reaches the reported coverage range (97.69–99.53% with Casanovo) while amino-acid-level accuracy stays at 94.47–95.26%.

## Related tools

- **Casanovo** (Deep-learning de novo predictor whose confidence-scored and precursor-mass-filtered output is the primary candidate set for filtering before ALPS assembly)
- **Novor** (Spectrum-graph-based de novo predictor whose predictions are filtered at its own tool-specific confidence threshold)
- **PointNovo** (Deep-learning de novo predictor whose output is subject to the same per-tool precision >= 50% threshold filtering)

## Evaluation signals

- After filtering, the retained predictions must sit at a threshold where amino-acid-level precision is >= 50% (per tool's own score scale)
- Downstream ALPS assembly (k = 7) of the Casanovo-filtered predictions must reproduce light-chain coverage of 97.69–99.53% with 94.47–95.26% amino-acid accuracy
- Applying the precursor mass filter in addition to the confidence threshold must not reduce coverage below the reported ranges while raising precision
- Heavy-chain assembly with Casanovo-filtered predictions should remain in the lower reported range of 76.39–93.72% coverage, confirming the filter was applied at the same stringency as the study

## Limitations

- Confidence thresholds are tool-specific; a cutoff calibrated on one algorithm cannot be reused on another
- Even after filtering, database-independent full-length assembly is limited, especially on heavy chains, and particular residue pairs (I & L; Q & AG; deamidated N & D) cannot be resolved regardless of filtering stringency
- Missing fragmentation sites, noisy spectra and long peptide sequences limit all de novo tools, so filtering alone cannot recover coverage lost at the spectral level

## Evidence

- [other] peptides filtered at each tool's confidence-score threshold where amino-acid-level precision falls below 50%: [evidence span withheld — anchor status: not_found]
- [discussion] The authors of Casanovo showed how a simple precursor mass filter yields much higher precision: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
- [discussion] advanced post-processing steps are necessary to improve their accuracy: "While newly developed tools demonstrate the potential of de novo sequencing, advanced post-processing steps are necessary to improve their accuracy"
- [other] Load the confidence-filtered de novo peptide predictions produced by task_007: [evidence span withheld — anchor status: not_found]
