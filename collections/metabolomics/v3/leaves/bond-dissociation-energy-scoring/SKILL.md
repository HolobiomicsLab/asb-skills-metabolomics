---
name: bond-dissociation-energy-scoring
description: Use when ranking candidate small-molecule structures against a high-resolution
  MS/MS peak list requires fragment matching scores weighted by bond dissociation
  energies of cleaved bonds.
when_to_use_negative:
- The measured spectrum is dominated by one or only a few fragments (e.g. a water
  loss) explainable by most candidates — it lacks the information content needed to
  discriminate candidates
- The goal is spectral-library matching against reference spectra (e.g. MassBank)
  rather than in silico fragmentation of database candidates
- Candidates come as already-scored feature tables without structure-to-fragment assignments,
  so BDE-weighted rescoring cannot be applied
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MetFrag2.2
  role: Performs the bond dissociation fragmentation of candidates and computes the
    BDE-weighted fragment score used to rank candidates against the HR-MS/MS peak
    list
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragCommandLine
  role: Command-line entry point of the relaunched MetFrag that depends on MetFragLib
    and executes the parameterized fragmentation/scoring run
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: metfRag (MetFragR)
  role: R wrapper around the MetFragLib java library for invoking in silico fragmentation
    and scoring from R workflows
  repo: https://github.com/ipb-halle/MetFragR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:984b47f08a728e62c9f67f5c18b69c13fa456b6a6fca77d0350ba2d0a3ac948d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag2.2
  - MetFragCommandLine
  - metfRag (MetFragR)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bond-dissociation-energy-scoring

## Summary

Computes the MetFrag2.2 in silico fragmentation score (S_C_Frag) that ranks candidate structures against an HR-MS/MS spectrum by summing matched-peak contributions of relative mass^alpha x relative intensity^beta, each divided by the summed bond dissociation energy (BDE) of the cleaved bonds raised to the power gamma, with optimized exponents alpha=1.84, beta=0.59, gamma=0.47.

## When to use

Use when ranking candidate small-molecule structures against a high-resolution MS/MS peak list requires fragment matching scores weighted by bond dissociation energies of cleaved bonds.

## When NOT to use

- The measured spectrum is dominated by one or only a few fragments (e.g. a water loss) explainable by most candidates — it lacks the information content needed to discriminate candidates
- The goal is spectral-library matching against reference spectra (e.g. MassBank) rather than in silico fragmentation of database candidates
- Candidates come as already-scored feature tables without structure-to-fragment assignments, so BDE-weighted rescoring cannot be applied

## Inputs

- Measured HR-MS/MS product-ion peak list (relative mass and relative intensity per peak)
- Candidate structure list (SDF upload or candidates retrieved from PubChem, ChemSpider or KEGG)
- Per-candidate in silico fragments with recorded bond dissociation energies (BDEs) of cleaved bonds
- Parameter file setting exponents alpha, beta, gamma (1.84, 0.59, 0.47) and candidate scoring weights (omegaFrag + omegaRT + omegaRefs = 1)

## Outputs

- Normalized fragment scores per candidate (range [0,1], top candidate exactly 1)
- Per-peak match assignments mapping measured product ions to in silico fragments
- Candidate ranking consistent with the deposited result and ranking files

## How to apply

Fragment each candidate in silico with the MetFrag2.2 bond dissociation approach, recording the BDEs of every cleaved bond per generated fragment. Match the generated fragments against the product-ion peaks of the measured HR-MS/MS spectrum; for each matched peak compute (relative peak mass)^alpha x (relative intensity)^beta divided by (summed BDE of cleaved bonds)^gamma. When several fragments explain the same peak, keep only the fragment with the lowest denominator value. Sum over all matched peaks, then normalize all candidate scores so the top-scoring candidate receives 1, using the optimized weights alpha=1.84, beta=0.59, gamma=0.47. The rationale is that a match requiring less energetically demanding bond cleavages (lower summed BDE) is more plausible, sharpening discrimination beyond raw mass agreement.

## Related tools

- **MetFrag2.2** (Performs the bond dissociation fragmentation of candidates and computes the BDE-weighted fragment score used to rank candidates against the HR-MS/MS peak list) — http://c-ruttkies.github.io/MetFrag/
- **MetFragCommandLine** (Command-line entry point of the relaunched MetFrag that depends on MetFragLib and executes the parameterized fragmentation/scoring run) — https://github.com/ipb-halle/MetFragRelaunched
- **metfRag (MetFragR)** (R wrapper around the MetFragLib java library for invoking in silico fragmentation and scoring from R workflows) — https://github.com/ipb-halle/MetFragR

## Examples

```
mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- All normalized S_C_Frag scores fall in [0, 1] and the top-scoring candidate receives exactly 1
- The produced candidate ranking matches the result and ranking files deposited in the dataset archive (EA, UF, EQEx, EQExPlus, HILL)
- For every peak matched by multiple fragments, the retained fragment has the lowest denominator value (summed BDE)^gamma among the competing matches
- The scoring weights (omegaFrag + omegaRT + omegaRefs) sum to 1, as enforced in the weight-optimization procedure drawing 1000 random combinations
- Fragment-explained peaks include at least one fragment peak other than the precursor, per the retained-spectra criterion

## Limitations

- Mass spectra dominated by one or only a few fragments that can be explained by most candidates simply do not contain enough information to distinguish candidates
- The optimized parameters and weights were verified on three additional datasets of 824 merged HR-MS/MS spectra in total; performance on unseen data types is not established
- Spectra lacking any fragment peak other than the precursor are not retained, limiting applicability to near-pure precursor spectra

## Evidence

- [methods] S_C_Frag score sums over matched peaks p, multiplying relative mass to the power alpha and relative intensity to the power beta: [evidence span withheld — anchor status: not_found]
- [intro] Candidates are then fragmented using a bond dissociation approach: "Candidates are then fragmented using a bond dissociation approach"
- [intro] these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data: "these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data"
- [results] the combination of each scoring term was assessed by randomly drawing 1000 different weight combinations such that (ωFrag+ωRT+ωRefs=1): [evidence span withheld — anchor status: not_found]
- [readme] MetFrag commandline version depends on MetFragLib: "MetFrag commandline version depends on MetFragLib"
