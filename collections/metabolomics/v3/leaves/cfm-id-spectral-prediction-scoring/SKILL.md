---
name: cfm-id-spectral-prediction-scoring
description: Use when ranking candidate compounds for Orbitrap HR-MS/MS spectra by
  fragmentation-only scoring (no reference or retention terms), particularly when
  comparing in silico fragmenters or when MetFrag's bond-dissociation scoring alone
  underperforms.
when_to_use_negative:
- When the goal is the best possible identification with reference and retention information
  available — the article's combined MetFrag2.2 workflow (420 Top 1 with ChemSpider,
  336 with PubChem) vastly outperforms fragmentation-only CFM-ID, so use the full
  MetFrag2.2 scoring instead.
- When spectra are dominated by one or only a few fragments (e.g. a single water loss)
  that most candidates can explain — the article notes such spectra 'simply do not
  contain enough information' to distinguish candidates, so any fragmentation-only
  scorer including CFM-ID will struggle.
- When ranking across a much larger or differently instrumented dataset without re-verification
  — the article's 43 Top 1 figure was established on the 473-spectrum Hill dataset
  and could shift after retraining CFM-ID on Orbitrap data.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CFM-ID
  role: Predicts in silico fragments for candidate structures via competitive fragment
    modelling and scores them against measured product ions to produce fragmentation-only
    candidate rankings
- name: MetFrag2.2
  role: Baseline comparator and provider of the candidate-retrieval, InChIKey-filtering,
    and ranking pipeline that CFM-ID is plugged into for benchmarking
  repo: http://c-ruttkies.github.io/MetFrag/
- name: PubChem
  role: Source of candidate structures for each precursor mass in the benchmarking
    comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:d068bc66e8a7adda393734135797e5574dd3b6a72c88a35bf04472192f058787
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MetFrag2.2
  - PubChem
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cfm-id-spectral-prediction-scoring

## Summary

Applies CFM-ID's competitive fragment modelling to predict in silico fragments for candidate structures and rank them against measured HR-MS/MS spectra using fragmentation information only, as benchmarked by MetFrag2.2 on the 473-spectrum Hill dataset (CFM-ID achieved 43 Top 1 ranks vs. MetFrag2.2's 30 with PubChem candidates).

## When to use

Use when ranking candidate compounds for Orbitrap HR-MS/MS spectra by fragmentation-only scoring (no reference or retention terms), particularly when comparing in silico fragmenters or when MetFrag's bond-dissociation scoring alone underperforms.

## When NOT to use

- When the goal is the best possible identification with reference and retention information available — the article's combined MetFrag2.2 workflow (420 Top 1 with ChemSpider, 336 with PubChem) vastly outperforms fragmentation-only CFM-ID, so use the full MetFrag2.2 scoring instead.
- When spectra are dominated by one or only a few fragments (e.g. a single water loss) that most candidates can explain — the article notes such spectra 'simply do not contain enough information' to distinguish candidates, so any fragmentation-only scorer including CFM-ID will struggle.
- When ranking across a much larger or differently instrumented dataset without re-verification — the article's 43 Top 1 figure was established on the 473-spectrum Hill dataset and could shift after retraining CFM-ID on Orbitrap data.

## Inputs

- Merged HR-MS/MS spectra (e.g. 473 merged Orbitrap spectra from the Hill dataset, merged via mzClust_hclust with eppm = 5e-6 and eabs = 0.001 Da)
- Candidate structure sets retrieved from PubChem for each precursor mass
- Measured product ion m/z values from the spectra

## Outputs

- Per-spectrum candidate rankings using fragmentation-only CFM-ID scores
- Ranking summary with Top 1 and Top 5 counts (benchmark: 43 Top 1 and 170 Top 5 with PubChem)
- InChIKey-filtered candidate list (best-scored candidate per first InChIKey block)

## How to apply

For each merged HR-MS/MS spectrum, retrieve candidate structures from the compound database (e.g. PubChem) and apply CFM-ID's competitive fragment modelling to generate predicted fragments for every candidate. Score candidates solely by how well their predicted fragments explain the measured product ions, deliberately excluding reference and retention information terms so results are directly comparable to fragmentation-only MetFrag2.2. Apply the article's stereoisomer-reduction step — filter candidates to keep only the best-scored candidate per first InChIKey block — before ranking. Produce per-spectrum candidate rankings and count Top 1 (and optionally Top 5) hits. In the article's benchmark on 473 merged Orbitrap spectra from the Hill dataset with PubChem candidates, CFM-ID scored 43 Top 1 and 170 Top 5 ranks, and the authors note further gains could come from retraining CFM-ID on Orbitrap data.

## Related tools

- **CFM-ID** (Predicts in silico fragments for candidate structures via competitive fragment modelling and scores them against measured product ions to produce fragmentation-only candidate rankings)
- **MetFrag2.2** (Baseline comparator and provider of the candidate-retrieval, InChIKey-filtering, and ranking pipeline that CFM-ID is plugged into for benchmarking) — http://c-ruttkies.github.io/MetFrag/
- **PubChem** (Source of candidate structures for each precursor mass in the benchmarking comparison)

## Examples

```
python run_cfm_ranking.py --spectra hill_merged.ms --candidates pubchem.sdf --filter-inchikey-block1 --out ranking_summary.csv
```

## Evaluation signals

- Reproducing the article's benchmark on the 473 merged Hill-dataset Orbitrap spectra with PubChem candidates should yield exactly 43 Top 1 ranks and 170 Top 5 ranks for CFM-ID.
- The comparison baseline should also reproduce MetFrag2.2's 30 Top 1 and 145 Top 5 counts on the same spectra.
- After InChIKey filtering, only one candidate per first InChIKey block should remain per spectrum (stereoisomer redundancy removed).
- Rankings must be computed without reference or retention information terms — verify the scoring used fragmentation match only.
- Output should include a ranking_summary.csv containing per-spectrum rankings and the aggregated Top 1 counts.

## Limitations

- Further validation of the MetFrag2.2+CFM-ID combination was beyond the scope of the current article, as further improvements could be made by retraining CFM-ID on Orbitrap data.
- Mass spectra dominated by one or only a few fragments (e.g. a water loss) that can be explained by most candidates simply do not contain enough information to distinguish candidates.
- Candidate ranking becomes more challenging over time as PubChem grows — the article notes PubChem had doubled in size since the original 2010 MetFrag paper.
- No sufficiently pure reference standard was available to confirm some identifications (e.g. 4-isopropylbenzenesulfonic acid), so Top 1 rank does not guarantee a chemically confirmed hit.

## Evidence

- [abstract] Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database.: "Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database."
- [intro] Implemented in CFM-ID (competitive fragment modelling), this can be used to assign fragments to spectra: "Implemented in CFM-ID (competitive fragment modelling), this can be used to assign fragments to spectra"
- [results] Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [results] These spectra were merged using the mzClust_hclust function in xcms [53] (parameters eppm = 5×10-6 and eabs = 0.001 Da): [evidence span withheld — anchor status: not_found]
- [results] Further validation was beyond the scope of the current article, as further improvements could be made by retraining CFM-ID on Orbitrap data: "Further validation was beyond the scope of the current article, as further improvements could be made by retraining CFM-ID on Orbitrap data"
