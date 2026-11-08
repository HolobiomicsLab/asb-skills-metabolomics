---
name: neutral-loss-removal-from-pseudo-spectra
description: Use when Mass2Motif pseudo-spectra containing both fragment and neutral-loss
  entries must be fed to a spectral-matching-based annotation tool (e.g. MS2Analyzer-style
  matching) that only accepts fragments.
when_to_use_negative:
- When the annotation tool natively supports neutral losses — the removal is a workaround
  because 'losses are not currently supported by spectral matching tools', not an
  inherent requirement
- When the goal is motif discovery itself (unsupervised LDA extraction of fragments
  and neutral losses) rather than automated annotation of already-characterized motifs
- When pseudo-spectra are already fragment-only or come from negative-ionization-mode
  or non-beer matrices, which were not part of the reported validation
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MS2LDA
  role: Provides the Mass2Motif representation whose motif spectra are the object
    of the loss-removal step, and hosts the automated annotation/spectral-matching
    procedure applied afterwards
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: Reference substructure-matching style used for the automated annotation of
    the fragment-only motif spectra
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:474484f3a5eb4f313cad8eba953c6bf83c71ddaefb19c699d6340e4d11af5929
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2Analyzer
  - MS2LDA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# neutral-loss-removal-from-pseudo-spectra

## Summary

Removes neutral-loss entries from Mass2Motif pseudo-spectra so they can be subjected to spectral matching for automated substructure annotation, because spectral matching tools do not support losses. This adaptation is what enabled 19 of the structurally characterized positive-mode beer Mass2Motifs to be correctly annotated.

## When to use

Use when Mass2Motif pseudo-spectra containing both fragment and neutral-loss entries must be fed to a spectral-matching-based annotation tool (e.g. MS2Analyzer-style matching) that only accepts fragments.

## When NOT to use

- When the annotation tool natively supports neutral losses — the removal is a workaround because 'losses are not currently supported by spectral matching tools', not an inherent requirement
- When the goal is motif discovery itself (unsupervised LDA extraction of fragments and neutral losses) rather than automated annotation of already-characterized motifs
- When pseudo-spectra are already fragment-only or come from negative-ionization-mode or non-beer matrices, which were not part of the reported validation

## Inputs

- Mass2Motif pseudo-spectra (fragment + neutral-loss entries) from structurally characterized positive-mode beer data (deposited beer dataset, dx.doi.org/10.5525/gla.researchdata.313)
- Manual substructure annotations for the 30-40 characterized positive-ionization-mode beer Mass2Motifs

## Outputs

- Fragment-only Mass2Motif pseudo-spectra (neutral-loss entries removed)
- Per-motif agreement table (motif ID, manual annotation, automated annotation, probability score, agreement flag), e.g. motif_annotation_agreement.csv
- Tally of correctly annotated motifs (expected: 19)

## How to apply

Load the Mass2Motif pseudo-spectra from the target dataset (e.g. the deposited beer data at dx.doi.org/10.5525/gla.researchdata.313) in the MS2LDA motif representation, where each motif is a set of co-occurring molecular fragments and neutral losses. Since losses are not supported by spectral matching tools, delete every neutral-loss entry from each pseudo-spectrum, keeping only the fragment peaks. Run automated spectral matching of each fragment-only motif spectrum using the MS2LDA annotation procedure (MS2Analyzer-style substructure matching) and apply the 0.1 probability threshold the paper used when validating substructure annotations (where 81.5% and 63.3% of substructure annotations met the threshold). Compare each automated annotation against the manually characterized substructure label for the 30-40 structurally characterized positive-ionization-mode beer motifs, mark agreement per motif, tally correct annotations, and write a per-motif table (motif ID, manual annotation, automated annotation, probability score, agreement flag). The rationale is that the motif itself can be represented as a spectrum and subjected to spectral matching, but only after loss entries are dropped.

## Related tools

- **MS2LDA** (Provides the Mass2Motif representation whose motif spectra are the object of the loss-removal step, and hosts the automated annotation/spectral-matching procedure applied afterwards) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Reference substructure-matching style used for the automated annotation of the fragment-only motif spectra)

## Evaluation signals

- Tally of correctly annotated motifs equals 19, matching the reported outcome in SI Appendix section S2.13
- Per-motif agreement table covers the 30-40 structurally characterized positive-mode beer Mass2Motifs with one row per motif
- No residual neutral-loss entries (negative mass shifts) remain in any filtered pseudo-spectrum
- Each automated annotation carries a probability score, and annotation calls are restricted to those passing the 0.1 probability threshold
- Per-motif agreement flags are consistent between the manual substructure labels and the automated spectral-matching results

## Limitations

- Removing neutral losses discards part of each motif's biochemical signature, since Mass2Motifs are defined as sets of co-occurring fragments AND neutral losses; the omission is forced by tool limitations
- Automated Mass2Motif annotation via this route was validated only on characterized positive-mode beer motifs; other ionization modes and matrices are not reported
- MS2LDA itself extracts and uses neutral losses (e.g. for biochemically relevant motif characterization), so loss removal applies only at the spectral-matching/annotation stage, not during motif extraction

## Evidence
