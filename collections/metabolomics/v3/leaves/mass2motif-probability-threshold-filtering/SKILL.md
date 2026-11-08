---
name: mass2motif-probability-threshold-filtering
description: Use when filtering MS2LDA decomposition output where each spectrum–Mass2Motif
  link carries a probability and you must decide which memberships count as validated
  substructure annotations, e.g. before cross-library validation against GNPS or Mass
  Bank spectra.
when_to_use_negative:
- Input is already a curated motif-annotation table where probabilities are no longer
  available
- The task is de novo motif discovery rather than filtering existing memberships
- Working with data-independent acquisition fragments that lack reliable MS1-to-MS2
  matching, since Mass2Motif membership depends on that linkage
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: Produces the Mass2Motif membership probabilities being filtered and the motif
    pseudo-spectra used in validation
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: GNPS
  role: External reference spectral library used to validate filtered substructure
    annotations
- name: Mass Bank
  role: External reference spectral library used to validate filtered substructure
    annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:c95967e422a5dfca1ce6de8342d42cc82cc24f6ca179d440cb5dff0e8f66de5f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - GNPS
  - MS2LDA
  - Mass Bank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass2motif-probability-threshold-filtering

## Summary

Applying the MS2LDA paper's probability threshold of 0.1 to decide which substructure annotations (Mass2Motif-to-spectrum memberships) are accepted as validated, enabling benchmarking against GNPS and Mass Bank reference spectra and selective retention of motif links.

## When to use

Use when filtering MS2LDA decomposition output where each spectrum–Mass2Motif link carries a probability and you must decide which memberships count as validated substructure annotations, e.g. before cross-library validation against GNPS or Mass Bank spectra.

## When NOT to use

- Input is already a curated motif-annotation table where probabilities are no longer available
- The task is de novo motif discovery rather than filtering existing memberships
- Working with data-independent acquisition fragments that lack reliable MS1-to-MS2 matching, since Mass2Motif membership depends on that linkage

## Inputs

- MS2LDA decomposition output (spectrum-to-Mass2Motif probability memberships)
- Structurally characterized Mass2Motifs as pseudo-spectra of co-occurring fragments and neutral losses
- Reference spectra from GNPS and Mass Bank (e.g. the spectra used in the CSI:FingerID paper)

## Outputs

- Filtered set of Mass2Motif-spectrum memberships retained at probability ≥ 0.1
- Validation table (CSV) of per-motif match status, hit score, and matched reference library
- Counts of spectra carrying at least one validated substructure annotation and the percentage of validated annotations per library

## How to apply

After running LDA inference (e.g. variational Bayesian inference decomposing the GNPS dataset of 5,670 spectra into 500 Mass2Motifs), inspect the probability of each Mass2Motif membership per spectrum and retain only links with probability ≥ 0.1. When validating structurally characterized motifs, represent each motif as a pseudo-spectrum of its co-occurring fragments (also retained at probability 0.1) and match against GNPS/Mass Bank reference libraries, omitting neutral losses since spectral matching tools do not support them; apply the paper's 90% normalized score threshold for accepting spectral library hits. At the 0.1 probability threshold the paper reports 81.5% of substructure annotations validated on Mass Bank and 63.3% on GNPS, with 694 and 613 spectra respectively carrying at least one validated annotation, so these values serve as the expected operating point. Record per-motif match status (matched library, hit score, matched reference) in a validation table such as CSV.

## Related tools

- **MS2LDA** (Produces the Mass2Motif membership probabilities being filtered and the motif pseudo-spectra used in validation) — https://github.com/vdhooftcompmet/MS2LDA
- **GNPS** (External reference spectral library used to validate filtered substructure annotations)
- **Mass Bank** (External reference spectral library used to validate filtered substructure annotations)

## Examples

```
from MS2LDA import LDA; # filter memberships at probability 0.1 before pseudo-spectral matching against Mass Bank/GNPS
```

## Evaluation signals

- Fraction of validated substructure annotations at probability threshold 0.1 should fall near the reported 81.5% (Mass Bank) and 63.3% (GNPS)
- Number of spectra with at least one validated annotation should be on the order of 694 (Mass Bank) and 613 (GNPS) for the characterized positive-mode beer motifs
- Validation table rows must carry a matched library, a hit score at or above the 90% normalized score threshold, and a matched reference identifier
- Motif pseudo-spectra used in matching must contain only fragments retained at the 0.1 probability threshold, with neutral losses omitted

## Limitations

- Neutral losses cannot be included in automated annotation because spectral matching tools do not support them
- Validation coverage is limited to the 19 characterized positive-mode beer Mass2Motifs; other modes or matrices are not reported
- Mass2LDA works best on complex mixtures with information-rich MS2 spectra (e.g. ramped or stepped collision energy) and relies on reliable matching of MS1 peaks to MS2 spectra

## Evidence

- [discussion] LDA decomposition scale on GNPS data: [evidence span withheld — anchor status: not_found]
