---
name: gnps-query-relaxation-annotation
description: Use when FBMN/GNPS molecular networking leaves many features unannotated
  under the strict library search, and per-feature GNPS indicator values (c_g, t_g,i,
  t_g,s) are needed for MS2DECIDE K-estimation despite weak analog matches.
when_to_use_negative:
- When a single strict GNPS library search suffices (strict mode uses 0.02 Da mass
  difference tolerance and 0.001 threshold and launches only one FBMN job instead
  of 27)
- When the goal is novelty assessment from a zero ISDB-LOTUS (0.02 Da strict library
  search) answer rather than maximizing GNPS annotation coverage
- When no GNPS account or FBMN job capacity is available, since the iterative workflow
  runs 27 FBMN jobs and can take up to three hours
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: Consumes the GNPS I indicators (c_g, t_g,i, t_g,s) as part of its K-value
    expert aggregation of GNPS, Sirius 6, and ISDB-LOTUS annotations
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Target annotation platform queried with the strict q(6, 0.02) query and progressively
    relaxed variants; iterative workflow launches 27 FBMN jobs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:d52c390cbd98d39ef16d781b87164739474eb70e3000e5691120a9c9b6de9427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - MS2DECIDE
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gnps-query-relaxation-annotation

## Summary

Implements GNPS I, the MS2DECIDE alternative use of GNPS molecular networking that improves annotation availability via progressive query relaxation: start from the strict GNPS query q(6, 0.02) and relax constraints stepwise, discounting each returned cosine score by the relaxation degree reached (Table S4).

## When to use

Use when FBMN/GNPS molecular networking leaves many features unannotated under the strict library search, and per-feature GNPS indicator values (c_g, t_g,i, t_g,s) are needed for MS2DECIDE K-estimation despite weak analog matches.

## When NOT to use

- When a single strict GNPS library search suffices (strict mode uses 0.02 Da mass difference tolerance and 0.001 threshold and launches only one FBMN job instead of 27)
- When the goal is novelty assessment from a zero ISDB-LOTUS (0.02 Da strict library search) answer rather than maximizing GNPS annotation coverage
- When no GNPS account or FBMN job capacity is available, since the iterative workflow runs 27 FBMN jobs and can take up to three hours

## Inputs

- Per-feature GNPS FBMN molecular networking annotation results (cosine scores, precursor masses, shared-peak counts, InChI availability)
- GNPS account credentials (username, password, email) for launching FBMN jobs
- Feature quantification table (.csv, e.g. MZmine >2.53 FBMN export) and MS/MS spectra in .mgf format

## Outputs

- gnps_i_indicators.csv — per-feature GNPS I indicator values (c_g, t_g,i, t_g,s) after discounting and fallback rules
- gnps_i_procedure.py — saved implementation of the relaxation procedure
- empty.tsv — optional report of empty annotations generated to complete missing InChIs or SMILES

## How to apply

Load the per-feature GNPS molecular networking results (cosine scores, masses, shared-peak counts, InChI availability) into a pandas DataFrame, then for each feature issue the strict GNPS query q(6, 0.02) (6 shared peaks, 0.02 Da mass tolerance) and record the top match score. If no strict match is returned, progressively relax the query: widen the mass tolerance to 0.1 Da, then to 10 Da, and relax the shared-peak constraint, keeping the first match score returned by GNPS and discounting it according to the relaxation degree reached as specified in Table S4. Apply the fallback defaults: when a match is found with score-without-InChI only, set t_g,i = t_g,s = 0.7 (consistent with the README's rule for instances where the GNPS–Sirius Tanimoto cannot be computed), and when no match at all is found, set c_g = t_g,i = t_g,s = 0. Assemble the per-feature indicators (c_g, t_g,i, t_g,s) into gnps_i_indicators.csv and save the implementation as gnps_i_procedure.py. Note the practical cost: the iterative workflow launches 27 FBMN jobs from the GNPS account (versus one for strict) and can take up to three hours, so raising the library score threshold from 0.001 to e.g. 0.1 is recommended to avoid FBMN failures.

## Related tools

- **MS2DECIDE** (Consumes the GNPS I indicators (c_g, t_g,i, t_g,s) as part of its K-value expert aggregation of GNPS, Sirius 6, and ISDB-LOTUS annotations) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Target annotation platform queried with the strict q(6, 0.02) query and progressively relaxed variants; iterative workflow launches 27 FBMN jobs)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()  # choose 'iterative' at the GNPS workflow prompt to run the 27-job iterative weighted analog search
```

## Evaluation signals

- Every feature row in gnps_i_indicators.csv carries a complete set of indicator values (c_g, t_g,i, t_g,s) consistent with the Table S4 discount rules
- Fallback correctness: rows matched with score-without-InChI only show t_g,i = t_g,s = 0.7; rows with no match show c_g = t_g,i = t_g,s = 0
- Spot-check features matched under the strict query against the q(6, 0.02) (6 shared peaks, 0.02 Da) results — they should be unaffected by discounting
- Iterative-mode sanity: the workflow should have launched 27 FBMN jobs (strict launches 1), and FBMN threshold should have been raised above 0.001 (e.g. 0.1) to avoid job failures
- Optional empty.tsv report matches the set of features left with empty annotations after the iterative weighted analog search

## Limitations

- The iterative weighted analog search launches 27 FBMN jobs from the user's GNPS account and can take up to three hours
- With the default library score threshold of 0.001, some FBMN GNPS workflow failures can occur; increasing the threshold (e.g. to 0.1) is recommended
- In scenarios with no GNPS match (or no Sirius match), the GNPS–Sirius Tanimoto cannot be calculated, so defaults of 0.7 for T_gs and T_gi are imposed, which is an approximation rather than a measured similarity
- Scores are discounted according to relaxation degree, so annotations recovered only at high relaxation carry systematically lower indicator values than strict-match annotations

## Evidence

- [supplementary] GNPS I queries GNPS starting with the strict query q(6, 0.02) and progressively relaxes constraints (0.1 Da then 10 Da mass difference, then shared-peak requirements), keeping the first match score returned by GNPS; the score is discounted according to the relaxation degree reached.: [evidence span withheld — anchor status: not_found]
- [readme] MS2DECIDE integrates GNPS (enhanced with a iterative weighted analog search), ISDB-LOTUS and Sirius6 annotations to deliver reliable recommendation even in uncertain scenarios.: [evidence span withheld — anchor status: external_doc]
- [readme] `iterative`: for iterative weighted analog search (can take up to three hours). ... the iterative workflow launches 27 FBMN jobs from your GNPS account. In the case of `strict`, only one job will be launched.: [evidence span withheld — anchor status: external_doc]
- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] - `strict`: Uses a typical mass difference tolerance of 0.02 Da and threshold value of 0.001. ... We recommend to increase the threshold value (for e.g., 0.1).: [evidence span withheld — anchor status: external_doc]
