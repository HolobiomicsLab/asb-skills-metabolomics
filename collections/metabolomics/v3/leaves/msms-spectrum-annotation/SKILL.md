---
name: msms-spectrum-annotation
description: Use when given a precursor peak with its MS/MS fragment peak list (composite
  spectra across collision energies) and no reference spectra, and candidate structures
  must be ranked by how well each explains the observed fragments.
when_to_use_negative:
- When informative reference spectra are available for the unknown — library matching,
  not in silico substructure annotation, is appropriate
- When candidate generation was restricted to <1200 Da or element set C,H,N,O,P,S
  but the unknown falls outside that chemical space — the correct structure may be
  absent from the candidate list
- When fragment peaks require more than 3 bond dissociations (as in challenge 16,
  where higher-energy peaks remained unexplained with the default fragmentation level)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MAGMa
  role: 'Core engine: converts peak lists to input files, fragments candidate molecules
    in silico (up to 3 bond dissociations plus hydroxyl/amine losses), matches MS/MS
    peaks to substructures, and computes penalty scores for ranking'
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules, retrieved in SDF format from a local snapshot
    (2 December 2013) filtered by elemental composition and mass; refscore used to
    order/filter candidates
- name: emetabolomics web interface
  role: Alternative to command line mode for processing input data and viewing MAGMa
    results (www.emetabolomics.org/magma)
  repo: http://www.emetabolomics.org/results_CASMI2013.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:a4a73f887d00be96a4939a77f3e316dfb7949e31b772de06c8b79c95a3e5ffd8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# msms-spectrum-annotation

## Summary

Annotate MS/MS fragment peaks by matching them against in silico generated substructures of candidate molecules (from PubChem), compute intensity-weighted penalty scores, and rank candidates by reversed penalty score. Used to solve CASMI 2013 contest challenges without reference spectra.

## When to use

Use when given a precursor peak with its MS/MS fragment peak list (composite spectra across collision energies) and no reference spectra, and candidate structures must be ranked by how well each explains the observed fragments.

## When NOT to use

- When informative reference spectra are available for the unknown — library matching, not in silico substructure annotation, is appropriate
- When candidate generation was restricted to <1200 Da or element set C,H,N,O,P,S but the unknown falls outside that chemical space — the correct structure may be absent from the candidate list
- When fragment peaks require more than 3 bond dissociations (as in challenge 16, where higher-energy peaks remained unexplained with the default fragmentation level)

## Inputs

- MS/MS peak lists from the CASMI contest (composite spectra merging peaks from different collision energies, taking the highest intensity for ions present at both energies)
- MAGMa input files in hierarchical m/z:intensity parenthesis format (extensible to multistage MSn spectral trees)
- Candidate molecular structures retrieved from PubChem in SDF format (local database, compounds on 2 December 2013)

## Outputs

- Per-candidate total penalty scores (intensity-weighted average of substructure penalties, penalty of 10 per non-matched peak)
- Ranked candidate list with reversed ranking values, ties ordered by PubChem refscore (e.g. magma_challenge11_0.005da_ranking.csv)
- Per-peak substructure annotations with best scoring substructure assignments and penalty values

## How to apply

Convert the provided peak lists into the MAGMa hierarchical m/z:intensity parenthesis input format, then run MAGMa in command line mode over candidates retrieved from a local PubChem database (compounds on 2 December 2013; elements C,H,N,O,P,S, up to 1200 Da). Generate substructures by applying up to 3 bond dissociations, expanded with single losses of hydroxyl (neutral water loss) or primary amine groups (neutral ammonia loss). Match each MS/MS peak m/z with its best scoring substructure using a mass deviation of 10 ppm with an absolute minimum of 0.002 Da (4 ppm for retrieving candidates in challenges 4, 5, and 6; relax to a 0.005 Da minimum tolerance if peaks like m/z 119.0538, 143.0534, 158.0396 remain unexplained). Compute each candidate's total penalty score as the intensity-weighted average of substructure penalties plus a penalty of 10 for non-matched peaks, rank candidates by reversed penalty score, and order equally scoring candidates by their PubChem refscore. For very large candidate lists (challenges 1, 2, 14), apply a refscore threshold of 5, or stepwise raise it to reduce sets to <1000, <500, and <200 molecules.

## Related tools

- **MAGMa** (Core engine: converts peak lists to input files, fragments candidate molecules in silico (up to 3 bond dissociations plus hydroxyl/amine losses), matches MS/MS peaks to substructures, and computes penalty scores for ranking) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules, retrieved in SDF format from a local snapshot (2 December 2013) filtered by elemental composition and mass; refscore used to order/filter candidates)
- **emetabolomics web interface** (Alternative to command line mode for processing input data and viewing MAGMa results (www.emetabolomics.org/magma)) — http://www.emetabolomics.org/results_CASMI2013.html

## Examples

```
magma --job challenge11.msms --candidates pubchem_2013-12-02.sdf --mass-tolerance 10ppm --min-tolerance 0.005Da --max-bond-dissociations 3
```

## Evaluation signals

- Correct structure ranked 1st in the reversed penalty-score ranking (achieved for 6 of 12 submitted CASMI category 2 solutions); all top-ranked category 1 molecular formulas correct
- All or nearly all MS/MS peaks matched to substructures — unexplained peaks at 10 ppm/0.002 Da tolerance (e.g. m/z 119.0538, 143.0534, 158.0396 in challenge 11) should be re-matched at 0.005 Da minimum tolerance
- Candidate set reduction traceable: refscore threshold stepwise raised until candidate sets are <1000, <500, and <200 molecules
- Correct candidate confirmed present in the initial PubChem candidate set before attributing a poor rank to tolerance or fragmentation parameters
- Full ranked output persisted (e.g. magma_challenge11_0.005da_ranking.csv) and reproducible via the published results at emetabolomics.org/results_CASMI2013.html

## Limitations

- Success requires inclusion of the relevant candidate molecules, appropriate mass tolerance, and sufficient in silico fragmentation — a failure can stem from any of these, not just absence of the true structure
- Default of up to 3 bond dissociations was insufficient for challenge 16, where several higher-energy peaks (m/z 122.0408, 178.03072, 179.03857, 193.04172, 194.02574, 199.05114, 205.04175, 219.05744, 221.07309) could not be explained
- Mass deviation of 10 ppm with 0.002 Da absolute minimum was too restrictive for challenge 11 (demethoxycurcumin tautomers initially missed); a 0.005 Da minimum was needed
- Very large candidate sets require refscore filtering, which risks discarding the correct structure

## Evidence

- [results] Tolerance failure diagnosis: [evidence span withheld — anchor status: not_found]
- [results] Composite spectra construction: [evidence span withheld — anchor status: not_found]
- [readme] MAGMa repository and license: [evidence span withheld — anchor status: external_doc]
