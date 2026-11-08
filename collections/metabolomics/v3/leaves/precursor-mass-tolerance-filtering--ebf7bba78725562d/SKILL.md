---
name: precursor-mass-tolerance-filtering
description: Use when implementing de novo peptide prediction from MS/MS spectra and
  you need to constrain beam-search candidates to peptides consistent with the observed
  precursor m/z, accounting for isotope-error offsets.
when_to_use_negative:
- 'Running current Casanovo in de novo mode: the article''s methods notes state ''The
  precursor mass filter is no longer applied in *de novo* mode'', so the filter is
  historical/implementation-specific'
- Spectra without reliable precursor m/z and charge information, since the filter
  cannot be parameterized
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3292
tools:
- name: Casanovo
  role: Transformer-based de novo peptide sequencer whose beam search decoding applies
    precursor mass filtering via precursor_mass_tol and isotope_error_range config
    options
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:22923172ca9a8380066b08c07961a799550362aa579b5e97a04b668a4db188e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# precursor-mass-tolerance-filtering

## Summary

During Casanovo's beam search decoding, candidate peptide beams whose accumulated amino-acid mass exceeds the precursor m/z are pruned, and final predictions are scored against the precursor mass within a configured tolerance across an isotope-error range.

## When to use

Use when implementing de novo peptide prediction from MS/MS spectra and you need to constrain beam-search candidates to peptides consistent with the observed precursor m/z, accounting for isotope-error offsets.

## When NOT to use

- Running current Casanovo in de novo mode: the article's methods notes state 'The precursor mass filter is no longer applied in *de novo* mode', so the filter is historical/implementation-specific
- Spectra without reliable precursor m/z and charge information, since the filter cannot be parameterized

## Inputs

- MS/MS spectra with precursor m/z and charge (e.g. annotated MGF files)
- Casanovo config.yaml specifying precursor_mass_tol and isotope_error_range

## Outputs

- Predicted peptide sequences per spectrum with scores filtered/penalized by precursor mass tolerance
- Beams pruned during decoding whose accumulated mass exceeds the precursor m/z

## How to apply

Load the Casanovo config.yaml to obtain the precursor_mass_tol and isotope_error_range settings that parameterize the mass filter. During autoregressive beam search, expand every retained beam with all candidate tokens and keep the top-k beams by cumulative score; because peptide mass grows monotonically, prune beams whose accumulated amino-acid mass already exceeds the precursor m/z during decoding rather than after termination. Terminate beams at the stop token, break ties randomly among beams with equal predicted peptide scores, and exclude peptides with incorrectly placed N-terminal modifications (multiple or internal positions). Compute the final peptide score as the product of the raw amino acid scores. At the end, admit predictions that fit the precursor mass within tolerance across the configured isotope_error_range offsets and penalize predictions outside tolerance with a negative score. Validate with unit tests on synthetic spectra confirming that only peptides within the precursor tolerance and isotope offsets are reported.

## Related tools

- **Casanovo** (Transformer-based de novo peptide sequencer whose beam search decoding applies precursor mass filtering via precursor_mass_tol and isotope_error_range config options) — https://github.com/Noble-Lab/casanovo

## Evaluation signals

- Unit tests on synthetic spectra confirm only peptides within the precursor tolerance and configured isotope offsets are reported
- No retained beam at decoding termination exceeds the precursor m/z (monotonic-mass pruning invariant)
- Predictions outside the tolerance receive a negative score rather than being silently reported
- Final peptide score equals the product of raw amino acid scores
- Peptides with multiple or internal N-terminal modifications are excluded from reported predictions

## Limitations

- The precursor mass filter is no longer applied in de novo mode in current Casanovo, so the filter behavior is version-dependent
- Predictions outside the precursor tolerance are penalized with a negative score rather than strictly excluded
- Tie-breaking among equal-scoring beams is random, so results may not be fully deterministic

## Evidence

- [methods] The precursor mass filter is no longer applied in *de novo* mode: "The precursor mass filter is no longer applied in *de novo* mode"
- [intro] Beam search added with MassIVE-KB training: "and we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [readme] Casanovo is a de novo sequencing tool: "Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing."
