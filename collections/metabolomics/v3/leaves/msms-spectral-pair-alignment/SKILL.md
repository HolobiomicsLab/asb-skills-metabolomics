---
name: msms-spectral-pair-alignment
description: Use when a pair of tandem MS spectra from structurally related analogs
  must be peak-aligned to identify which fragment peaks are shifted between the MS2-known
  and MS2-unknown spectra, prior to modification-site localization or analog-pair
  filtering.
when_to_use_negative:
- You have only a single MS/MS spectrum with no structurally related analog partner
  — alignment requires two spectra from a known-unknown analog pair
- The two spectra come from compounds with the same precursor mass (no mass difference),
  so no shifted peaks can exist to inform localization
- You need library identification of a single compound against a reference database
  — use MS/MS library search or molecular networking instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: Implements the bipartite-graph greedy matching used to align MS2-known and
    MS2-unknown peaks; the alignment output feeds ModiFinder's substructure scoring
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:6e74d81e15b5ccae8b905461212cc3348b6b521abf2cfcb2e9fccc4970ad5824
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# msms-spectral-pair-alignment

## Summary

Aligns MS2 peaks between a structurally related known-unknown analog pair (MS2-known vs MS2-unknown) using a bipartite-graph greedy matching at 40 ppm m/z tolerance, producing matched shifted and unshifted peak sets that are the prerequisite input for ModiFinder's modification-site localization scoring.

## When to use

Use when a pair of tandem MS spectra from structurally related analogs must be peak-aligned to identify which fragment peaks are shifted between the MS2-known and MS2-unknown spectra, prior to modification-site localization or analog-pair filtering.

## When NOT to use

- You have only a single MS/MS spectrum with no structurally related analog partner — alignment requires two spectra from a known-unknown analog pair
- The two spectra come from compounds with the same precursor mass (no mass difference), so no shifted peaks can exist to inform localization
- You need library identification of a single compound against a reference database — use MS/MS library search or molecular networking instead

## Inputs

- MS2-known spectrum (peak list of m/z and intensity) for the structurally characterized analog
- MS2-unknown spectrum (peak list of m/z and intensity) for the analog to be localized
- Precursor mass difference between the analog pair (used to classify matched peaks as shifted vs unshifted)

## Outputs

- Matched peak list partitioned into shifted peaks and unshifted peaks, each associated with both spectra
- Unmatched peaks in either spectrum

## How to apply

Load the MS2-known and MS2-unknown spectra of an analog pair; first apply peak preprocessing by removing all peaks with intensities less than 1% of the base peak and normalizing peaks to sum to a Euclidean norm of 1. Then construct a bipartite graph where the nodes represent the peaks of MS2-known and MS2-unknown respectively, with an edge between two peaks if their m/z values are within the 40 ppm m/z tolerance. Apply a greedy matching over this bipartite graph to produce two matched peak classes: shifted peaks (m/z difference consistent with the known precursor mass difference) and unshifted peaks. These matched shifted/unshifted peak sets are subsequently consumed by ModiFinder's substructure-assignment step, where shifted peaks increase likelihood scores of atoms in assigned substructures and unshifted peaks decrease them.

## Related tools

- **ModiFinder** (Implements the bipartite-graph greedy matching used to align MS2-known and MS2-unknown peaks; the alignment output feeds ModiFinder's substructure scoring) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
python -m modi_finder.align --known known.mgf --unknown unknown.mgf --mz-tolerance 40
```

## Evaluation signals

- Verify that peaks with intensities < 1% of the base peak have been removed before matching
- Verify that peaks are normalized to sum to a Euclidean norm of 1
- Check that every matched peak pair falls within the 40 ppm m/z tolerance
- Confirm the matched peak sets are partitioned into shifted and unshifted classes relative to the precursor mass difference
- Downstream check: shifted peaks should correspond to substructures of the known molecule that contain the site of the modification when substructure annotation is applied

## Limitations

- Alignment quality is bounded by the 40 ppm tolerance choice, which may miss peaks at the tolerance boundary or mism atch close-lying peaks
- Peaks with intensity below 1% of the base peak are discarded, so low-intensity but informative fragment peaks may be lost
- The greedy matching may not produce a globally optimal assignment over the bipartite graph

## Evidence

- [readme] readme role description: [evidence span withheld — anchor status: not_found]
