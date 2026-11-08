---
name: mibig-json-data-parsing
description: Use when you need structured MIBiG biosynthetic gene cluster annotations
  in machine-readable form — e.g. to count annotated compounds/A-domains, build substrate-SMILES
  maps, or harvest training data for ML predictors.
when_to_use_negative:
- The analysis targets predicted (not experimentally validated) BGCs — MIBiG contains
  standardised entries only for experimentally validated BGCs of known function
- You need raw nucleotide sequence for all clusters — the `genbanks` directory only
  contains a handful of sequence files not available from NCBI GenBank/RefSeq
- You need NRPS A-domain substrate *predictions* rather than the experimentally validated
  substrate specificity labels MIBiG curates
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: PIKAChU
  role: Validate syntactic correctness of extracted SMILES strings (used in the paper
    for A-domain substrates and structure counts)
- name: SANDPUMA
  role: 'Benchmark reference: its training-set size is compared against MIBiG 3.0''s
    2775 labelled A-domain data points'
- name: NRPSPredictor2
  role: 'Benchmark reference: its training-set size is compared against MIBiG 3.0''s
    labelled A-domain count to confirm the 3-4x advantage'
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:95e7b7d2fdea4677dbadc58a5e13269d33d4792fb530e5b8995cf1e29824ab6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NRPSPredictor2
  - PIKAChU
  - SANDPUMA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mibig-json-data-parsing

## Summary

Parse the JSON-formatted MIBiG 3.0 annotation data from the mibig-secmet/mibig-json GitHub repository to extract compound structures, bioactivities, and NRPS A-domain substrate annotations, then validate extracted tallies against the published MIBiG 3.0 figures (e.g. 2775 A-domains, 274 unique substrates, 1188 SMILES, 1002 bioactivity-annotated compounds).

## When to use

Use when you need structured MIBiG biosynthetic gene cluster annotations in machine-readable form — e.g. to count annotated compounds/A-domains, build substrate-SMILES maps, or harvest training data for ML predictors.

## When NOT to use

- The analysis targets predicted (not experimentally validated) BGCs — MIBiG contains standardised entries only for experimentally validated BGCs of known function
- You need raw nucleotide sequence for all clusters — the `genbanks` directory only contains a handful of sequence files not available from NCBI GenBank/RefSeq
- You need NRPS A-domain substrate *predictions* rather than the experimentally validated substrate specificity labels MIBiG curates

## Inputs

- MIBiG 3.0 JSON-formatted cluster entries from the `data` directory of the mibig-secmet/mibig-json GitHub repository
- MIBiG 2.0 counts (e.g. 755 A-domains, 1347 BGCs with structural data) as baselines for version comparison
- Published training-set sizes of SANDPUMA and NRPSPredictor2 for fold-ratio benchmarks

## Outputs

- CSV tables such as a_domain_substrates.csv (substrate-to-SMILES mapping with per-substrate A-domain counts), bioactivity_coverage_table.csv, and structure_annotation_coverage.csv
- Validated tallies of annotated BGC entries, compounds with SMILES, unique bioactivities, and unique A-domain substrates
- Pandas DataFrames suitable for downstream machine-learning training on sequence-structure-function relationships

## How to apply

Clone the mibig-secmet/mibig-json repository where the current MIBiG annotations live in the `data` directory (entry status tracked via the `cluster.status` field), with a handful of sequence files in `genbanks` not available from NCBI GenBank/RefSeq. Recursively parse each cluster entry JSON with Python's json library, extracting the fields relevant to your question: compound SMILES strings (validate syntax with PIKAChU), bioactivity terms and chemical target annotations, or NRPS A-domain substrate annotations with stereochemically curated isomeric SMILES per substrate. Normalise and deduplicate the extracted terms (e.g. reduce activity terms to 51 unique bioactivities mapped onto 68 functional categories, or 274 unique A-domain substrates identified by isomeric SMILES). Tally counts and cross-check against reported figures: 2775 A-domain annotations (up from 755 in MIBiG 2.0), 1188 compounds with SMILES, 1860 BGCs with structural data, 1002 bioactivity-annotated compounds, 95 compounds with molecular targets, and 661 new entries added while 69 duplicate/low-quality entries were removed. Write resulting tables (e.g. a_domain_substrates.csv, bioactivity_coverage_table.csv, structure_annotation_coverage.csv) with pandas.

## Related tools

- **PIKAChU** (Validate syntactic correctness of extracted SMILES strings (used in the paper for A-domain substrates and structure counts))
- **SANDPUMA** (Benchmark reference: its training-set size is compared against MIBiG 3.0's 2775 labelled A-domain data points)
- **NRPSPredictor2** (Benchmark reference: its training-set size is compared against MIBiG 3.0's labelled A-domain count to confirm the 3-4x advantage)

## Examples

```
git clone https://github.com/mibig-secmet/mibig-json && python -c "import json,glob; entries=[json.load(open(f)) for f in glob.glob('mibig-json/data/*.json')]"
```

## Evaluation signals

- Total annotated NRPS A-domains must equal 2775 (vs 755 in MIBiG 2.0)
- Unique A-domain substrates, deduplicated by stereochemically curated isomeric SMILES, must equal 274
- Compounds with SMILES structural identifiers must equal 1188, and BGCs with structural data must reconcile 1347 (v2.0) to 1860 (v3.0)
- Bioactivity annotations must tally to 1002 compounds, 51 unique bioactivities, 68 functional categories, and 95 compounds with chemical target data
- Every counted SMILES string must parse successfully with PIKAChU before inclusion in the tally

## Limitations

- 69 duplicate and low-quality entries were removed in MIBiG 3.0, so tallies differ between versions and stale mirrors may give inconsistent counts
- Entry status is tracked via the `cluster.status` field; users must account for entry status when counting
- The repository `genbanks` directory contains only a handful of sequence files not available from GenBank/RefSeq, so sequence retrieval may require external sources

## Evidence

- [readme] Repos README: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] Article intro: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures, the annotation of 1002 bioactivities of BGC products"
