---
name: tandem-ms-candidate-ranking
description: Use when assigning ranks to candidate structures for an HR-MS/MS peak
  with no spectral-library match, especially when candidate lists come from PubChem/ChemSpider
  and reference or retention-time information is available.
when_to_use_negative:
- The mass spectrum is dominated by one or only a few fragments (e.g. a single water
  loss) explainable by most candidates, so it lacks information to distinguish candidates.
- A confirmed reference standard or direct spectral-library match already establishes
  the identity, making re-ranking unnecessary.
- The analysis has no retention-time or reference information and relies only on fragmentation
  of very large databases, where candidate ranking becomes increasingly hard (PubChem
  has doubled since 2009).
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetFrag2.2
  role: 'Core engine: retrieves candidates from PubChem/ChemSpider, performs in silico
    bond-dissociation fragmentation and combines fragment, reference/patent and retention-time
    (user-defined) scores for ranking.'
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFragCommandLine
  role: Command-line batch scoring of spectra against candidate lists; built with
    Maven from the MetFragRelaunched sources.
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: CFM-ID
  role: Competitive fragmentation modelling prediction added as an additional scoring
    function to the MetFrag score (raised PubChem refs+RT Top 1 from 336 to 343).
- name: metfRag
  role: R interface to MetFragLib, installable via install_github("ipb-halle/MetFragR/metfRag").
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:61d16b6f89fcc9b179e1a3c43c1f5d2bd777ed34674a2167d55e181736a564d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - CFM-ID
  - Maven
  - MetFrag2.2
  - MetFragCommandLine
  - metfRag
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# tandem-ms-candidate-ranking

## Summary

Rank candidate molecular structures for tandem MS (HR-MS/MS) spectra by combining in silico bond-dissociation fragment explanation scores with reference/patent counts and retention-time (log P) scoring, as implemented in MetFrag2.2 and benchmarked on the Hill 102-compound and 473-spectrum Eawag Orbitrap XL datasets.

## When to use

Use when assigning ranks to candidate structures for an HR-MS/MS peak with no spectral-library match, especially when candidate lists come from PubChem/ChemSpider and reference or retention-time information is available.

## When NOT to use

- The mass spectrum is dominated by one or only a few fragments (e.g. a single water loss) explainable by most candidates, so it lacks information to distinguish candidates.
- A confirmed reference standard or direct spectral-library match already establishes the identity, making re-ranking unnecessary.
- The analysis has no retention-time or reference information and relies only on fragmentation of very large databases, where candidate ranking becomes increasingly hard (PubChem has doubled since 2009).

## Inputs

- HR-MS/MS peak list or merged spectra (e.g. merged with mzClust_hclust in xcms, eppm=5e-6, eabs=0.001 Da) with precursor mass and fragment peaks
- Candidate structure lists from PubChem/ChemSpider web services (June 2015 snapshots) or user-supplied SDF / suspect-list InChIKeys
- Retention times per substance for the RT–log P model (MassBank records)
- Reference/data-source/patent counts retrieved via ChemSpider and PubChem web services
- MetFrag parameter files (HILL, EA, UF, EQEx, EQExPlus datasets from CHIN-D-15-00088 ZIP archives)

## Outputs

- Score-ranked candidate list per spectrum with the correct candidate's rank (Top 1/3/5 counts)
- Median rank and average candidates-per-compound statistics (e.g. metfrag2010_hill_ranking_results.csv)
- Retained weight combination (ωFrag, ωRT, ωRefs) from the 1000-draw optimization

## How to apply

Retrieve candidate structures for the precursor mass from PubChem or ChemSpider (or a user-supplied SDF / suspect list such as STOFF-IDENT InChIKeys), then fragment each candidate in silico with the MetFrag bond-dissociation fragmenter and score fragments against the measured product ions. Enable reference, data-source and patent scoring terms retrieved via web services, and add retention time as a user-defined score using an RT–log P model established per dataset by tenfold cross-validation. Optimize the weights by randomly drawing 1000 combinations with (ωFrag+ωRT+ωRefs=1), then filter stereoisomers keeping only the best-scored candidate by the first InChIKey block. Build the command-line tool from ipb-halle/MetFragRelaunched with Maven ('mvn clean install -pl MetFragCommandLine -am', optionally -DskipTests) and run batch scoring; correct ranks are reported as median rank (e.g. 8 on the Hill dataset vs. Feb 2006 PubChem, 31.5 vs. 2009 PubChem) or Top 1/3/5 counts (e.g. 420/473 Top 1 with ChemSpider refs+RT).

## Related tools

- **MetFrag2.2** (Core engine: retrieves candidates from PubChem/ChemSpider, performs in silico bond-dissociation fragmentation and combines fragment, reference/patent and retention-time (user-defined) scores for ranking.) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFragCommandLine** (Command-line batch scoring of spectra against candidate lists; built with Maven from the MetFragRelaunched sources.) — https://github.com/ipb-halle/MetFragRelaunched
- **CFM-ID** (Competitive fragmentation modelling prediction added as an additional scoring function to the MetFrag score (raised PubChem refs+RT Top 1 from 336 to 343).)
- **metfRag** (R interface to MetFragLib, installable via install_github("ipb-halle/MetFragR/metfRag").) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- On the Hill 102-compound dataset with MetFrag2010 fragment-only scoring, expect median rank 8 (avg 338 candidates/compound) vs. Feb 2006 PubChem and median rank 31.5 (avg 2508) vs. the 2009 PubChem snapshot.
- On the 473-spectrum Eawag Orbitrap XL dataset, expect 420 Top 1 ranks (89%) with ChemSpider refs+RT, 336 (71%) with PubChem refs+RT, and 343 (89%) with PubChem refs+RT+CFM-ID.
- MetFrag2.2 vs. MetFrag2010 on Hill data should improve median rank from 18.5 to 14.5 and Top 1/3/5 from 8/20/28 to 9/24/34.
- Weight-optimization draws must satisfy the constraint ωFrag+ωRT+ωRefs=1 over 1000 random combinations.
- After stereoisomer filtering, no two retained candidates share the same first InChIKey block.

## Limitations

- Mass spectra dominated by one or a few fragments do not contain enough information to distinguish candidates.
- Further validation of the MetFrag2.2+CFM-ID combination was beyond the article's scope; retraining CFM-ID on Orbitrap data could improve results.
- Parameter, spectral, candidate, result and ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets exist only as external ZIP archives (http://msbi.ipb-halle.de/download/CHIN-D-15-00088/), and database snapshots age — larger current PubChem versions make ranking harder than the benchmarked June 2015 / 2009 / Feb 2006 versions.

## Evidence

- [abstract] Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %), respectively, and even up to 343 Top 1 ranks (PubChem): "Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %)"
- [results] Weight optimization by random draws: [evidence span withheld — anchor status: not_found]
- [results] Stereoisomer filtering by InChIKey first block: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [results] RT–logP model via tenfold cross-validation: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [readme] README build instructions for the command-line tool: "mvn clean install -pl MetFragCommandLine -am"
