---
name: reference-patent-count-scoring
description: Use when ranking tandem-MS candidate lists whose in silico fragmentation
  scores alone (HR-MS/MS only) leave the correct compound outside the Top 1, and reference/patent/retention
  metadata can be retrieved per candidate.
when_to_use_negative:
- Spectra dominated by one or a few fragments explainable by most candidates — such
  spectra 'simply do not contain enough information' to distinguish candidates regardless
  of scoring terms
- When no retention time or reference metadata is available for the candidate set,
  since ωRT and ωRefs terms cannot be established
- Candidates from databases without web-service reference/patent counts (e.g. user-supplied
  SDFs lacking data-source annotations)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MetFrag2.2
  role: Command-line candidate ranking engine whose reference, data-source and patent
    scoring terms are enabled; built from ipb-halle/MetFragRelaunched with Maven
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: CFM-ID
  role: Competitive fragmentation modelling prediction added as an additional scoring
    function to the MetFrag score (raises PubChem Top 1 from 336 to 343)
- name: ChemSpider
  role: Source of reference/data-source/patent counts via web services; 420 Top 1
    ranks (89%) with refs+RT
- name: PubChem
  role: Alternative candidate database and reference-information source; 336 Top 1
    ranks (71%), 343 with CFM-ID
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:b6f4663c5ed872fc56538c4c75103baa1deeec47e85af8600bab5dfdec187d5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - CFM-ID
  - ChemSpider
  - MetFrag2.2
  - PubChem
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# reference-patent-count-scoring

## Summary

Augments MetFrag2.2 candidate ranking with reference, data-source and patent counts retrieved via ChemSpider/PubChem web services, plus a retention-time score, lifting Top 1 ranks from 30 to 420 (ChemSpider) and 336 (PubChem) on the 473-spectrum Eawag Orbitrap XL benchmark.

## When to use

Use when ranking tandem-MS candidate lists whose in silico fragmentation scores alone (HR-MS/MS only) leave the correct compound outside the Top 1, and reference/patent/retention metadata can be retrieved per candidate.

## When NOT to use

- Spectra dominated by one or a few fragments explainable by most candidates — such spectra 'simply do not contain enough information' to distinguish candidates regardless of scoring terms
- When no retention time or reference metadata is available for the candidate set, since ωRT and ωRefs terms cannot be established
- Candidates from databases without web-service reference/patent counts (e.g. user-supplied SDFs lacking data-source annotations)

## Inputs

- HR-MS/MS tandem mass spectra (e.g. 473-spectrum Eawag Orbitrap XL dataset)
- Candidate structure lists retrieved from ChemSpider or PubChem (June 2015 snapshots) or user-supplied SDF
- Retention times of substances per dataset (e.g. extracted from MassBank records)
- Reference/data-source/patent counts per candidate from ChemSpider and PubChem web services

## Outputs

- Score-ranked candidate list per spectrum with combined ωFrag+ωRT+ωRefs weighted score
- Top 1 / Top 3 / Top 5 correct-candidate rank counts (expected: 420 ChemSpider, 336 PubChem, 343 PubChem+CFM-ID)
- Results table of ranking counts per configuration

## How to apply

After MetFrag's in silico bond dissociation fragmentation and fragment-to-peak scoring, enable the reference, data-source and patent information scoring terms retrieved from ChemSpider and PubChem web services (databases downloaded June 2015). Add retention time as a user-defined score built from an RT–log P model established per dataset via tenfold cross-validation, exploiting the correlation between octanol–water partitioning coefficient and reverse-phase LC retention. Optimize the combination of scoring terms by randomly drawing 1000 weight combinations such that (ωFrag+ωRT+ωRefs=1). Filter stereoisomer redundancy by keeping only the best-scored candidate per first InChIKey block, then count spectra for which the correct candidate is ranked Top 1. Optionally add CFM-ID's competitive fragmentation modelling prediction as an additional scoring function on top of the MetFrag score for further gains.

## Related tools

- **MetFrag2.2** (Command-line candidate ranking engine whose reference, data-source and patent scoring terms are enabled; built from ipb-halle/MetFragRelaunched with Maven) — https://github.com/ipb-halle/MetFragRelaunched
- **CFM-ID** (Competitive fragmentation modelling prediction added as an additional scoring function to the MetFrag score (raises PubChem Top 1 from 336 to 343))
- **ChemSpider** (Source of reference/data-source/patent counts via web services; 420 Top 1 ranks (89%) with refs+RT)
- **PubChem** (Alternative candidate database and reference-information source; 336 Top 1 ranks (71%), 343 with CFM-ID)

## Examples

```
mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- Top 1 correct-candidate counts reproduce article targets: 420/473 (89%) with ChemSpider+refs+RT, 336/473 (71%) with PubChem+refs+RT, 343 with PubChem+refs+RT+CFM-ID
- All 1000 random weight draws satisfy the constraint ωFrag+ωRT+ωRefs=1
- Stereoisomer filtering leaves exactly one candidate per first-InChIKey-block group, keeping the best-scored one
- Baseline sanity check: HR-MS/MS-only scoring with PubChem yields ~30 Top 1 ranks (MetFrag2.2) vs 43 (CFM-ID), far below the ref/RT-enhanced counts
- RT–log P model validated by tenfold cross-validation per dataset before weighting

## Limitations

- Weights were optimized by random draw (1000 combinations), so results depend on the drawn combinations; optimal parameters were only verified on three additional datasets of 824 merged spectra in total
- Further validation of the MetFrag2.2+CFM-ID combination was beyond the article's scope; CFM-ID could improve further if retrained on Orbitrap data
- Databases grow: PubChem had doubled since 2009, making candidate ranking harder over time — counts may not reproduce on current database snapshots (article used June 2015 versions)
- Retention-time score assumes LC/MS on reverse-phase columns where RT correlates with log P

## Evidence

- [abstract] Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %), respectively, and even up to 343 Top 1 ranks (PubChem): "Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %)"
- [results] the combination of each scoring term was assessed by randomly drawing 1000 different weight combinations such that (ωFrag+ωRT+ωRefs=1): [evidence span withheld — anchor status: not_found]
- [results] The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently"
- [results] Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [abstract] retrieval of reference, data source and patent information via ChemSpider and PubChem web services: "retrieval of reference, data source and patent information via ChemSpider and PubChem web services"
- [readme] mvn clean install -pl MetFragCommandLine -am: "mvn clean install -pl MetFragCommandLine -am"
