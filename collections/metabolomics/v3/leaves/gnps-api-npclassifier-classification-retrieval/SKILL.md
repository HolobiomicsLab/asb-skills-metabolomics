---
name: gnps-api-npclassifier-classification-retrieval
description: Use when you have SMILES strings for natural products (e.g. from MIBiG
  json entries) and need biosynthetically-informed chemical class terms (pathway/class/subclass)
  to match against BGC class ontologies.
when_to_use_negative:
- 'Input molecules are >850 Da or lack SMILES: NPClassifier requires chemical structure
  input, and the article notes CANOPUS/structure-based predictions have a 850 Da mass
  cut-off, so structure-class retrieval is inappropriate there.'
- 'The goal is MS/MS-spectrum-level class prediction rather than SMILES-based classification:
  use CANOPUS via Sirius or MolNetEnhancer on GNPS molecular networking mgf files
  instead.'
- The analysis needs ClassyFire hierarchical superclass/class/subclass ontologies
  rather than biosynthetic-pathway-informed NPClassifier terms; the two ontologies
  serve different downstream matching tables.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: NPClassifier
  role: Chemical classification service that assigns pathway/class/subclass terms
    to input SMILES; queried via the GNPS API to obtain NP classifications for MIBiG
    product structures
- name: GNPS
  role: Hosts the API endpoint through which NPClassifier (and ClassyFire) classification
    calls are made during the ontology-bridging workflow
- name: mibig_classifications
  role: Jupyter notebooks and scripts that extract SMILES and BGC classes from MIBiG
    json entries, retrieve NPClassifier classifications, and build the class-matching
    co-occurrence tables used by NPClassScore
  repo: github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:b448add6122ca00456282b55fb2f89dd28c85d6f2794ca14bb951507a88ebf47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - NPClassifier
  - NPLinker
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# gnps-api-npclassifier-classification-retrieval

## Summary

Retrieve NPClassifier pathway-level chemical classifications for natural product SMILES by querying the GNPS API, as used to bridge MIBiG BGC classes to metabolomics class ontologies in the NPClassScore pipeline.

## When to use

Use when you have SMILES strings for natural products (e.g. from MIBiG json entries) and need biosynthetically-informed chemical class terms (pathway/class/subclass) to match against BGC class ontologies.

## When NOT to use

- Input molecules are >850 Da or lack SMILES: NPClassifier requires chemical structure input, and the article notes CANOPUS/structure-based predictions have a 850 Da mass cut-off, so structure-class retrieval is inappropriate there.
- The goal is MS/MS-spectrum-level class prediction rather than SMILES-based classification: use CANOPUS via Sirius or MolNetEnhancer on GNPS molecular networking mgf files instead.
- The analysis needs ClassyFire hierarchical superclass/class/subclass ontologies rather than biosynthetic-pathway-informed NPClassifier terms; the two ontologies serve different downstream matching tables.

## Inputs

- MIBiG 2.0 json entries downloaded from mibig.secondarymetabolites.org containing SMILES strings and MIBiG (sub)classes
- SMILES strings of natural products (e.g. experimentally validated BGC products from MIBiG)
- MIBiG genome-based BGC class labels (e.g. polyketide, RiPP, NRP, terpene) extracted from the json fields

## Outputs

- NPClassifier pathway, class and subclass terms per input SMILES retrieved via the GNPS API
- Class-by-class co-occurrence table mapping MIBiG BGC classes to NPClassifier pathway terms
- Match-fraction statistics (e.g. fraction of polyketide BGCs matching Polyketides pathway)

## How to apply

Extract the SMILES strings from each MIBiG 2.0 json entry downloaded from mibig.secondarymetabolites.org, then submit them to the NPClassifier chemical classification service accessed through the GNPS API to obtain pathway, class and subclass assignments. NPClassifier was developed specifically for natural product classification, taking both chemistry and biosynthetic pathways into account, making it the appropriate ontology to pair with genome-based BGC classes rather than purely structure-based ontologies. For each input SMILES, record the returned pathway term (e.g. Polyketides, Amino acids and Peptides, Terpenoids) and tabulate counts against the corresponding MIBiG (sub)classes extracted from the same json fields. The resulting class-by-class co-occurrence table quantifies how well genome-based BGC classes match structure-based classes and forms the empirical basis for the NPClassScore scoring tables. Cross-validate by confirming expected match fractions, such as 76% of polyketide BGCs matching the Polyketides pathway and 65% of terpene BGCs matching Terpenoids.

## Related tools

- **NPClassifier** (Chemical classification service that assigns pathway/class/subclass terms to input SMILES; queried via the GNPS API to obtain NP classifications for MIBiG product structures)
- **GNPS** (Hosts the API endpoint through which NPClassifier (and ClassyFire) classification calls are made during the ontology-bridging workflow)
- **mibig_classifications** (Jupyter notebooks and scripts that extract SMILES and BGC classes from MIBiG json entries, retrieve NPClassifier classifications, and build the class-matching co-occurrence tables used by NPClassScore) — github.com/louwenjjr/mibig_classifications

## Examples

```
import requests; smi = 'CC(=O)OC1=C(O)C=CC=C1O'; r = requests.post('https://npclassifier.gnps2.org/classify', data={'smiles': smi}); print(r.json()['pathway'])
```

## Evaluation signals

- Reproduce the published co-occurrence pattern: 76% of polyketide BGCs match the NPClassifier 'Polyketides' pathway, most RiPPs and NRPs match 'Amino acids and Peptides', and 65% of terpene BGCs match 'Terpenoids'
- Every SMILES extracted from a MIBiG json entry should yield a pathway classification (or a documented failure) — no unhandled missing values in the resulting table
- The resulting class-match co-occurrence table must be usable as a scoring matrix for NPClassScore, i.e. row labels are MIBiG (sub)classes and column labels are NPClassifier pathway/class terms
- Cross-check a subset of classifications against the ClassyFire retrievals performed via the same GNPS API to confirm both ontologies were obtained for the same SMILES set

## Limitations

- Classifications depend on the availability and correctness of SMILES strings in the MIBiG json entries; entries without valid SMILES cannot be classified
- NPClassifier terms reflect biosynthetic-pathway-informed classes; molecules whose biosynthesis is ambiguous may receive inconsistent pathway assignments
- Retrieval through the GNPS API is subject to the API's availability and rate limits; large MIBiG-scale batch queries may require retries or chunking
- The scoring tables derived from these match fractions are empirical and version-specific to MIBiG 2.0; newer MIBiG releases may require re-derivation

## Evidence

- [results] MIBiG-based class matching shows 76% of polyketide BGCs match the NPClassifier 'Polyketides' pathway, most RiPPs and NRPs match 'Amino acids and Peptides', and 65% of terpene BGCs match 'Terpenoids'.: [evidence span withheld — anchor status: not_found]
- [intro] NPClassifier was developed specifically for NP classifications, taking both chemistry and biosynthetic pathways into account: "Recently, NPClassifier was developed specifically for NP classifications, taking both chemistry and biosynthetic pathways into account [17]"
- [discussion] Extract SMILES and MIBiG (sub)classes from MIBiG 2.0 json entries, then retrieve classifications via the GNPS API: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] README: series of jupyter notebooks and scripts to retrieve chemical classes (ClassyFire/NPClassifier), as well as BGC classifications for MIBiG BGCs and their products: [evidence span withheld — anchor status: external_doc]
- [readme] README: counting interactions between the two classification types, matching certain chemical and BGC classes; relative counts assess how well each class-term pair matches: [evidence span withheld — anchor status: external_doc]
