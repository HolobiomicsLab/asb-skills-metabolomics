---
name: link-rank-comparison-table-reproduction
description: Use when benchmarking a BGC–MS/MS linking method (e.g. NPClassScore vs
  Metcalf-only) against a set of experimentally validated links from the PoDP, and
  you must report per-link retention and rank changes at a fixed score cut-off.
when_to_use_negative:
- When no experimentally validated links are available for the paired dataset — the
  article notes 'the lack of verified genome-metabolome links remains one of the current
  bottlenecks to validate new integrative omics mining methods'
- When the metabolites of interest exceed the ~850 Da CANOPUS mass cut-off or lack
  any CANOPUS/MolNetEnhancer class prediction, since such links are discarded by the
  filter_missing_scores toggle
- When the goal is de novo link discovery rather than benchmarking retention/rank
  against known validated links
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Platform in which standardised Metcalf scoring and NPClassScore filtering
    are run to obtain ranked candidate BGC-GCF to MS/MS spectrum links
  repo: github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical chemical-class matching score (cut-off 0.25) used to re-rank and
    filter candidate links before/after comparison
  repo: github.com/louwenjjr/mibig_classifications
- name: GNPS
  role: Source of molecular networking MS/MS data downloaded and extracted into NPLinker
    via GNPSDownloader/GNPSExtractor
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (docker run, cut-off 0.3) used as the
    GCF side of candidate links
- name: CANOPUS
  role: Predicts structure-based NP classes for spectra (via SIRIUS v4.9.3 on the
    GNPS mgf) required for NPClassScore scoring
  repo: github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:c6853a778fdf94901ae99000acce4847bf0d6f5551b0c21872d0296edf77e709
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - CANOPUS
  - GNPS
  - NPClassScore
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# link-rank-comparison-table-reproduction

## Summary

Reproduces the ranked-position comparison table (Table 1) showing where experimentally validated BGC–MS/MS links sit before and after NPClassScore filtering in NPLinker, quantifying retention (23/24 detectable links, 96%) and rank improvements such as staurosporine and rosamicin moving to rank 1.

## When to use

Use when benchmarking a BGC–MS/MS linking method (e.g. NPClassScore vs Metcalf-only) against a set of experimentally validated links from the PoDP, and you must report per-link retention and rank changes at a fixed score cut-off.

## When NOT to use

- When no experimentally validated links are available for the paired dataset — the article notes 'the lack of verified genome-metabolome links remains one of the current bottlenecks to validate new integrative omics mining methods'
- When the metabolites of interest exceed the ~850 Da CANOPUS mass cut-off or lack any CANOPUS/MolNetEnhancer class prediction, since such links are discarded by the filter_missing_scores toggle
- When the goal is de novo link discovery rather than benchmarking retention/rank against known validated links

## Inputs

- 26 experimentally validated BGC–MS/MS links from the PoDP (Cyanobacteria MSV000084950, Nocardia MSV000084771, Streptomyces/Salinispora dataset from Crüsemann et al.)
- GNPS molecular networking MS/MS spectra (mgf) for the three datasets
- antiSMASH (v3 merged with v6) BGC annotations and BiG-SCAPE gene cluster families (cut-off 0.3)
- CANOPUS (via SIRIUS v4.9.3) and MolNetEnhancer structure-based class predictions
- NPLinker config (nplinker.toml) with strain mappings assembled from the three PoDP accessions

## Outputs

- Per-link table of ranked positions before and after NPClassScore filtering (reproducing Table 1)
- Retention fraction after NPClassScore filtering at 0.25 cut-off (expect 23/24 detectable links = 96%)
- Full-workflow accuracy vs the Metcalf co-occurrence threshold (expect 21/26 = 75%, 5 links below threshold)
- List of validated links excluded due to missing spectra (retimycin, nocardimicin)

## How to apply

Retrieve the 26 validated genome-metabolome links recorded on the Paired Omics Data Platform for the three datasets (Cyanobacteria MSV000084950, Nocardia MSV000084771, and the Streptomyces/Salinispora dataset from Crüsemann et al.), and load them into NPLinker with GNPS molecular networking, antiSMASH/BiG-SCAPE (cut-off 0.3) GCFs, and CANOPUS/MolNetEnhancer structure classes. Compute standardised Metcalf scores with a 2.5 cut-off (lowered to 2.0 for the Nocardia dataset, whose validated links had low Metcalf scores), and record which validated links pass co-occurrence filtering. Apply NPClassScore with its default 0.25 cut-off to the surviving candidate links, discarding links without structure-based classes (1988 spectra lacked CANOPUS/MolNetEnhancer predictions), and record each validated link's ranked position before and after filtering. Note that 2 of the 26 validated compounds (retimycin and nocardimicin) have no detectable MS/MS spectrum, so retention is judged on the 24 detectable links. Acceptance requires 23/24 (96%) retention after NPClassScore, 21/26 (75%) passing the full Metcalf-then-NPClassScore workflow, and per-link rank changes matching Table 1, including staurosporine and rosamicin reaching rank 1.

## Related tools

- **NPLinker** (Platform in which standardised Metcalf scoring and NPClassScore filtering are run to obtain ranked candidate BGC-GCF to MS/MS spectrum links) — github.com/NPLinker/nplinker
- **NPClassScore** (Empirical chemical-class matching score (cut-off 0.25) used to re-rank and filter candidate links before/after comparison) — github.com/louwenjjr/mibig_classifications
- **GNPS** (Source of molecular networking MS/MS data downloaded and extracted into NPLinker via GNPSDownloader/GNPSExtractor)
- **BiG-SCAPE** (Groups BGCs into gene cluster families (docker run, cut-off 0.3) used as the GCF side of candidate links)
- **CANOPUS** (Predicts structure-based NP classes for spectra (via SIRIUS v4.9.3 on the GNPS mgf) required for NPClassScore scoring) — github.com/kaibioinfo/canopus_treemap

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Retention after NPClassScore filtering equals 23/24 detectable validated links (96%)
- Full-workflow accuracy equals 21/26 (75%) with exactly 5 validated links falling below the Metcalf co-occurrence threshold
- Two validated compounds (retimycin, nocardimicin) are correctly flagged as having no detectable MS/MS spectrum
- Per-link rank changes reproduce Table 1, with staurosporine and rosamicin ranked 1 after NPClassScore filtering
- Nocardia validated links are retained only when the standardised Metcalf threshold is lowered from 2.5 to 2.0

## Limitations

- No MS/MS spectrum could be found for retimycin and nocardimicin, so only 24 of 26 validated links are assessable
- The Nocardia dataset required lowering the Metcalf threshold from 2.5 to 2, indicating weaker co-occurrence performance on the smallest dataset
- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions and their links were discarded; not filtering decreases average candidate links per GCF by only 49%
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are poorly handled
- antiSMASH 3 and antiSMASH 6 outputs were merged across 50 strains, introducing version inconsistency

## Evidence

- [abstract] 96% of experimentally validated links in three paired omics datasets were retained and prioritised by NPClassScore: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] Metcalf and NPClassScore cut-offs: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] Nocardia threshold adjustment: "the Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, which is why we changed the standardised Metcalf threshold"
- [discussion] Missing spectra for validated compounds: "No MS/MS spectrum could be found this way for retimycin and nocardimicin."
- [discussion] Rank recording procedure: "The ranked position for each validated link was recorded before and after filtering with NPClassScore"
- [methods] NPLinker API usage: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
