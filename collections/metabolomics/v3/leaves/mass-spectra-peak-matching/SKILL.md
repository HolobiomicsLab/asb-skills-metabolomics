---
name: mass-spectra-peak-matching
description: Use when comparing a query tandem mass spectrum against reference spectra
  (e.g., ESI-MS2 peak lists of authentic compounds) to produce a similarity score
  for compound identification, as in MassBank's Spectral Search.
when_to_use_negative:
- Comparing EI-MS spectra, where Stein and Scott's exponents (m = 0.6, n = 3) rather
  than the ESI-MS2-optimized m = 0.5, n = 2 are appropriate
- Input is raw instrument data (e.g., mzML/profile spectra) rather than peak lists
  — centroiding and peak picking must happen first
- Need for identity-level confirmation rather than similarity-based ranking; a score
  above 0.6 alone does not guarantee correct identification
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassBank3
  role: MassBank spectral reference library web app and REST API whose Spectral Search
    uses this weighted cosine correlation score
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-cli-tools
  role: Command line tools including the MassBank record validator; used to check
    reference records before matching
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:b72afa505eb5f9c1c463f1661377c027af3581832cbd27e27a902e37b3526869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank-cli-tools
  - MassBank3
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-spectra-peak-matching

## Summary

Compute the MassBank spectral search similarity score: a weighted cosine correlation between query and reference peak lists in which each peak is weighted by W_i = [Intensity_i]^m [m/z_i]^n, with exponents optimized for ESI-MS2 data as m = 0.5 and n = 2.

## When to use

Use when comparing a query tandem mass spectrum against reference spectra (e.g., ESI-MS2 peak lists of authentic compounds) to produce a similarity score for compound identification, as in MassBank's Spectral Search.

## When NOT to use

- Comparing EI-MS spectra, where Stein and Scott's exponents (m = 0.6, n = 3) rather than the ESI-MS2-optimized m = 0.5, n = 2 are appropriate
- Input is raw instrument data (e.g., mzML/profile spectra) rather than peak lists — centroiding and peak picking must happen first
- Need for identity-level confirmation rather than similarity-based ranking; a score above 0.6 alone does not guarantee correct identification

## Inputs

- Query spectrum peak list as (m/z, intensity) pairs
- Reference spectrum peak list as (m/z, intensity) pairs, e.g., from MassBank ESI-MS2 records of authentic compounds

## Outputs

- Scalar similarity score in [0, 1] from the weighted cosine correlation

## How to apply

Load the query and reference peak lists as arrays of (m/z, intensity) pairs. Compute the weighted value W_i = [Intensity_i]^0.5 * [m/z_i]^2 for every peak in both spectra — these exponents were optimized on 8785 ESI-MS2 spectra of ca. 700 authentic primary metabolites and differ from Stein and Scott's EI-MS exponents (m = 0.6, n = 3). Align query peaks to reference peaks on m/z so matched peaks form paired components of the weighted vectors and unmatched peaks contribute zero. Compute the cosine correlation: dot product of the weighted vectors divided by the product of their norms, yielding a scalar in [0, 1]. Interpret scores against MassBank's reported identification threshold of 0.6, where merging ESI-MS2 data under different collision-induced dissociation conditions improved identification precision by 21-23%.

## Related tools

- **MassBank3** (MassBank spectral reference library web app and REST API whose Spectral Search uses this weighted cosine correlation score) — https://github.com/MassBank/MassBank3
- **MassBank-cli-tools** (Command line tools including the MassBank record validator; used to check reference records before matching) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
score = weighted_cosine(query_peaks, ref_peaks, m=0.5, n=2)  # assert 0.6 <= score for identification candidate
```

## Evaluation signals

- Identical query and reference peak lists must yield a score of exactly 1.0
- All returned scores must fall in the range [0, 1]
- Score scale must be consistent with MassBank's reported identification threshold of 0.6
- Changing a single unmatched peak should leave the matched-pair contribution unchanged but lower the score via the vector norms (zero-contribution check)
- Exponents used must be m = 0.5 and n = 2 for ESI-MS2, not m = 0.6 and n = 3 (EI-MS)

## Limitations

- Exponents m = 0.5 and n = 2 were optimized specifically for ESI-MS2 data of authentic compounds (8785 spectra of ca. 700 primary metabolites) and may not transfer to EI-MS or other ionization modes
- Identification precision is only ~21-23% improved by merging ESI-MS2 data under different collision-induced dissociation conditions at a similarity score of 0.6; the threshold is not a guarantee of correct identification
- Unmatched peaks contribute zero, so spectra with few shared peaks can still yield misleadingly moderate scores depending on norms

## Evidence

- [abstract] weighted cosine correlation with ESI-MS2 exponents: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [abstract] identification threshold 0.6 and merging benefit: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [readme] current MassBank software location: "The new MassBank web app is maintained in repo https://github.com/MassBank/MassBank3."
