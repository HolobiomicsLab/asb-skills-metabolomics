---
name: peptide-monoisotopic-mass-computation
description: Use when a predicted peptide sequence from de novo sequencing (e.g. Casanovo
  transformer output) must be checked against the observed precursor m/z within precursor_mass_tol
  across the isotope_error_range, or when beams must be pruned during decoding.
when_to_use_negative:
- Running Casanovo in current de novo mode, where 'The precursor mass filter is no
  longer applied in *de novo* mode' — using it as a hard filter there would misreport
  valid predictions
- When a rigorous spectral match score already exists (e.g. Casanovo-DB mode, 'casanovo
  db_search', where Casanovo acts as a learned score function) and mass filtering
  alone would not rank candidates
- Spectra lacking a reliable precursor m/z or charge annotation, since the tolerance
  and isotope-error comparison cannot be computed
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Casanovo
  role: Transformer de novo sequencing tool whose beam search decoding and precursor
    mass filter rely on monoisotopic peptide mass computation; precursor_mass_tol
    and isotope_error_range come from its config.yaml
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:4e688a40acb4f90286c35dc6c149927b48383d6c860d9c1a3adb99bd2614ba82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# peptide-monoisotopic-mass-computation

## Summary

Computes the monoisotopic mass (or m/z at a given charge) of a predicted peptide sequence so that de novo predictions can be validated against the observed precursor mass within a configured tolerance and isotope-error range, as done in Casanovo's beam search decoding.

## When to use

Use when a predicted peptide sequence from de novo sequencing (e.g. Casanovo transformer output) must be checked against the observed precursor m/z within precursor_mass_tol across the isotope_error_range, or when beams must be pruned during decoding.

## When NOT to use

- Running Casanovo in current de novo mode, where 'The precursor mass filter is no longer applied in *de novo* mode' — using it as a hard filter there would misreport valid predictions
- When a rigorous spectral match score already exists (e.g. Casanovo-DB mode, 'casanovo db_search', where Casanovo acts as a learned score function) and mass filtering alone would not rank candidates
- Spectra lacking a reliable precursor m/z or charge annotation, since the tolerance and isotope-error comparison cannot be computed

## Inputs

- Predicted peptide sequences (amino-acid tokens, optionally with N-terminal modifications) from Casanovo beam search decoding
- Observed precursor m/z and charge from MS/MS spectra (MGF or msconvert-converted input files)
- Casanovo config.yaml with precursor_mass_tol and isotope_error_range settings

## Outputs

- Computed monoisotopic neutral peptide mass (and precursor m/z at the observed charge)
- Pass/fail decision per prediction against the precursor tolerance across isotope offsets, or a negative penalty score for out-of-tolerance predictions

## How to apply

Sum the monoisotopic residue masses of the amino acids in the candidate peptide (plus termini and any placed N-terminal modifications) to obtain the neutral peptide mass, and convert to m/z for the spectrum's charge state. During Casanovo beam search decoding, accumulated amino-acid mass grows monotonically, so any beam whose accumulated mass already exceeds the precursor mass can be discarded immediately rather than being scored further. After decoding, apply the precursor m/z tolerance filter: admit predictions whose computed mass fits the precursor within precursor_mass_tol when accounting for isotope offsets in the configured isotope_error_range (e.g. charge-1 vs isotope-shifted interpretations), and penalize predictions outside the tolerance with a negative score. Load precursor_mass_tol and isotope_error_range from the Casanovo default config.yaml rather than hard-coding them. Note that the peptide score is the product of the raw amino-acid scores, so the mass filter acts as an additional consistency check on candidate peptides rather than the primary ranking signal.

## Related tools

- **Casanovo** (Transformer de novo sequencing tool whose beam search decoding and precursor mass filter rely on monoisotopic peptide mass computation; precursor_mass_tol and isotope_error_range come from its config.yaml) — https://github.com/Noble-Lab/casanovo

## Evaluation signals

- Unit tests on synthetic spectra confirm that only peptides whose computed mass fits the precursor within tolerance across the isotope_error_range offsets are reported
- Beams whose accumulated amino-acid mass exceeds the precursor m/z are pruned during decoding (monotonic-mass invariant)
- Predictions outside the precursor tolerance receive a negative score rather than being silently retained
- Peptides with incorrectly placed N-terminal modifications (multiple or internal positions) are excluded before mass scoring
- Ties among beams with equal predicted peptide scores are broken randomly, so repeated runs should vary only among equal-score candidates

## Limitations

- The monotonic-mass pruning assumption only holds while mass is added one amino acid at a time during decoding; modifications with negative or unusual mass shifts need care
- The field lacks a rigorous method for confidence estimation for de novo sequencing, so a passing mass filter is a weak correctness signal on its own
- Version-dependent behavior: recent Casanovo releases no longer apply the precursor mass filter in de novo mode, so the filter is mode- and version-specific

## Evidence

- [intro] we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum.: "we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [methods] The precursor mass filter is no longer applied in *de novo* mode: "The precursor mass filter is no longer applied in *de novo* mode"
- [methods] Beam search decoding has been optimized for computational efficiency: "Beam search decoding has been optimized for computational efficiency"
- [readme] Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing.: "Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing."
