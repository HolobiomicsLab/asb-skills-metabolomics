---
name: adduct-mass-shift-calculation
description: Use when a list of unannotated experimental m/z peaks (e.g. 200 peaks
  of 150-300 Da from an E. coli untargeted metabolomics dataset) must be matched against
  predicted compounds from a Pickaxe network expansion by enumerating adducted m/z
  values.
when_to_use_negative:
- The metabolomics peaks are already annotated to known metabolite identities, so
  mass-based adduct matching adds nothing
- The goal is pruning by chemical similarity to target compounds, thermodynamics,
  or reaction feasibility rather than by m/z matching
- No adduct list or mass tolerance can be specified, so candidate m/z values cannot
  be computed defensibly
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pickaxe
  role: Provides the filter framework in which adduct m/z computation and peak matching
    run on-the-fly during network generation
  repo: https://github.com/tyo-nu/MINE-Database
- name: minedatabase
  role: Python package containing Pickaxe, installed to implement and apply the metabolomics
    (adduct m/z) filter
  repo: https://pypi.org/project/minedatabase/2.0.0/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-023-05149-8
    title: 'Pickaxe: a Python library for the prediction of novel metabolic reactions'
  source_content_hash: sha256:459bb73c52c726dd3aacdc43320f161419ecf05fff164a63f678f73debb3262d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tyo-nu/MINE-Database
  tools:
  - Pickaxe
  - minedatabase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# adduct-mass-shift-calculation

## Summary

Computes the possible ionized m/z values for each predicted compound by applying user-supplied adducts to its neutral mass, so that network-expansion compounds can be matched against unannotated metabolomics peaks within a mass tolerance. This is the mass-matching core of Pickaxe's metabolomics filter, which prunes predicted reaction networks to compounds that could explain experimental peaks.

## When to use

Use when a list of unannotated experimental m/z peaks (e.g. 200 peaks of 150-300 Da from an E. coli untargeted metabolomics dataset) must be matched against predicted compounds from a Pickaxe network expansion by enumerating adducted m/z values.

## When NOT to use

- The metabolomics peaks are already annotated to known metabolite identities, so mass-based adduct matching adds nothing
- The goal is pruning by chemical similarity to target compounds, thermodynamics, or reaction feasibility rather than by m/z matching
- No adduct list or mass tolerance can be specified, so candidate m/z values cannot be computed defensibly

## Inputs

- List of unannotated experimental m/z peak values (e.g. 200 peaks, 150-300 Da, from an E. coli untargeted metabolomics dataset)
- Set of predicted compounds from a Pickaxe network expansion (with neutral masses)
- User-supplied adduct list to apply to each compound's neutral mass
- User-defined mass tolerance for peak matching

## Outputs

- Set of retained compounds whose adducted m/z matches at least one experimental peak
- CSV table with one row per retained compound recording the experimental m/z peak and the adduct that produced the match

## How to apply

Load the unannotated peak list (in the E. coli case study, 200 peaks with m/z between 150 and 300 Da) and the predicted compounds from the Pickaxe network expansion. For each predicted compound, compute candidate m/z values by applying each adduct in the user-supplied adduct list to the compound's neutral mass. Match computed m/z values against the peak list using the user-defined mass tolerance, retaining only compounds with at least one hit, per the Pickaxe metabolomics filter that 'only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset'. Implement this as a Python module within the minedatabase/Pickaxe filter framework so it can run on-the-fly during network generation, and write retained compounds to a CSV recording the matched experimental peak and the adduct that produced the match.

## Related tools

- **Pickaxe** (Provides the filter framework in which adduct m/z computation and peak matching run on-the-fly during network generation) — https://github.com/tyo-nu/MINE-Database
- **minedatabase** (Python package containing Pickaxe, installed to implement and apply the metabolomics (adduct m/z) filter) — https://pypi.org/project/minedatabase/2.0.0/

## Examples

```
pip install minedatabase
```

## Evaluation signals

- Every compound retained in the output table must have at least one computed adduct m/z within the user-defined tolerance of an input experimental peak
- Each output CSV row must record both the matched experimental peak and the specific adduct that produced the match
- No compound whose adducted m/z values all fall outside tolerance should appear in the retained set (the filter must have pruned it)
- Recomputing adducted m/z from a retained compound's neutral mass and recorded adduct must reproduce the matched peak within tolerance
- The filter must be callable as a Python module within the Pickaxe filter framework so it applies during network generation

## Limitations

- Compounds are only retained if their computed adducted m/z matches a peak; correct structures with adducts absent from the user-supplied adduct list will be pruned
- Matches depend on the user-defined mass tolerance; too wide a tolerance inflates false-positive assignments, too narrow loses true ones
- Runtime performance benchmarks were referenced as a supplementary file whose content is not included, so scaling behavior is not documented here
- The metabolomics filter cannot distinguish which of several matching compounds explains a peak; it only annotates candidate compounds per peak/adduct pair

## Evidence

- [results] Metabolomics filter: keep only compounds whose m/z matches peaks in the unannotated dataset: "the metabolomics filter, which only keeps compounds whose m/z value matches one of those in the unannotated metabolomics dataset"
- [results] 200 unannotated peaks with m/z 150-300 Da used as metabolomics filter inputs: "200 unannotated peaks with m/z values between 150 and 300 Da were selected from an experimental dataset as inputs to the metabolomics filter"
- [abstract] Pickaxe includes metabolomics among its on-the-fly filters: "The filters include chemical similarity to target molecules, metabolomics, thermodynamics, and reaction feasibility filters."
- [readme] minedatabase package provides Pickaxe for running and filtering network expansions: [evidence span withheld — anchor status: external_doc]
