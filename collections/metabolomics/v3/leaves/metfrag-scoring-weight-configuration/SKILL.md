---
name: metfrag-scoring-weight-configuration
description: Use when ranking candidate structures from HR-MS/MS spectra with MetFrag2.2
  and you need to set or verify the ωFrag, ωRT and ωRefs weights (summing to 1) against
  validation datasets.
when_to_use_negative:
- When only HR-MS/MS information is available and no retention time or reference data
  exist — in that case MetFrag2.2 scored much worse (30 Top 1 ranks with PubChem vs.
  336 with added RT and reference information)
- When spectra are dominated by one or only a few fragments (e.g. a water loss) that
  most candidates can explain — they lack information to distinguish candidates regardless
  of weights
- When a dedicated spectral library search against standards (e.g. MassBank) would
  directly confirm the compound instead of in silico ranking
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetFrag2.2 / MetFragCommandLine
  role: Executes in silico bond-dissociation fragmentation, candidate scoring with
    configurable weights (ωFrag+ωRT+ωRefs = 1) and InChIKey-based stereoisomer filtering
  repo: http://c-ruttkies.github.io/MetFrag/
- name: xcms
  role: Merges replicate spectra per dataset with mzClust_hclust (eppm = 5e-6, eabs
    = 0.001 Da) before ranking
- name: OpenBabel
  role: Converts SMILES to InChIKeys (e.g. for suspect lists such as STOFF-IDENT)
    used as MetFrag input
- name: metfRag
  role: R interface to MetFrag for running the scoring configuration from R
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:a4bc0aa5979011d8ed449b23200c905060598f6313d2a218f04be265733ca173
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFrag2.2
  - MetFrag2.2 / MetFragCommandLine
  - MetFragCommandLine
  - MetFragLib
  - OpenBabel
  - metfRag
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metfrag-scoring-weight-configuration

## Summary

Configures and verifies MetFrag2.2 scoring parameters and weights (fragmentation score, RT–log P user-defined score, reference counts) so that candidate rankings of small molecules from HR-MS/MS spectra are reproducible across datasets.

## When to use

Use when ranking candidate structures from HR-MS/MS spectra with MetFrag2.2 and you need to set or verify the ωFrag, ωRT and ωRefs weights (summing to 1) against validation datasets.

## When NOT to use

- When only HR-MS/MS information is available and no retention time or reference data exist — in that case MetFrag2.2 scored much worse (30 Top 1 ranks with PubChem vs. 336 with added RT and reference information)
- When spectra are dominated by one or only a few fragments (e.g. a water loss) that most candidates can explain — they lack information to distinguish candidates regardless of weights
- When a dedicated spectral library search against standards (e.g. MassBank) would directly confirm the compound instead of in silico ranking

## Inputs

- HR-MS/MS peak lists of merged spectra (mzClust_hclust in xcms, eppm = 5e-6, eabs = 0.001 Da) with known compound identities
- Candidate structure lists from PubChem/ChemSpider or user-supplied SDF
- Retention times per substance (e.g. from MassBank records) for RT–log P model fitting
- MetFrag2.2 parameter files

## Outputs

- Per-spectrum candidate rankings from MetFrag2.2 with the optimized weight set
- Top 1 / Top 3 / Top 5 correct-candidate counts and rank percentages per dataset (e.g. verification_rankings.csv)

## How to apply

Retrieve candidate structures from PubChem/ChemSpider (or a supplied SDF), fragment each candidate via MetFrag2.2's in silico bond-dissociation approach, and compare generated fragments with the measured product ions. Weight the three scoring terms by randomly drawing 1000 combinations such that ωFrag+ωRT+ωRefs = 1, where the RT term comes from an RT–log P model established per dataset via tenfold cross-validation, and references are retrieved via ChemSpider/PubChem web services. Filter stereoisomer redundancy by keeping only the best-scored candidate based on the first part of the InChIKey. Verify the chosen weights on independent merged spectra (e.g. UF, EQex, EQexPlus; 824 merged spectra total, merged with mzClust_hclust in xcms at eppm = 5×10^-6 and eabs = 0.001 Da, retaining only spectra with at least one fragment peak besides the precursor), counting correct candidates in Top 1, Top 3 and Top 5 ranks.

## Related tools

- **MetFrag2.2 / MetFragCommandLine** (Executes in silico bond-dissociation fragmentation, candidate scoring with configurable weights (ωFrag+ωRT+ωRefs = 1) and InChIKey-based stereoisomer filtering) — http://c-ruttkies.github.io/MetFrag/
- **xcms** (Merges replicate spectra per dataset with mzClust_hclust (eppm = 5e-6, eabs = 0.001 Da) before ranking)
- **OpenBabel** (Converts SMILES to InChIKeys (e.g. for suspect lists such as STOFF-IDENT) used as MetFrag input)
- **metfRag** (R interface to MetFrag for running the scoring configuration from R) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
mvn clean install -pl MetFragCommandLine -am
```

## Evaluation signals

- Weight combinations must satisfy ωFrag + ωRT + ωRefs = 1 (e.g. drawn from 1000 random combinations during optimization)
- On the verification datasets (UF, EQex, EQexPlus), Top 1 rank percentages should fall in the ~63–82 % range, with a reference dataset like the original (71 %) within that range
- Per-dataset Top 1 / Top 3 / Top 5 correct-candidate counts are reproducible (e.g. on Hill et al.: top 1/3/5 improved from 8 to 9, 20 to 24 and 28 to 34 vs. MetFrag2010)
- Stereoisomer redundancy check: each first-block InChIKey group retains exactly one (best-scored) candidate in the ranking
- Output CSV (e.g. verification_rankings.csv) contains per-dataset ranking statistics for all 824 merged spectra

## Limitations

- Larger compound databases make ranking harder over time (PubChem has doubled since 2009), so weights verified on 2015 database versions may degrade on current releases
- Combining MetFrag2.2 with CFM-ID was not further validated beyond this article, and further improvements would require retraining CFM-ID on Orbitrap data
- Correct Top 1 ranking does not equal confirmed identification — confirmation still requires reference standards
- The MetFrag2.2 dataset-specific parameter/spectral/candidate/ranking files (EA, UF, EQEx, EQExPlus, HILL) are only available as external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/

## Evidence

- [results] Weight optimization by randomly drawing 1000 combinations summing to 1: [evidence span withheld — anchor status: not_found]
- [results] InChIKey-based stereoisomer filtering: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [abstract] Verified weights on three additional datasets: "The optimal parameters and weights were verified using three additional datasets of 824 merged HR-MS/MS spectra in total."
- [results] Spectra merging parameters in xcms: [evidence span withheld — anchor status: not_found]
- [readme] MetFragCommandLine build dependency: "MetFrag commandline version depends on MetFragLib"
