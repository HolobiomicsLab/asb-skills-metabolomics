---
name: npclassscore-class-matching-filtering
description: Use when NPLinker co-occurrence (standardised Metcalf) scoring yields
  long candidate BGC-GCF to MS/MS spectrum link lists and you need to filter or prioritise
  links using chemical-class agreement, e.g. on paired PoDP genomics–metabolomics
  datasets.
when_to_use_negative:
- Large metabolites above the ~850 Da mass cut-off where CANOPUS coverage fails —
  Spec2Vec or MS2DeepScore embeddings may be better suited
- Datasets where validated links show very low standardised Metcalf scores (as with
  Nocardia) unless the Metcalf threshold is explicitly lowered, otherwise links are
  removed before class filtering
- Molecules whose MS/MS spectra cannot be retrieved (e.g. retimycin and nocardimicin
  had no matching spectrum), which cannot be assessed at all
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: NPLinker
  role: Platform implementing standardised Metcalf scoring and the NPClassScore filter;
    run via nplinker.toml config and load_data/get_links API
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical class matching score applied at 0.25 cut-off to remove candidate
    links with mismatched BGC/compound classes
  repo: https://github.com/louwenjjr/mibig_classifications
- name: CANOPUS
  role: SIRIUS-based structure-class prediction run directly on the GNPS mgf file
    to supply metabolome-side NP classes
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: BiG-SCAPE
  role: Groups antiSMASH BGCs into gene cluster families (cut-off 0.3) used as the
    genome-side units for linking
- name: GNPS
  role: Provides molecular networking output and API access to ClassyFire/NPClassifier
    compound classifications
- name: antiSMASH
  role: Genome mining for BGC identification (antiSMASH 6 output merged with antiSMASH
    3 data); validated BGC locations checked with cblaster
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:af1e5f5fa4bb4fe1880815919e1f823dc6def6cdaf4a99444fac67ce3006d587
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - CANOPUS
  - GNPS
  - NPClassScore
  - NPLinker
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# npclassscore-class-matching-filtering

## Summary

NPClassScore is an empirical natural-product class matching score, implemented in NPLinker, that filters candidate BGC–MS/MS spectrum links by matching genome-based BGC classes (via antiSMASH/BiG-SCAPE and MIBiG-derived scoring tables) to metabolite classes (CANOPUS/MolNetEnhancer, ClassyFire/NPClassifier), reducing false-positive links by ~63% over co-occurrence-only strategies.

## When to use

Use when NPLinker co-occurrence (standardised Metcalf) scoring yields long candidate BGC-GCF to MS/MS spectrum link lists and you need to filter or prioritise links using chemical-class agreement, e.g. on paired PoDP genomics–metabolomics datasets.

## When NOT to use

- Large metabolites above the ~850 Da mass cut-off where CANOPUS coverage fails — Spec2Vec or MS2DeepScore embeddings may be better suited
- Datasets where validated links show very low standardised Metcalf scores (as with Nocardia) unless the Metcalf threshold is explicitly lowered, otherwise links are removed before class filtering
- Molecules whose MS/MS spectra cannot be retrieved (e.g. retimycin and nocardimicin had no matching spectrum), which cannot be assessed at all

## Inputs

- Paired genomics–metabolomics datasets from the Paired Omics Data Platform (e.g. MSV000084950 Cyanobacteria, MSV000084771 Nocardia, Streptomyces/Salinispora dataset from Crüsemann et al.)
- GNPS molecular networking output (mgf files) and spectra for candidate MS/MS links
- antiSMASH (v3/v6) gbk files and BiG-SCAPE GCF clustering (cut-off 0.3)
- MIBiG 2.0 entries (json) with SMILES and (sub)classes for scoring tables
- CANOPUS/SIRIUS and MolNetEnhancer chemical class predictions

## Outputs

- Filtered list of BGC-GCF to MS/MS spectrum links passing NPClassScore ≥ 0.25
- Ranked positions of validated links before and after NPClassScore filtering (Table 1)
- Retention/accuracy statistics (e.g. 23/24 detectable validated links retained, 96%; 5/26 below Metcalf threshold, 75% full-workflow accuracy)

## How to apply

First run standardised Metcalf scoring in NPLinker with a cut-off of 2.5 (lowered to 2 for the Nocardia dataset MSV000084771, where validated links had low Metcalf scores), after which candidate links are filtered out if their NPClassScore is below the 0.25 cut-off. Class inputs come from antiSMASH 6 (merged with prior antiSMASH 3 data) and BiG-SCAPE (cut-off 0.3) on the genome side, and CANOPUS (run via SIRIUS v4.9.3 on the GNPS mgf file) plus MolNetEnhancer (run externally on the GNPS platform) on the metabolome side; BGC and compound class ontologies are bridged through MIBiG 2.0 scoring tables. Discard candidate links lacking structure-based classes (1988 spectra had no CANOPUS or MolNetEnhancer prediction). Record the ranked position of each validated link before and after filtering to confirm prioritisation (e.g. staurosporine and rosamicin moving to rank 1).

## Related tools

- **NPLinker** (Platform implementing standardised Metcalf scoring and the NPClassScore filter; run via nplinker.toml config and load_data/get_links API) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Empirical class matching score applied at 0.25 cut-off to remove candidate links with mismatched BGC/compound classes) — https://github.com/louwenjjr/mibig_classifications
- **CANOPUS** (SIRIUS-based structure-class prediction run directly on the GNPS mgf file to supply metabolome-side NP classes) — https://github.com/kaibioinfo/canopus_treemap
- **BiG-SCAPE** (Groups antiSMASH BGCs into gene cluster families (cut-off 0.3) used as the genome-side units for linking)
- **GNPS** (Provides molecular networking output and API access to ClassyFire/NPClassifier compound classifications)
- **antiSMASH** (Genome mining for BGC identification (antiSMASH 6 output merged with antiSMASH 3 data); validated BGC locations checked with cblaster)

## Examples

```
pip install git+https://github.com/nplinker/nplinker@dev; config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Retention check: 23 of 24 detectable validated links (96%) pass the default NPClassScore 0.25 threshold (2 of 26 validated links had missing spectra)
- Full-workflow accuracy: 5 of 26 validated links fail the co-occurrence (Metcalf 2.5) threshold, giving 75% accuracy for the full NPLinker workflow
- Per-GCF candidate link reduction averages 63% compared to co-occurrence-based strategy alone; not discarding links without structure-based classes reduces this reduction to 49%
- Rank comparison: validated link ranks improve after NPClassScore filtering, reproducing Table 1 values including staurosporine and rosamicin at rank 1
- cblaster with default settings should confirm the BGC locations of validated links against the antiSMASH gbk files of each dataset

## Limitations

- Lack of verified genome–metabolome links remains a bottleneck for validating integrative omics mining methods
- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions, so their candidate links were discarded; keeping them halves the filtering benefit (average 49% instead of 63% link reduction)
- CANOPUS coverage is limited by an 850 Da mass cut-off; larger metabolites are not well handled by current tools
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally on the GNPS platform
- Merged antiSMASH 3 and antiSMASH 6 outputs introduce a version inconsistency across the 50 strains

## Evidence

- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] 96% of experimentally validated links in three paired omics datasets were retained and prioritised by NPClassScore: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [abstract] the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [discussion] Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [readme] These matched classes can then be used to rule out unreliable BGC-compound matches, as is done in the NPClassScore method: [evidence span withheld — anchor status: external_doc]
