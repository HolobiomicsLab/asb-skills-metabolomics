---
name: analog-pair-eligibility-filtering
description: Use when building an evaluation benchmark of analog pairs for modification-site
  localization, or when you need a defensible subset of structurally related compound
  pairs from GNPS libraries before running ModiFinder, CFM-ID, or random baselines.
when_to_use_negative:
- When the goal is general analog identification rather than benchmarking site-localization
  accuracy — the four checks are deliberately strict (single modification, same adduct)
  and would discard valid but multi-site analogs
- When candidate pairs already come from a curated benchmark with matching adducts
  and verified single-site differences — re-filtering wastes time and risks removing
  legitimate pairs
- When precursor mass or adduct metadata is missing or ambiguous — silently imputing
  values would violate the eligibility checks' intent
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: consumes the filtered pairs as benchmark input for likelihood scoring; the
    paper's evaluation framework is built on the pair set produced by this filtering
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MS/MS library search
  role: source of candidate analog pairs; the eligibility filter operates on library
    search results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:51860dd267f62b8849930f8107368173e46ab07522b5580cd4022279192f7109
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - CFM-ID
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# analog-pair-eligibility-filtering

## Summary

A benchmark-construction procedure that screens candidate known/unknown analog pairs from GNPS library MS/MS data against four eligibility checks (precursor mass <2000 Da, mass difference <50%, shared M+H adduct, single-site structural difference) before any site-localization scoring is attempted.

## When to use

Use when building an evaluation benchmark of analog pairs for modification-site localization, or when you need a defensible subset of structurally related compound pairs from GNPS libraries before running ModiFinder, CFM-ID, or random baselines.

## When NOT to use

- When the goal is general analog identification rather than benchmarking site-localization accuracy — the four checks are deliberately strict (single modification, same adduct) and would discard valid but multi-site analogs
- When candidate pairs already come from a curated benchmark with matching adducts and verified single-site differences — re-filtering wastes time and risks removing legitimate pairs
- When precursor mass or adduct metadata is missing or ambiguous — silently imputing values would violate the eligibility checks' intent

## Inputs

- GNPS library MS/MS spectra (mzML or mgf) for candidate analog pairs from the four cited libraries
- Precursor mass and adduct metadata for each candidate compound
- 2D chemical structures (e.g., InChI or SMILES) for each candidate compound to check for single-site structural difference

## Outputs

- A set of eligibility-filtered analog pairs, each with the known structure, unknown structure, modification site, and shared adduct
- A discarded set of pairs failing one or more eligibility checks (for auditing)

## How to apply

Enumerate candidate analog pairs by comparing each unknown compound against compounds in the reference libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) and apply the four eligibility checks verbatim: (i) precursor mass less than 2000 Da, (ii) precursor mass difference less than 50% of the smaller compound, (iii) both molecules share the same M+H adduct, and (iv) the structures differ by exactly one modification site, meaning the known structure can be derived from the unknown by removing one substructure. Only pairs passing all four checks enter the benchmark; pairs failing any single check (e.g., missing adduct metadata, multiple modifications, or precursor mass out of range) are dropped. This filtering is a prerequisite for the downstream Average-distance scoring and the baseline comparisons (Random Choice, Random Distribution, CFM-ID permutation) reported in the paper.

## Related tools

- **ModiFinder** (consumes the filtered pairs as benchmark input for likelihood scoring; the paper's evaluation framework is built on the pair set produced by this filtering) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MS/MS library search** (source of candidate analog pairs; the eligibility filter operates on library search results)

## Evaluation signals

- Every retained pair has a precursor mass strictly below 2000 Da; any pair at or above 2000 Da in the output is a filter failure
- For every retained pair, |mass_unknown − mass_known| / min(mass_unknown, mass_known) < 0.5; check this ratio on the raw masses before any unit conversion
- Both members of each retained pair report the same M+H adduct; cross-check the adduct field in the metadata rather than assuming from charge state
- Known and unknown structures differ by removal of exactly one connected substructure — verify by diffing the molecular graphs; more than one modification site invalidates the pair
- The size of the filtered pair set should be substantially smaller than the raw candidate set; if the reduction ratio is near 1, the checks were likely not applied

## Limitations

- The checks are only as good as the precursor mass, adduct, and structure metadata in the source GNPS libraries; missing or stale metadata silently distorts the benchmark
- 'Exactly one modification site' is a graph-level assertion that can be hard to verify automatically for complex natural products; ambiguous cases may need manual curation
- The filter is designed for [M+H]+ pairs only; evaluating other adducts or negative-mode data would require redefining checks (iii) and (ii)
- Pairs with large structural differences but small mass differences (e.g., isomeric multi-modifications) can pass the mass-ratio check while violating the single-site rule; mass-based checks alone are insufficient

## Evidence

- [methods] Benchmark pair eligibility checks: "(i) their precursor mass is checked to be less than 2000 Da, (ii) the difference in precursor masses is less than 50%"
- [methods] Source libraries: "[1-GNPS-MSMLS, 2-GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, 3-GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE35, and 4-BERKELEY-LAB]"
- [readme] Filter rationale: [evidence span withheld — anchor status: external_doc]
