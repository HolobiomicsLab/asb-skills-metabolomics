---
name: database-annotation-scale-comparison
description: Use when you need to compute or restate the fraction of compounds in
  a large reference collection (e.g. PubChem's >91M compounds as of June 2016) that
  have received a chemical class annotation from a given system (e.g. MeSH thesaurus
  vs ClassyFire), and compare two annotation systems on scale.
when_to_use_negative:
- The analysis target is not a large chemical database coverage metric, but instead
  a single-compound classification task — use a direct ClassyFire classification workflow
  instead.
- The comparison needs a third annotation system with a different unit of scale (e.g.
  an ontology node count rather than a compound annotation count), since the source
  figures are compound-coverage percentages over a single reference collection (PubChem).
- The database annotation counts you have are already per-class breakdowns rather
  than a single overall coverage percentage, so the 0.12% / 0.85 ratio framing does
  not apply.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: Source of the automated annotation count baseline (>77 million compounds)
    used to compute the coverage comparison against the MeSH thesaurus
  repo: http://classyfire.wishartlab.com
- name: manual expert review
  role: Reference method whose coverage baseline (MeSH thesaurus, 0.12% of PubChem)
    is the comparator in the scale-comparison table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:bdf7fea616cb033b8bce31b8ef08713fa00954e2f743fa11f091b0a063fc9bd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# database-annotation-scale-comparison

## Summary

Reproduces the coverage baseline that motivated ClassyFire's large-scale automated chemical classification: the manually maintained MeSH thesaurus covered only 0.12% of PubChem's >91,000,000 compounds (≈109,200 compounds), while ClassyFire annotated >77 million, so a comparison table quantifies the ~0.85 (≈84.6%) deployment coverage ratio and exposes the annotation scale gap.

## When to use

Use when you need to compute or restate the fraction of compounds in a large reference collection (e.g. PubChem's >91M compounds as of June 2016) that have received a chemical class annotation from a given system (e.g. MeSH thesaurus vs ClassyFire), and compare two annotation systems on scale.

## When NOT to use

- The analysis target is not a large chemical database coverage metric, but instead a single-compound classification task — use a direct ClassyFire classification workflow instead.
- The comparison needs a third annotation system with a different unit of scale (e.g. an ontology node count rather than a compound annotation count), since the source figures are compound-coverage percentages over a single reference collection (PubChem).
- The database annotation counts you have are already per-class breakdowns rather than a single overall coverage percentage, so the 0.12% / 0.85 ratio framing does not apply.

## Inputs

- PubChem compound count baseline (>91,000,000 as of June 2016)
- MeSH thesaurus classification coverage fraction (0.12% of PubChem)
- ClassyFire annotation count baseline (>77 million compounds)
- MeSH thesaurus classification counts (manual expert review)

## Outputs

- coverage_comparison_table.csv with one row per classification system (MeSH thesaurus manual coverage vs ClassyFire automated coverage) and columns for compound counts and percent coverage of PubChem
- Computed MeSH-classified PubChem compound count (≈109,200 compounds)
- Computed ClassyFire deployment coverage ratio (≈0.85 of the PubChem collection)

## How to apply

1. Record the reported baseline figures as constants in a Python script: the PubChem compound count of >91,000,000 (as of June 2016), the reported MeSH thesaurus classification coverage of 0.12%, and the ClassyFire annotation count of >77 million compounds. 2. Compute the absolute number of MeSH-classified PubChem compounds as 0.0012 × 91,000,000 ≈ 109,200 compounds. 3. Compute the ClassyFire deployment coverage ratio as 77,000,000 / 91,000,000 ≈ 0.846 (≈84.6% of the PubChem collection). 4. Assemble a comparison table with one row per classification system (MeSH thesaurus manual coverage vs ClassyFire automated coverage) and columns for compound counts and percent coverage of PubChem. 5. Write the table to coverage_comparison_table.csv using pandas and report the two computed values. The rationale is that ClassyFire's large-scale annotation (>77M compounds) was designed to address the narrow manually curated MeSH baseline (0.12% of PubChem), so the comparison exposes the annotation scale gap.

## Related tools

- **ClassyFire** (Source of the automated annotation count baseline (>77 million compounds) used to compute the coverage comparison against the MeSH thesaurus) — http://classyfire.wishartlab.com
- **manual expert review** (Reference method whose coverage baseline (MeSH thesaurus, 0.12% of PubChem) is the comparator in the scale-comparison table)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame({'classification_system': ['MeSH thesaurus (manual)', 'ClassyFire (automated)'], 'compound_count': [0.0012*91000000, 77000000], 'percent_coverage_of_pubchem': [0.12, 77000000/91000000*100]}); df.to_csv('coverage_comparison_table.csv', index=False)"
```

## Evaluation signals

- Verify the computed MeSH-classified PubChem compound count equals ≈109,200 (0.0012 × 91,000,000).
- Verify the MeSH coverage fraction equals 0.12% exactly as reported in the article.
- Verify the ClassyFire coverage ratio is ≈0.85 (i.e. >77M of >91M PubChem compounds), matching the baseline figures reported in the article.
- Verify coverage_comparison_table.csv has one row per classification system (MeSH thesaurus manual vs ClassyFire automated) and columns for compound counts and percent coverage of PubChem.
- Console output reports both computed values (the absolute MeSH-classified count and the ClassyFire coverage ratio) without unit or scale errors.

## Limitations

- The reported coverage figures are anchored to a June 2016 snapshot of PubChem (>91,000,000 compounds); the coverage ratio changes as PubChem grows.
- ClassyFire's annotation count is reported as '>77 million', so the computed 0.85 ratio is a lower bound on deployment coverage rather than an exact value.
- The MeSH thesaurus coverage figure (0.12%) is a manually maintained, partially mapped baseline; the article notes ClassyFire has been manually mapped to MeSH 'although only partially', so the two systems are not perfectly commensurable.
- ClassyFire's source code was not yet freely available at the time of writing (promised by December 2016), limiting independent verification of the annotation counts.
- No quantitative performance figures (runtime, accuracy) are cited for ClassyFire's classification in this article section.

## Evidence

- [abstract] ClassyFire has been used to annotate over 77 million compounds: "ClassyFire has been used to annotate over 77 million compounds and has already been integrated into other software packages"
- [intro] manual MeSH baseline covers only 0.12% of PubChem's >91 million compounds: [evidence span withheld — anchor status: not_found]
- [methods] ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
- [methods] The compounds are all included in the PubChem database: "The compounds are all included in the PubChem database."
- [discussion] commitment to making the resource fully open source by December 2016: "we are actively working on a version of ClassyFire that has freely accessible source code and documentation. We are committed to making this resource fully open source (by December 2016)"
