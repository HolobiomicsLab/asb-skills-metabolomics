---
name: hr-msms-spectrum-merging
description: Use when multiple HR-MS/MS spectra of the same precursor/compound exist
  and must be consolidated into one representative spectrum before in silico fragmentation
  scoring.
when_to_use_negative:
- Each precursor already has a single, non-redundant spectrum — merging would discard
  legitimate replicate variation without benefit.
- Spectra come from different instrument types or collision energies that should not
  be pooled; mzClust_hclust assumes comparable peak positions.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: xcms
  role: Provides mzClust_hclust, the function actually invoked to cluster and merge
    HR-MS/MS spectra with the specified ppm and absolute-mass tolerances.
- name: MetFrag2.2
  role: 'Downstream consumer: merged spectra are the input spectra against which in
    silico fragments from candidate structures are scored for Top-1/3/5 ranking.'
  repo: http://c-ruttkies.github.io/MetFrag/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:5b44f46549fc4e8b8532f6705e9cf804cd48ca1a4a91c431df041a69aa4bc317
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragLib
  - OpenBabel
  - xcms
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hr-msms-spectrum-merging

## Summary

Merge replicate high-resolution MS/MS spectra of the same analyte into a single consensus spectrum using mzClust_hclust in xcms (eppm = 5×10⁻⁶, eabs = 0.001 Da) so that downstream candidate ranking operates on one spectrum per compound.

## When to use

Use when multiple HR-MS/MS spectra of the same precursor/compound exist and must be consolidated into one representative spectrum before in silico fragmentation scoring.

## When NOT to use

- Each precursor already has a single, non-redundant spectrum — merging would discard legitimate replicate variation without benefit.
- Spectra come from different instrument types or collision energies that should not be pooled; mzClust_hclust assumes comparable peak positions.

## Inputs

- Raw HR-MS/MS spectra (e.g. Orbitrap tandem MS spectra)
- Per-spectrum precursor m/z and fragment peak lists

## Outputs

- Merged consensus spectra, one per compound/precursor
- Filtered set of merged spectra with ≥1 non-precursor fragment peak

## How to apply

Load the raw HR-MS/MS spectra and call the mzClust_hclust function in xcms with parameters eppm = 5×10⁻⁶ and eabs = 0.001 Da to cluster spectra by precursor mass and merge them. After merging, retain only spectra that contain at least one fragment peak other than the precursor, since spectra dominated solely by the precursor carry no discriminating information for candidate ranking. Apply this per dataset (e.g. EA, UF, EQEx, EQExPlus) before extracting retention times for RT–logP modelling and before feeding spectra into MetFrag2.2, so that each compound contributes exactly one merged spectrum to the Top-1/Top-3/Top-5 evaluation over the 824-spectra validation pool.

## Related tools

- **xcms** (Provides mzClust_hclust, the function actually invoked to cluster and merge HR-MS/MS spectra with the specified ppm and absolute-mass tolerances.)
- **MetFrag2.2** (Downstream consumer: merged spectra are the input spectra against which in silico fragments from candidate structures are scored for Top-1/3/5 ranking.) — http://c-ruttkies.github.io/MetFrag/

## Examples

```
R: library(xcms); merged <- mzClust_hclust(spectra, eppm = 5e-6, eabs = 0.001)
```

## Evaluation signals

- Each input spectrum maps to exactly one merged spectrum per precursor after mzClust_hclust (eppm=5e-6, eabs=0.001 Da).
- Merged spectra lacking any fragment peak besides the precursor are excluded — count of retained spectra matches the article's retained set.
- Retention times extracted from merged MassBank records support a tenfold-cross-validated RT–logP model per dataset.
- Downstream MetFrag2.2 rankings computed on merged spectra reproduce the reported Top 1 percentages of 63–82 % across the three validation datasets.
- Total retained merged spectra across validation datasets equal 824.

## Limitations

- Spectra dominated by one or a few fragments (e.g. a water loss) that most candidates can explain do not contain enough information to distinguish candidates, even after merging.
- Merging parameters were tuned for Orbitrap HR-MS/MS; re-verification may be needed for other instrument types.
- Spectra with no non-precursor fragment peak cannot contribute to candidate discrimination and are discarded.

## Evidence

- [results] Merging HR-MS/MS spectra with xcms before MetFrag scoring: [evidence span withheld — anchor status: not_found]
- [results] Retention of spectra with fragment peaks only: "retained where they contained at least one fragment peak other than the precursor"
- [abstract] Validation on 824 merged HR-MS/MS spectra: "The optimal parameters and weights were verified using three additional datasets of 824 merged HR-MS/MS spectra in total."
- [intro] Low-information spectra limit candidate ranking: "mass spectra that are dominated by one or only a few fragments (e.g. a water loss) that can be explained by most of the candidates simply do not contain enough information"
