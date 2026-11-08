---
name: retention-time-prior-integration
description: Use when the MS1 database contains RT entries (e.g. '30;60' elution ranges)
  for compounds, and LC-MS feature annotations must be filtered or reweighted by whether
  the feature's measured retention time falls within that expected range.
when_to_use_negative:
- When the compound database has no 'RT' information — the RT prior cannot be applied
  and annotations are scored on mass accuracy, isotope pattern and prior knowledge
  alone.
- When the LC method or column is drastically different from the one used to build
  the database RT entries — the '30;60'-style elution ranges will be invalid and may
  incorrectly penalise true annotations.
- When hard filtering of annotations by RT is desired — ipaPy2 treats RT as a Bayesian
  prior with soft probabilities (pRTNone=0.8, pRTout=0.4), not a cutoff.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ipaPy2
  role: Implements the Bayesian annotation engine whose MS1annotation() and MSMSannotation()
    functions apply the retention-time prior (pRTNone/pRTout) to candidate annotations.
  repo: https://github.com/francescodc87/ipaPy2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:b412c85d537f9b3f1013dcb614933bef0f350b954cd583cacac0995dcd02c23a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - Python
  - ipaPy2
  - numpy
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# retention-time-prior-integration

## Summary

Integrates known retention time (RT) ranges from a compound database into the IPA Bayesian annotation of LC-MS untargeted metabolomics features, adjusting the posterior annotation probabilities of each candidate.

## When to use

Use when the MS1 database contains RT entries (e.g. '30;60' elution ranges) for compounds, and LC-MS feature annotations must be filtered or reweighted by whether the feature's measured retention time falls within that expected range.

## When NOT to use

- When the compound database has no 'RT' information — the RT prior cannot be applied and annotations are scored on mass accuracy, isotope pattern and prior knowledge alone.
- When the LC method or column is drastically different from the one used to build the database RT entries — the '30;60'-style elution ranges will be invalid and may incorrectly penalise true annotations.
- When hard filtering of annotations by RT is desired — ipaPy2 treats RT as a Bayesian prior with soft probabilities (pRTNone=0.8, pRTout=0.4), not a cutoff.

## Inputs

- pandas dataframe of MS1 annotations from ipaPy2 (keyed by feature id, with 'RT' entries from the database)
- compound database file with 'RT' column, e.g. DB/IPA_MS1.csv
- feature RT values from the clustered MS1 feature table (e.g. ExampleDatasets/README/df_test_pos.csv)

## Outputs

- posterior probability per annotation ('post' column) adjusted by the RT prior (pRTNone/pRTout)
- list of candidate annotations per feature retaining 'prior' and 'post' columns

## How to apply

In the IPA pipeline, this skill is exercised inside the MS1annotation() or MSMSannotation() functions when the database (e.g. DB/IPA_MS1.csv) has a non-empty 'RT' column for an entry. For each candidate annotation, ipaPy2 checks whether the feature's measured RT lies within the compound's known elution range; if it does, the annotation is retained with a probability governed by the pRTNone parameter, and if it falls outside, the probability is multiplied by the pRTout parameter (default 0.4). Entries with no RT information receive the pRTNone default (0.8), so the RT prior acts as a soft multiplicative penalty rather than a hard filter. The rationale is to leverage prior LC method knowledge — instrument-specific retention behaviour — to down-rank chemically plausible but chromatographically unlikely annotations, thereby reducing false positives in a Bayesian posterior rather than by an arbitrary cutoff.

## Related tools

- **ipaPy2** (Implements the Bayesian annotation engine whose MS1annotation() and MSMSannotation() functions apply the retention-time prior (pRTNone/pRTout) to candidate annotations.) — https://github.com/francescodc87/ipaPy2

## Examples

```
from ipaPy2 import ipa; annots = ipa.MS1annotation(df, allAddsPos, ppm=3, ncores=1)  # annotations dictionary integrates RT priors via pRTNone/pRTout
```

## Evaluation signals

- Each annotation row carries a populated 'post' column; compare 'prior' vs 'post' to confirm the RT prior has shifted probabilities for entries with a defined RT.
- For database entries whose RT is empty, confirm the posterior is only affected by mass accuracy and isotope-pattern scores, not by a spurious RT penalty.
- Check that a feature whose RT falls outside a compound's listed range (e.g. outside '30;60') receives a lower 'post' than the same annotation when RT falls inside the range.
- Verify that no annotation has been dropped outright — the RT prior is multiplicative (pRTout=0.4), so all candidates should still appear in the annotations dictionary with non-zero probabilities.

## Limitations

- RT information is optional in the database; entries without it get the default pRTNone (0.8) rather than a true RT-based adjustment.
- The RT prior assumes the LC method used for the new samples is comparable to the method from which the database RT entries were derived; the skill has no mechanism to recalibrate RT across instruments or gradients.
- Because the prior is soft (pRTout=0.4), true positives with slightly off-target RT may still be retained, so the skill reduces rather than eliminates chromatographically implausible annotations.

## Evidence

- [readme] The IPA method requires a pandas dataframe containing the database against which the annotation is performed.: "The IPA method requires a pandas dataframe containing the database against which the annotation is performed."
- [readme] RT: if known, retention time range (in seconds) where this compound is expected to elute (e.g., '30;60') - *optional*: "RT: if known, retention time range (in seconds) where this compound is expected to elute (e.g., '30;60') - *optional*"
- [abstract] An automated annotation method for LC-MS-based untargeted metabolomics experiments that provides statistically rigorous estimates of the probabilities: "The Integrated Probabilistic Annotation (IPA) is an automated annotation method for LC-MS-based untargeted metabolomics experiments that provides statistically rigorous estimates of the probabilities"
- [methods] The help of this function provides a detailed description of it.: "This is achieved through the map_isotope_patterns(). The help of this function provides a detailed description of it."
- [methods] If no fragmentation data is available, the MS1annotation() function should be used: "If no fragmentation data is available, the MS1annotation() function should be used"
