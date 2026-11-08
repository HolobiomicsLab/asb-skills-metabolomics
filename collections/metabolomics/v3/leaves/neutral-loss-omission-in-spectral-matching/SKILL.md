---
name: neutral-loss-omission-in-spectral-matching
description: Use when decomposing external reference MS2 spectra (Mass Bank, GNPS)
  with pre-characterized Mass2Motifs and running motif-based spectral library matching,
  since loss fields are unsupported by matching tools.
when_to_use_negative:
- A spectral matching tool that natively supports neutral losses is available — the
  omission is a workaround, not a principled choice
- The goal is de novo substructure discovery on your own complex-mixture data rather
  than validating already-characterized motifs against reference libraries
- The MS2 spectra are information-poor (single collision energy, few fragments per
  metabolite) where MS2LDA itself works poorly
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Extracts the Mass2Motifs whose fragment sets are converted to pseudo-spectra
    for matching
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: Mass Bank
  role: External reference spectral library used to validate motif annotations
- name: GNPS
  role: External reference spectral library used to validate motif annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:3628a076af87fd80ac615652eeb1ac07301abd09c46cfb8da72262774854c38a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - GNPS
  - MS2LDA
  - Mass Bank
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Neutral-loss omission in spectral matching

## Summary

When validating MS2LDA Mass2Motifs against external spectral libraries such as Mass Bank and GNPS, co-occurring fragments of a motif are used as peaks but the motif's neutral losses are omitted, because spectral matching tools do not support losses. This skill prescribes that omission plus the 90% normalized-score acceptance threshold and the 0.1 probability cutoff used to validate transferred substructure annotations.

## When to use

Use when decomposing external reference MS2 spectra (Mass Bank, GNPS) with pre-characterized Mass2Motifs and running motif-based spectral library matching, since loss fields are unsupported by matching tools.

## When NOT to use

- A spectral matching tool that natively supports neutral losses is available — the omission is a workaround, not a principled choice
- The goal is de novo substructure discovery on your own complex-mixture data rather than validating already-characterized motifs against reference libraries
- The MS2 spectra are information-poor (single collision energy, few fragments per metabolite) where MS2LDA itself works poorly

## Inputs

- Structurally characterized Mass2Motifs from MS2LDA (fragment and neutral-loss probabilities per motif)
- External reference MS2 spectra from Mass Bank and GNPS (as provided with the CSI:FingerID paper)
- Preprocessed MS2 peak lists (around 1,000 MS peaks per sample in positive and negative ionization mode)

## Outputs

- Per-motif pseudo-spectra with neutral losses removed
- CSV validation table with matched library, hit score, and matched reference per motif
- Counts of spectra carrying at least one validated substructure annotation (694 Mass Bank; 613 GNPS)
- Validated fraction of substructure annotations per library at probability threshold 0.1

## How to apply

Represent each structurally characterized Mass2Motif as a pseudo-spectrum in which its co-occurring fragments (retained at a probability threshold of 0.1) act as peaks; explicitly drop the motif's neutral-loss entries because 'losses are not currently supported by spectral matching tools and had to be omitted'. Load the GNPS and Mass Bank reference spectra (provided by the CSI:FingerID authors) and perform spectral matching of each motif pseudo-spectrum against the libraries, applying the 90% normalized score threshold the paper uses for spectral library matching to accept hits. For the validation of transferred annotations, keep the beer-characterized Mass2Motifs fixed during LDA inference while other motifs are learned, then count a substructure annotation as validated at the probability threshold of 0.1. Record per-motif match status (matched library, hit score, matched reference) in a CSV validation table; report the fraction of validated annotations separately for Mass Bank (expected ~81.5%) and GNPS (expected ~63.3%).

## Related tools

- **MS2LDA** (Extracts the Mass2Motifs whose fragment sets are converted to pseudo-spectra for matching) — https://github.com/vdhooftcompmet/MS2LDA
- **Mass Bank** (External reference spectral library used to validate motif annotations)
- **GNPS** (External reference spectral library used to validate motif annotations)

## Examples

```
# pseudo-spectrum per motif: peaks = fragments with prob >= 0.1; losses omitted
motif_ps = {m: {f: p for f, p in motif['fragments'].items() if p >= 0.1} for m in motifs}
```

## Evaluation signals

- Neutral losses are demonstrably absent from the pseudo-spectra submitted to the matcher, while fragment probabilities were filtered at 0.1
- Every motif hit satisfies the 90% normalized score threshold before being counted as validated
- The fraction of validated annotations is consistent with the published benchmark (81.5% for Mass Bank, 63.3% for GNPS; 19 positive-mode beer motifs correctly annotated)
- 694 and 613 spectra respectively carry at least one validated substructure annotation
- Validation runs use the beer-characterized motifs kept fixed during LDA inference, as the paper prescribes

## Limitations

- Neutral losses are not currently supported by spectral matching tools and had to be omitted, losing part of each motif's information
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other modes/matrices are not reported
- Matching tools return hits for only a fraction of spectra: the paper's own library matching obtained hits for only 25 and 6% of spectra at the same 90% threshold
- MS2LDA works best on complex mixtures with information-rich MS2 spectra (ramped or stepped collision energy)

## Evidence
