---
name: metcalf-co-occurrence-score-thresholding
description: Use when a paired genomics–metabolomics dataset (e.g. PoDP accessions
  MSV000084950, MSV000084771, or a Crüsemann et al. Streptomyces/Salinispora set)
  has been loaded into NPLinker and candidate BGC-GCF–to–spectrum links must be ranked
  and filtered by co-occurrence prior to class-based refinement.
when_to_use_negative:
- The goal is chemical-class-based filtering rather than co-occurrence ranking — NPClassScore
  (0.25 cut-off) complements rather than replaces the Metcalf step and must be applied
  to surviving links afterwards
- The dataset's validated links show low standardised Metcalf scores at the default
  2.5 cut-off, as happened for the Nocardia dataset, and no threshold adjustment is
  acceptable for the analysis
- No paired genome–metabolome data are available; Metcalf scoring requires strain-level
  co-occurrence across both omics layers
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Platform in which standardised Metcalf scores between GCFs and MS/MS spectra
    are computed and cut-offs applied (get_links with 'metcalf')
  repo: github.com/NPLinker/nplinker
- name: BiG-SCAPE
  role: Groups antiSMASH BGCs into gene cluster families (GCFs) that serve as the
    co-occurrence scoring units; run with cut-off 0.3
- name: GNPS
  role: Provides the molecular networking output whose MS/MS spectra form the metabolite
    side of the Metcalf links
- name: antiSMASH
  role: Identifies the BGCs (antiSMASH 6 merged with antiSMASH 3 data for 50 strains)
    that underlie the GCFs being scored
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:d3c1f69d026a423688f9f56ba01fc5b739816aea7dd40bcea31bb6e679d72e35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metcalf-co-occurrence-score-thresholding

## Summary

Compute standardised Metcalf co-occurrence scores between BGC-GCFs and MS/MS spectra in NPLinker and apply cut-off thresholding (2.5 by default, lowered to 2.0 for the Nocardia dataset) to retain a candidate BGC–MS/MS link set before downstream NPClassScore filtering.

## When to use

Use when a paired genomics–metabolomics dataset (e.g. PoDP accessions MSV000084950, MSV000084771, or a Crüsemann et al. Streptomyces/Salinispora set) has been loaded into NPLinker and candidate BGC-GCF–to–spectrum links must be ranked and filtered by co-occurrence prior to class-based refinement.

## When NOT to use

- The goal is chemical-class-based filtering rather than co-occurrence ranking — NPClassScore (0.25 cut-off) complements rather than replaces the Metcalf step and must be applied to surviving links afterwards
- The dataset's validated links show low standardised Metcalf scores at the default 2.5 cut-off, as happened for the Nocardia dataset, and no threshold adjustment is acceptable for the analysis
- No paired genome–metabolome data are available; Metcalf scoring requires strain-level co-occurrence across both omics layers

## Inputs

- Paired omics project loaded in NPLinker via nplinker.toml (load_config + npl.load_data())
- BiG-SCAPE GCF output (BiG-SCAPE cut-off 0.3) from antiSMASH 6 (merged with antiSMASH 3) BGC predictions
- GNPS molecular networking MS/MS data (downloaded and extracted with GNPSDownloader/GNPSExtractor)
- Strain mappings from Paired Omics Data Platform accessions (genomes downloaded from NCBI when RefSeq or GenBank identifiers available)

## Outputs

- Link graph of candidate BGC-GCF to MS/MS spectrum links with standardised Metcalf scores
- List of links surviving the Metcalf cut-off (2.5, or 2.0 for the Nocardia dataset)
- Per-validated-link pass/fail record at the co-occurrence threshold (expect 5/26 failures, 75% accuracy)

## How to apply

Load the paired data with NPLinker (load_config on a nplinker.toml, npl.load_data()) and compute standardised Metcalf scores between GCFs (BiG-SCAPE output, cut-off 0.3) and GNPS MS/MS spectra via npl.get_links. Apply the standardised Metcalf cut-off of 2.5 to retain candidate links; note that for the Nocardia dataset (MSV000084771) validated links exhibited low scores, so the threshold is lowered to 2.0 to include them. Record for each experimentally validated link from the PoDP whether it passes the co-occurrence threshold — in the reference run 5 of 26 validated links failed, giving 75% full-workflow accuracy. The surviving candidate links then feed into NPClassScore filtering at a 0.25 cut-off, and ranked positions of validated links are recorded before and after that filtering.

## Related tools

- **NPLinker** (Platform in which standardised Metcalf scores between GCFs and MS/MS spectra are computed and cut-offs applied (get_links with 'metcalf')) — github.com/NPLinker/nplinker
- **BiG-SCAPE** (Groups antiSMASH BGCs into gene cluster families (GCFs) that serve as the co-occurrence scoring units; run with cut-off 0.3)
- **GNPS** (Provides the molecular networking output whose MS/MS spectra form the metabolite side of the Metcalf links)
- **antiSMASH** (Identifies the BGCs (antiSMASH 6 merged with antiSMASH 3 data for 50 strains) that underlie the GCFs being scored)

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Reproduce the reported 5/26 validated links failing the co-occurrence threshold (75% full-workflow accuracy) across the three PoDP datasets
- All 24 detectable validated links (2 of 26 lacked spectra, e.g. retimycin and nocardimicin) are carried into the NPClassScore filtering stage
- Per-link ranked positions recorded before NPClassScore filtering match the reference run, with staurosporine and rosamicin improving to rank 1 only after that subsequent filtering
- BiG-SCAPE GCF grouping used cut-off 0.3 and the Nocardia dataset threshold was 2.0 rather than the default 2.5

## Limitations

- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring the threshold be lowered from 2.5 to 2 — performance degrades on the smallest dataset
- 5 of 26 experimentally validated links fail the co-occurrence step, so Metcalf thresholding alone is insufficient and must be complemented by NPClassScore class matching

## Evidence

- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] Nocardia threshold lowering: "we changed the standardised Metcalf threshold from 2.5 to 2 for this dataset to include the validated links"
- [discussion] BiG-SCAPE cut-off: "first run through the docker version with a BiG-SCAPE cut-off of 0.3"
- [methods] get_links usage: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [discussion] PoDP datasets: "The Cyanobacteria and Nocardia datasets were accessed through their PoDP accessions, MSV000084950 and MSV000084771"
