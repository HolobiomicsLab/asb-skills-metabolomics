---
name: bgc-msms-candidate-link-scoring
description: Use when you have paired omics data (genome-derived GCFs plus GNPS molecular
  networking MS/MS spectra) and need to reduce long candidate BGC–spectrum link lists,
  e.g. via NPLinker with Metcalf scoring followed by NPClassScore class-matching.
when_to_use_negative:
- The dataset's MS/MS spectra largely lack CANOPUS or MolNetEnhancer structure-based
  class predictions — links without classes are discarded, and without this filtering
  the average decrease in candidate links per GCF drops to 49%
- No paired genome–metabolome data exist (no GCFs from antiSMASH/BiG-SCAPE alongside
  a GNPS molecular network), e.g. when only a single validated genome-metabolome pair
  is available
- The goal is de novo structure elucidation of large metabolites — CANOPUS coverage
  is limited by an 850 Da mass cut-off and Spec2Vec/MS2DeepScore may be better suited
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: NPLinker
  role: Platform that assembles GNPS molecular networking, BiG-SCAPE GCFs and strain
    mappings, computes standardised Metcalf links via get_links, and implements NPClassScore
    filtering
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical class-matching score used to filter Metcalf candidate links below
    the 0.25 cut-off
  repo: https://github.com/louwenjjr/mibig_classifications
- name: GNPS
  role: Provides molecular networking output (mgf) and the API for ClassyFire/NPClassifier
    structure-based classifications
- name: BiG-SCAPE
  role: Groups antiSMASH BGCs into GCFs (cut-off 0.3) used as one side of the link
    graph
- name: antiSMASH
  role: Genome mining to identify BGCs (antiSMASH 6 output merged with prior antiSMASH
    3 data)
- name: CANOPUS
  role: Predicts structure-based compound classes from MS/MS spectra via Sirius v4.9.3
    on the GNPS mgf file
  repo: https://github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:cad00e30cb62709de390e742022caa49014981258647aa3bd4f8717dd202ad7e
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
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# bgc-msms-candidate-link-scoring

## Summary

Score and filter candidate BGC–MS/MS spectrum links by combining co-occurrence-based standardised Metcalf scoring in NPLinker with the class-matching NPClassScore, which matches genome-based (BiG-SCAPE/antiSMASH) and structure-based (ClassyFire/NPClassifier/CANOPUS) natural product ontologies to prune false-positive links.

## When to use

Use when you have paired omics data (genome-derived GCFs plus GNPS molecular networking MS/MS spectra) and need to reduce long candidate BGC–spectrum link lists, e.g. via NPLinker with Metcalf scoring followed by NPClassScore class-matching.

## When NOT to use

- The dataset's MS/MS spectra largely lack CANOPUS or MolNetEnhancer structure-based class predictions — links without classes are discarded, and without this filtering the average decrease in candidate links per GCF drops to 49%
- No paired genome–metabolome data exist (no GCFs from antiSMASH/BiG-SCAPE alongside a GNPS molecular network), e.g. when only a single validated genome-metabolome pair is available
- The goal is de novo structure elucidation of large metabolites — CANOPUS coverage is limited by an 850 Da mass cut-off and Spec2Vec/MS2DeepScore may be better suited

## Inputs

- Paired omics dataset from the PoDP (e.g. MassIVE accessions MSV000078836/MSV000078839/MSV000079284 for Streptomyces/Salinispora, MSV000084950 for Cyanobacteria, MSV000084771 for Nocardia)
- nplinker.toml configuration file (podp mode with dataset PoDP id)
- GNPS molecular networking output (mgf and network files)
- antiSMASH 3/6 BGC annotations and BiG-SCAPE GCF clustering (cut-off 0.3)
- MIBiG 2.0 entries (json) with SMILES and (sub)classes for class-matching tables
- Structure-based class predictions: ClassyFire/NPClassifier via GNPS API and CANOPUS (Sirius v4.9.3) on the GNPS mgf

## Outputs

- Ranked list of candidate BGC–MS/MS spectrum links per GCF after Metcalf scoring
- Filtered link list after NPClassScore cut-off 0.25
- Per-dataset candidate links per GCF before/after filtering and percentage reductions
- Ranked positions of experimentally validated links before and after NPClassScore filtering

## How to apply

Instantiate NPLinker from a nplinker.toml (podp mode with the dataset's PoDP id) and load GNPS molecular networking, antiSMASH/BiG-SCAPE GCFs (BiG-SCAPE cut-off 0.3), and strain mappings with load_data(). Compute candidate links with npl.get_links(npl.gcfs, 'metcalf') using a standardised Metcalf cut-off of 2.5 (lowered to 2 for the Nocardia dataset, whose validated links had low Metcalf scores). Then filter candidate links whose NPClassScore falls below 0.25, also discarding links lacking structure-based classes (filter_missing_scores toggle). Structure-based classes come from ClassyFire/NPClassifier via the GNPS API and CANOPUS run through Sirius v4.9.3 directly on the GNPS mgf file; class-matching tables are trained on MIBiG 2.0 entries via the mibig_classifications repo. Count candidate links per GCF before and after filtering to quantify reduction, and record ranked positions of experimentally validated links.

## Related tools

- **NPLinker** (Platform that assembles GNPS molecular networking, BiG-SCAPE GCFs and strain mappings, computes standardised Metcalf links via get_links, and implements NPClassScore filtering) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Empirical class-matching score used to filter Metcalf candidate links below the 0.25 cut-off) — https://github.com/louwenjjr/mibig_classifications
- **GNPS** (Provides molecular networking output (mgf) and the API for ClassyFire/NPClassifier structure-based classifications)
- **BiG-SCAPE** (Groups antiSMASH BGCs into GCFs (cut-off 0.3) used as one side of the link graph)
- **antiSMASH** (Genome mining to identify BGCs (antiSMASH 6 output merged with prior antiSMASH 3 data))
- **CANOPUS** (Predicts structure-based compound classes from MS/MS spectra via Sirius v4.9.3 on the GNPS mgf file) — https://github.com/kaibioinfo/canopus_treemap

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Reproduced per-dataset reductions in candidate links per GCF match the reported values: 68% (Streptomyces/Salinispora), 53% (Cyanobacteria, 27→13), 69% (Nocardia, 206→64)
- Average decrease in candidate links per GCF across the three datasets equals the reported 63%
- 96% of experimentally validated links in the three paired omics datasets are retained and prioritised after NPClassScore filtering
- Filter parameters match the protocol: standardised Metcalf cut-off 2.5 (2 for Nocardia), NPClassScore cut-off 0.25, BiG-SCAPE cut-off 0.3
- Validated-link ranked positions before and after filtering are recorded and link-count tables are written to output

## Limitations

- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction; links without structure-based classes were discarded, and not filtering them yields an average decrease of only 49% of candidate links per GCF
- No MS/MS spectrum could be found for the validated compounds retimycin and nocardimicin, so they cannot be scored
- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring the threshold to be lowered from 2.5 to 2
- CANOPUS coverage is limited by an 850 Da mass cut-off; MolNetEnhancer cannot currently be run within NPLinker and must be run externally on the GNPS platform
- antiSMASH 6 output was merged with antiSMASH 3 data for 50 strains, introducing a version inconsistency

## Evidence

- [abstract] Abstract reduction claim: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [discussion] Filter thresholds: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] Validated link retention: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] CANOPUS run on GNPS mgf: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [readme] Purpose of matched classes (README): [evidence span withheld — anchor status: external_doc]
