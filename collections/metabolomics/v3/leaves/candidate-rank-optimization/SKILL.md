---
name: candidate-rank-optimization
description: Use when ranking candidate structures for an LC-HR-MS/MS unknown where
  spectral library matches are insufficient and extra evidence (reference/patent counts,
  RT–log P, CFM-ID predictions) should be fused into the MetFrag score.
when_to_use_negative:
- Input is already a spectral-library match confirmed by a pure reference standard
  — no ranking optimization is needed.
- Mass spectrum is dominated by one or a few fragments (e.g. a water loss) explainable
  by most candidates — the article states such spectra 'simply do not contain enough
  information' to distinguish candidates by fragmentation alone.
- Working with a small, curated database (e.g. STOFF-IDENT suspect list) where database
  size does not make candidate ranking hard — the improvement mainly addresses growing
  PubChem/ChemSpider candidate lists.
edam_operation: http://edamontology.org/operation_3809
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MetFrag2.2
  role: 'Core ranking engine: computes the fragmentation score and accepts user-defined
    (RT–log P) plus reference/data-source/patent scoring terms whose weights are optimized'
  repo: http://c-ruttkies.github.io/MetFrag/
- name: CFM-ID
  role: 'Optional additional scoring function: competitive fragmentation modelling
    predictions added to the MetFrag score for the PubChem run to reach 343 Top 1
    ranks'
- name: ChemSpider
  role: Candidate source and provider of reference/data-source/patent counts via web
    services for the 420-Top-1 configuration
- name: PubChem
  role: Candidate source and provider of reference/data-source/patent counts via web
    services for the 336/343-Top-1 configurations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:a0806115a84aad6b3f598724331660c518c59f1e31adf0a7795d50a2e4776f0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - ChemSpider
  - MetFrag2.2
  - PubChem
  edam_operation: http://edamontology.org/operation_3809
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# candidate-rank-optimization

## Summary

Optimize MetFrag2.2 candidate ranking weights by adding reference, data-source, patent and retention-time (RT–log P) scoring terms to the in silico fragmentation score, so that the correct candidate reaches Top 1 on the 473-spectrum Eawag Orbitrap XL benchmark.

## When to use

Use when ranking candidate structures for an LC-HR-MS/MS unknown where spectral library matches are insufficient and extra evidence (reference/patent counts, RT–log P, CFM-ID predictions) should be fused into the MetFrag score.

## When NOT to use

- Input is already a spectral-library match confirmed by a pure reference standard — no ranking optimization is needed.
- Mass spectrum is dominated by one or a few fragments (e.g. a water loss) explainable by most candidates — the article states such spectra 'simply do not contain enough information' to distinguish candidates by fragmentation alone.
- Working with a small, curated database (e.g. STOFF-IDENT suspect list) where database size does not make candidate ranking hard — the improvement mainly addresses growing PubChem/ChemSpider candidate lists.

## Inputs

- Merged HR-MS/MS spectra (e.g. the 473 Eawag Orbitrap XL spectra merged with xcms mzClust_hclust, eppm=5e-6, eabs=0.001 Da)
- Candidate structures retrieved from ChemSpider/PubChem (June 2015) or supplied SDF
- Retention time per substance extracted from MassBank records for RT–log P model fitting
- Reference, data-source and patent counts retrieved via ChemSpider and PubChem web services
- CFM-ID predicted spectra for PubChem candidates (optional combined run)

## Outputs

- Score-ranked candidate list per spectrum (MetFrag scoring output)
- Top 1 / Top 3 / Top 5 correct-candidate counts and ranking files (as in EA, UF, EQEx, EQExPlus, HILL result tables)
- Chosen weight combination (ωFrag, ωRT, ωRefs) from the 1000-draw optimization

## How to apply

Build MetFragLib and MetFragCommandLine from ipb-halle/MetFragRelaunched with Maven, retrieve candidates (ChemSpider/PubChem, June 2015 snapshots), and run the MetFrag bond-dissociation fragmenter on each 473 merged Orbitrap XL spectrum. Enable the reference, data-source and patent scoring terms fetched via ChemSpider/PubChem web services and add the RT–log P model (fit per dataset by tenfold cross-validation) as a user-defined score. Draw 1000 random weight combinations with ωFrag+ωRT+ωRefs=1, apply stereoisomer filtering on the first InChIKey block, and optionally add CFM-ID's competitive fragmentation prediction as an additional scoring term for the PubChem runs. Judge success by Top 1 counts: expect 420/473 (89%) with ChemSpider+refs/RT, 336/473 (71%) with PubChem+refs/RT, and 343 with PubChem+refs/RT+CFM-ID.

## Related tools

- **MetFrag2.2** (Core ranking engine: computes the fragmentation score and accepts user-defined (RT–log P) plus reference/data-source/patent scoring terms whose weights are optimized) — http://c-ruttkies.github.io/MetFrag/
- **CFM-ID** (Optional additional scoring function: competitive fragmentation modelling predictions added to the MetFrag score for the PubChem run to reach 343 Top 1 ranks)
- **ChemSpider** (Candidate source and provider of reference/data-source/patent counts via web services for the 420-Top-1 configuration)
- **PubChem** (Candidate source and provider of reference/data-source/patent counts via web services for the 336/343-Top-1 configurations)

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git && mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- Top 1 counts on the 473-spectrum Eawag dataset: 420 with ChemSpider+refs/RT, 336 with PubChem+refs/RT, 343 with PubChem+refs/RT+CFM-ID
- Median rank on the Hill et al. dataset improves from 18.5 (MetFrag2010) to 14.5 (MetFrag2.2); Top 1/3/5 correct candidates go from 8/20/28 to 9/24/34
- Weight-optimization invariant: the drawn 1000 combinations must satisfy ωFrag+ωRT+ωRefs = 1
- Stereoisomer filtering leaves only the best-scored candidate per first InChIKey block in each ranking
- Optimal weights verified against three additional datasets totalling 824 merged HR-MS/MS spectra

## Limitations

- 4-isopropylbenzenesulfonic acid identification could not be confirmed for lack of a sufficiently pure reference standard.
- Further validation of the MetFrag2.2+CFM-ID combination was beyond the article's scope because CFM-ID would need retraining on Orbitrap data.
- Parameter files, spectral data, candidate, result and ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives (http://msbi.ipb-halle.de/download/CHIN-D-15-00088/), not from the repository.
- PubChem doubled in size since 2009, making candidate ranking progressively harder — weights tuned on older snapshots may not transfer.

## Evidence

- [abstract] Top1 counts vs weight-optimization: "Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %), respectively"
- [abstract] CFM-ID combination outcome: [evidence span withheld — anchor status: not_found]
- [results] Weight optimization procedure: [evidence span withheld — anchor status: not_found]
- [results] RT–log P cross-validation: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [results] Stereoisomer filtering: [evidence span withheld — anchor status: not_found]
- [readme] Build command: "mvn clean install -pl MetFragCommandLine -am"
- [abstract] Verification datasets: "The optimal parameters and weights were verified using three additional datasets of 824 merged HR-MS/MS spectra in total."
- [results] Spectra merge parameters: [evidence span withheld — anchor status: not_found]
