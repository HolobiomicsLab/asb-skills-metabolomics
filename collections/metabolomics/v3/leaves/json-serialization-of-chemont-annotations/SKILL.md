---
name: json-serialization-of-chemont-annotations
description: Use when a chemical classification pipeline must expose ChemOnt annotations
  (category IDs, textual descriptions, per-compound results) in a machine-readable
  JSON payload for downstream programmatic access.
when_to_use_negative:
- When the input is already a JSON classification payload produced by the ClassyFire
  API — re-serializing adds no value
- When the goal is rule-based structural classification itself (feature extraction,
  category assignment, direct-parent selection) rather than packaging of the resulting
  annotations
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0089
tools:
- name: ClassyFire
  role: Produces the ChemOnt classifications and exposes the JSON output through its
    RESTful web application at classyfire.wishartlab.com
  repo: https://classyfire.wishartlab.com
- name: Ruby
  role: Ruby API wrapper that programmatically fetches the JSON classification and
    annotation payloads from the ClassyFire server and database
  repo: https://bitbucket.org/wishartlab/classyfire_api
- name: OBO-Edit
  role: Generates the ChemOnt OBO file whose category nodes and their descriptions
    feed the JSON serialization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:096aad8f0725ff7c080563733945f69f1d5283356bd7e5c744b1d7bdd793b31d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  - Ruby
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# json-serialization-of-chemont-annotations

## Summary

Serialize ChemOnt category identifiers, their textual descriptions, and per-compound classification results into downloadable JSON files served through the ClassyFire RESTful API and web server.

## When to use

Use when a chemical classification pipeline must expose ChemOnt annotations (category IDs, textual descriptions, per-compound results) in a machine-readable JSON payload for downstream programmatic access.

## When NOT to use

- When the input is already a JSON classification payload produced by the ClassyFire API — re-serializing adds no value
- When the goal is rule-based structural classification itself (feature extraction, category assignment, direct-parent selection) rather than packaging of the resulting annotations

## Inputs

- ChemOnt OBO taxonomy file (4825 categories, up to 11 levels) parsed from the ClassyFire/ChemOnt ontology
- Chemical Classification Dictionary entries (SMARTS patterns, Markush structures, physico-chemical constraints, formula rules) per ChemOnt category
- ClassyFire classification results including direct-parent category assignments
- Physico-chemical property values computed via ChemAxon's JChem API (version 15.5.25.0)

## Outputs

- JSON file mapping ChemOnt category identifiers (e.g. CHEMONTID:…) to their English textual descriptions
- JSON per-compound classification payloads (direct parent, ancestor categories, textual descriptions, structural features) served via the ClassyFire RESTful API

## How to apply

After ClassyFire assigns compounds to ChemOnt categories (up to 4825 categories across up to 11 levels, defined by unambiguous computable structural rules such as SMARTS patterns and Markush structures), compile the mapping from each ChemOnt category identifier to its English textual description drawn from the Chemical Classification Dictionary. Compose the per-category description by naming the shared structural pattern (e.g. ring counts, heteroatom composition, functional groups) and attach it to its CHEMONTID. Serialize the identifier-to-description mapping and the per-compound direct-parent results into a JSON file, then expose the result through the ClassyFire RESTful web application or the Ruby API (https://bitbucket.org/wishartlab/classyfire_api) so external programs can fetch classifications programmatically. Physico-chemical property values that accompany classifications are computed with ChemAxon's JChem API (version 15.5.25.0) and can be included in the same JSON payload. Validate that every ChemOnt category covered carries a non-empty, well-formed description and that the emitted file parses as valid JSON.

## Related tools

- **ClassyFire** (Produces the ChemOnt classifications and exposes the JSON output through its RESTful web application at classyfire.wishartlab.com) — https://classyfire.wishartlab.com
- **Ruby** (Ruby API wrapper that programmatically fetches the JSON classification and annotation payloads from the ClassyFire server and database) — https://bitbucket.org/wishartlab/classyfire_api
- **OBO-Edit** (Generates the ChemOnt OBO file whose category nodes and their descriptions feed the JSON serialization)

## Examples

```
curl https://classyfire.wishartlab.com/entities/<INCHIKEY>.json
```

## Evaluation signals

- Each of the 4825 ChemOnt categories carries a non-empty, well-formed English description consistent with the structural features in the Chemical Classification Dictionary
- The emitted file parses as valid JSON without errors
- Every serialized annotation references a valid CHEMONTID from the ChemOnt taxonomy (up to 11 hierarchy levels)
- Per-compound JSON payloads contain the direct parent category plus its ancestor chain as returned by ClassyFire
- Requests to the Ruby API or RESTful server return the same identifier-to-description mapping as the locally serialized JSON file

## Limitations

- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match the constraints described in a category description, so the JSON description may be inaccurate for transformed structures
- At the time of writing ClassyFire source code was not yet freely available; access relies on the web server and the Ruby API rather than local serialization of the full engine
- No specific ClassyFire software version is stated, complicating reproducibility of serialized annotation snapshots

## Evidence

- [results] ClassyFire has a comprehensive Chemical Classification Dictionary consisting of thousands of 20–50 word textual descriptions for different compound classes.: "a comprehensive Chemical Classification Dictionary consisting of thousands of 20–50 word textual descriptions for different compound classes"
- [methods] The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website.: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [abstract] a Ruby API version is available at https://bitbucket.org/wishartlab/classyfire_api, which provides programmatic access to the ClassyFire server and database: "a Ruby API version is available at https://bitbucket.org/wishartlab/classyfire_api, which provides programmatic access to the ClassyFire server and database"
- [methods] The calculation of physico-chemical properties is performed using ChemAxon’s JChem API (version 15.5.25.0).: "The calculation of physico-chemical properties is performed using ChemAxon’s JChem API (version 15.5.25.0)."
- [methods] The direct parent is the category defined by the largest structural feature that describes the compound.: "The direct parent is the category defined by the largest structural feature that describes the compound."
- [methods] The entire ClassyFire program has been converted to a web-based resource. It is a RESTful web application located at http://classyfire.wishartlab.com.: "The entire ClassyFire program has been converted to a web-based resource. It is a RESTful web application located at http://classyfire.wishartlab.com."
