---
name: smiles-based-compound-class-lookup
description: Use when you have SMILES strings for validated natural products (e.g.
  from MIBiG json entries) and need their biosynthetic-pathway-based chemical classes
  to bridge genome-based BGC classifications with structure-based metabolite classifications.
when_to_use_negative:
- When structures are already annotated with NPClassScore-compatible classes by an
  alternative pipeline (e.g. CANOPUS/MolNetEnhancer structure-based predictions on
  MS/MS spectra), as this skill adds no information and increases API calls
- When no SMILES strings are available for the metabolites in question — the lookup
  requires explicit structural identifiers, not just InChIKeys or mass values
- When working with metabolites above ~850 Da, where current class-prediction tools
  and this lookup's coverage become unreliable
edam_operation: http://edamontology.org/operation_3779
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3316
tools:
- name: NPClassifier
  role: Provides biosynthetic-pathway-aware chemical classifications for the extracted
    SMILES, retrieved through the GNPS API
- name: GNPS
  role: Hosts the API through which ClassyFire and NPClassifier chemical compound
    classifications are retrieved
- name: ClassyFire
  role: Optional hierarchical structure-based ontology (superclass/class/subclass)
    retrieved alongside NPClassifier classifications
- name: NPLinker
  role: Consumes the matched class tables as the NPClassScore empirical scoring method
    to filter BGC–MS/MS candidate links
  repo: https://github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:f01755af83cdde9fb402806757e86dbd702854ad52f35deb342cde38398e8144
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - ClassyFire
  - GNPS
  - NPClassifier
  - NPLinker
  edam_operation: http://edamontology.org/operation_3779
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# SMILES-based compound class lookup

## Summary

Retrieves NPClassifier (and optionally ClassyFire) pathway/class annotations for a set of SMILES structures via the GNPS API, enabling BGC-to-metabolite class matching in the NPClassScore workflow.

## When to use

Use when you have SMILES strings for validated natural products (e.g. from MIBiG json entries) and need their biosynthetic-pathway-based chemical classes to bridge genome-based BGC classifications with structure-based metabolite classifications.

## When NOT to use

- When structures are already annotated with NPClassScore-compatible classes by an alternative pipeline (e.g. CANOPUS/MolNetEnhancer structure-based predictions on MS/MS spectra), as this skill adds no information and increases API calls
- When no SMILES strings are available for the metabolites in question — the lookup requires explicit structural identifiers, not just InChIKeys or mass values
- When working with metabolites above ~850 Da, where current class-prediction tools and this lookup's coverage become unreliable

## Inputs

- MIBiG 2.0 json entries (downloaded from mibig.secondarymetabolites.org) containing SMILES strings and MIBiG (sub)classes
- Extracted SMILES strings for each validated BGC product

## Outputs

- NPClassifier pathway/superclass/class annotation per SMILES (retrieved via GNPS API)
- Optional ClassyFire superclass/class/subclass annotation per SMILES
- Class-by-class co-occurrence table (MIBiG genome-based class vs NPClassifier pathway) with match fractions (e.g. 76% polyketide→Polyketides, 65% terpene→Terpenoids)
- Empirical scoring tables consumable by the NPClassScore method in NPLinker

## How to apply

Download MIBiG 2.0 entries from mibig.secondarymetabolites.org in json format and extract the SMILES strings together with the MIBiG (sub)classes from each entry's json fields. Submit the extracted SMILES to the GNPS API to retrieve NPClassifier classifications (pathway, superclass, class) — and optionally ClassyFire superclass/class/subclass — so that each compound receives a biosynthetic-pathway-aware class label. The resulting class-by-class co-occurrence table (MIBiG genome-based class vs NPClassifier pathway) is then used to compute how well each pair of class terms matches: for example, 76% of polyketide BGCs match the 'Polyketides' pathway, most RiPPs and NRPs match 'Amino acids and Peptides', and 65% of terpene BGCs match 'Terpenoids'. These matched class tables feed the NPClassScore empirical scoring implemented in NPLinker, which filters out BGC–MS/MS spectrum links whose class terms disagree.

## Related tools

- **NPClassifier** (Provides biosynthetic-pathway-aware chemical classifications for the extracted SMILES, retrieved through the GNPS API)
- **GNPS** (Hosts the API through which ClassyFire and NPClassifier chemical compound classifications are retrieved)
- **ClassyFire** (Optional hierarchical structure-based ontology (superclass/class/subclass) retrieved alongside NPClassifier classifications)
- **NPLinker** (Consumes the matched class tables as the NPClassScore empirical scoring method to filter BGC–MS/MS candidate links) — https://github.com/NPLinker/nplinker

## Examples

```
download mibig.secondarymetabolites.org json entries; extract SMILES; submit to GNPS API for NPClassifier pathway annotations
```

## Evaluation signals

- Verify that the computed co-occurrence table reproduces the reported match fractions: 76% of polyketide BGCs map to NPClassifier 'Polyketides', most RiPPs/NRPs map to 'Amino acids and Peptides', and 65% of terpene BGCs map to 'Terpenoids'
- Check that every MIBiG entry with a non-null SMILES yields at least one NPClassifier pathway annotation (no silent failures in the GNPS API batch)
- Confirm that the class-by-class table dimensions match the number of distinct MIBiG (sub)classes × distinct NPClassifier pathways in the MIBiG 2.0 corpus
- Spot-check a handful of known MIBiG BGC–compound pairs to confirm that the assigned NPClassifier pathway is consistent with the expected biosynthetic class
- Ensure that scoring tables derived from this lookup, when plugged into NPClassScore with a cut-off of 0.25 after standardised Metcalf scoring (cut-off 2.5), reduce candidate links by roughly the reported average 63%

## Limitations

- The lookup depends on MIBiG entries having a parsable SMILES field; entries lacking SMILES cannot be classified
- GNPS API retrieval of NPClassifier/ClassyFire classifications requires network access and is subject to service availability and rate limits
- The resulting matched-class tables are empirical and biased toward the distribution of validated BGCs in MIBiG 2.0; rare classes may have unreliable match fractions
- Structure-based prediction tools such as CANOPUS are limited by a mass cut-off (~850 Da) and 1988 MS/MS spectra across the three PoDP datasets lacked CANOPUS or MolNetEnhancer predictions, so this lookup alone cannot annotate all metabolites in a molecular network

## Evidence

- [discussion] MIBiG 2.0 entries downloaded and SMILES extracted: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] Classifications retrieved through GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [abstract] Matched class tables used for NPClassScore filtering: "we match natural product (NP) ontologies previously independently developed for genomics and metabolomics"
- [abstract] NPClassScore reduces links by average 63%: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [discussion] Code and scoring tables available in repository: "Code and scoring tables are available at github.com/louwenjjr/mibig_classifications"
