---
name: expert-substructure-ambiguity-refinement
description: Use when ModiFinder MF-R output over a known/unknown analog pair (e.g.
  Kirromycin-Goldinodox, Naphthomycin A-B) leaves many atoms with high modification-site
  likelihood, and expert knowledge of gas-phase fragmentation can prune the peak-to-substructure
  assignments.
when_to_use_negative:
- When the initial ModiFinder likelihood distribution already shows a single dominant
  high-scoring modification site and no ambiguous substructure assignments remain.
- When no domain knowledge about likely gas-phase fragmentation mechanisms (e.g. amide-bond
  cleavage) is available to justify restricting peak-to-substructure assignments,
  since arbitrary pruning is not expert refinement.
- When spectra have not been preprocessed per ModiFinder defaults (peaks <1% of base
  peak removed, normalized to Euclidean norm 1), as raw peak lists will give misleading
  initial assignments.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ModiFinder
  role: Runs MF-R on analog spectrum pairs, exposes Compound/Spectrum loaders and
    peak-to-fragment maps for expert editing, and regenerates atom likelihoods and
    Evaluation Scores after refinement
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Provides the combinatorial fragmentation (breadth-first, depth 2) that assigns
    candidate substructures to MS/MS peaks; its assignments are the objects being
    pruned
- name: MS/MS library search
  role: Source of the Kirromycin/Goldinodox and Naphthomycin A/B spectra and 2D structures
    from TUEBINGEN/GNPS accessions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:d1f78da10408c2b1a369b1e8d93673564a627b9e4f92db4181a9cb38c5df9c44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# expert-substructure-ambiguity-refinement

## Summary

Refines ModiFinder's MF-R likelihood distributions by manually restricting which substructures may explain specific MS/MS peaks, combining computational modification-site localization with domain-knowledge expertise to reduce ambiguity in natural product analog pairs.

## When to use

Use when ModiFinder MF-R output over a known/unknown analog pair (e.g. Kirromycin-Goldinodox, Naphthomycin A-B) leaves many atoms with high modification-site likelihood, and expert knowledge of gas-phase fragmentation can prune the peak-to-substructure assignments.

## When NOT to use

- When the initial ModiFinder likelihood distribution already shows a single dominant high-scoring modification site and no ambiguous substructure assignments remain.
- When no domain knowledge about likely gas-phase fragmentation mechanisms (e.g. amide-bond cleavage) is available to justify restricting peak-to-substructure assignments, since arbitrary pruning is not expert refinement.
- When spectra have not been preprocessed per ModiFinder defaults (peaks <1% of base peak removed, normalized to Euclidean norm 1), as raw peak lists will give misleading initial assignments.

## Inputs

- Paired MS/MS spectra (MS2-known and MS2-unknown) with 2D structures for structurally related analogs, retrieved via ModiFinder Compound/Spectrum loaders from MS/MS libraries (TUEBINGEN/GNPS accessions)
- ModiFinder MF-R initial likelihood distribution over atoms of the known molecule
- Peak-to-fragment assignment maps from MAGMa combinatorial fragmentation (depth 2, 40 ppm tolerance)
- True modification site (for Evaluation Score computation, e.g. N-methylation of Kirromycin; methylation of Naphthomycin B)

## Outputs

- Refined peak-to-substructure assignment maps with restricted candidate substructures per shifted peak
- Regenerated post-refinement atom likelihood distributions
- Pre- and post-refinement Average-distance Evaluation Scores (e.g. 0.86->0.93, 0.56->0.72)
- Ambiguity-reduction counts (candidate substructures before/after, e.g. 7->1 and 27->2/40->2) and high-scoring modification-site counts (e.g. 24->2), exported to refinement_scores.csv

## How to apply

Run ModiFinder MF-R (MAGMa combinatorial fragmentation at depth 2, 40 ppm m/z tolerance) on preprocessed spectra (peaks <1% of base peak removed, normalized to Euclidean norm 1) and compute the baseline Average-distance Evaluation Score with the BasicEvaluationEngine against the true modification site. Inspect the peak-to-fragment maps for shifted peaks whose initial assignment spans many candidate substructures, then apply expert knowledge of likely gas-phase fragmentation to restrict each such peak to a small number of substructures: in the Kirromycin-Goldinodox case, the 112.04 and 178.05 m/z peaks were cut from 7 candidate substructures to 1 each (retaining only amide-bond fragmentation products); for Naphthomycin A/B, the 133.07 and 147.08 m/z peaks were cut from 27 and 40 substructures to 2 each based on fragmentation at the amide bond. Regenerate atom probabilities and recompute the Evaluation Score; for the natural-product case studies this raised the score from 0.86 to 0.93 (Kirromycin) and from 0.56 to 0.72 (Naphthomycin B), and reduced the count of high-scoring modification sites for Naphthomycin B from 24 to 2.

## Related tools

- **ModiFinder** (Runs MF-R on analog spectrum pairs, exposes Compound/Spectrum loaders and peak-to-fragment maps for expert editing, and regenerates atom likelihoods and Evaluation Scores after refinement) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Provides the combinatorial fragmentation (breadth-first, depth 2) that assigns candidate substructures to MS/MS peaks; its assignments are the objects being pruned)
- **MS/MS library search** (Source of the Kirromycin/Goldinodox and Naphthomycin A/B spectra and 2D structures from TUEBINGEN/GNPS accessions)

## Examples

```
modifinder MF-R --known kirromycin --unknown goldinodox --mz-tolerance 40ppm --depth 2 --edit-peaks 112.04:1sub,178.05:1sub --output refinement_scores.csv
```

## Evaluation signals

- Post-refinement Average-distance Evaluation Score exceeds the baseline computed with the BasicEvaluationEngine (expected 0.93 vs 0.86 for Kirromycin; 0.72 vs 0.56 for Naphthomycin B)
- Number of candidate substructures assigned to each edited peak is reduced as intended (Kirromycin: 7->1 for the 112.04 and 178.05 m/z peaks; Naphthomycin B: 27->2 and 40->2 for the 133.07 and 147.08 m/z peaks)
- Count of high-scoring modification sites for Naphthomycin B drops from 24 to 2 after refinement
- Baseline scores reproduce 0.86 (Kirromycin) and 0.56 (Naphthomycin B) before any expert editing, confirming spectra, structures, and MF-R parameters (depth 2, 40 ppm) match the case-study setup
- Pre- and post-refinement scores, ambiguity counts, and site counts for both compounds are written to refinement_scores.csv

## Limitations

- Refinement depends on expert domain knowledge of fragmentation mechanisms; without it, pruning peak-to-fragment maps risks discarding the correct substructure.
- The demonstrated scores (0.86->0.93, 0.56->0.72) come from a small set of empirical natural-product case studies (Kirromycin, Naphthomycin B) and may not generalize to all analog pairs.
- Helpers are unspecified in the MF-R configuration for these case studies, so reproduction may vary if helper-based substructure filtering is applied differently.

## Evidence

- [methods] Kirromycin restrict the 112.04 m/z and 178.05 m/z peaks to one substructure each (from 7, retaining only amide-bond fragmentation products): [evidence span withheld — anchor status: not_found]
- [intro] showcase empirical examples of how ModiFinder's computational approach can be combined with domain knowledge expertise to facilitate the discovery of new natural products: [evidence span withheld — anchor status: not_found]
- [readme] This documentation is available at: https://wang-bioinformatics-lab.github.io/ModiFinder_base/: [evidence span withheld — anchor status: external_doc]
