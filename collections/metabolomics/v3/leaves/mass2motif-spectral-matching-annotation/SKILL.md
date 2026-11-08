---
name: mass2motif-spectral-matching-annotation
description: Use when MS2LDA inference has produced Mass2Motif pseudo-spectra (fragment
  and neutral-loss masses with probabilities) that need automated putative substructure
  annotations, e.g. to label discovered motifs across beer, urine, or GNPS spectra
  without manual curation.
when_to_use_negative:
- The input is raw MS1/MS2 centroid data that has not yet been decomposed by MS2LDA
  into Mass2Motifs — run LDA inference first.
- Neutral-loss information is essential to the annotation and cannot be dropped, since
  spectral matching tools 'do not currently support' losses and they must be omitted.
- The mixture is simple or MS2 spectra are information-poor (no ramped/stepped collision
  energy), where MS2LDA motifs themselves are unreliable.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MS2LDA
  role: Produces the Mass2Motif pseudo-spectra that are represented as spectra and
    subjected to matching; also provides the LDA probabilities used as match weights.
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: Style of substructure/spectral matching applied to motif spectra for automatic
    annotation (contrast class for the unsupervised approach).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:3f92f5500f4886f4accbd6ecc94f02d9b92f1b52087ac97f1b9371a60d351a5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2Analyzer
  - MS2LDA
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# mass2motif-spectral-matching-annotation

## Summary

Automatically annotates MS2LDA-discovered Mass2Motifs by representing each motif as a pseudo-spectrum and subjecting it to spectral (substructure) matching, validating fragment-based substructure labels such as acetylations, nitro groups, and sugar fragments.

## When to use

Use when MS2LDA inference has produced Mass2Motif pseudo-spectra (fragment and neutral-loss masses with probabilities) that need automated putative substructure annotations, e.g. to label discovered motifs across beer, urine, or GNPS spectra without manual curation.

## When NOT to use

- The input is raw MS1/MS2 centroid data that has not yet been decomposed by MS2LDA into Mass2Motifs — run LDA inference first.
- Neutral-loss information is essential to the annotation and cannot be dropped, since spectral matching tools 'do not currently support' losses and they must be omitted.
- The mixture is simple or MS2 spectra are information-poor (no ramped/stepped collision energy), where MS2LDA motifs themselves are unreliable.

## Inputs

- Mass2Motif pseudo-spectra (fragment and neutral-loss masses with LDA posterior probabilities) from MS2LDA decomposition of MS2 spectra
- Reference substructure fragment sets / spectral libraries (e.g. Mass Bank, NIST MS/MS-derived fragment lists)
- Characterized positive-mode beer Mass2Motif pseudo-spectra from the deposited beer dataset (dx.doi.org/10.5525/gla.researchdata.313)

## Outputs

- Automated substructure annotations per Mass2Motif (motif ID, matched substructure label, match probability score)
- Per-motif agreement table comparing automated vs. manually characterized annotations (motif_annotation_agreement.csv)
- Tally of correctly annotated motifs (e.g. 19 of the characterized positive-mode beer motifs)

## How to apply

Represent each Mass2Motif as a pseudo-spectrum whose peaks are the fragment masses (and neutral losses, if the matcher supports them) weighted by their posterior probabilities from the LDA decomposition. Subject each motif spectrum to spectral matching against known substructure fragment sets (MS2Analyzer-style matching) at the paper's probability threshold of 0.1; because standard spectral matching tools do not support neutral losses, omit loss entries from the motif spectra before matching. Apply the procedure to motifs whose occurrence spans the dataset (Mass2Motif degree ranged from 1 to over 200 spectra), prioritizing motifs recurring across many spectra. Validate on motifs with known structural characterizations — in the beer positive-ionization data 30–40 motifs per file were manually characterized, and the automated procedure correctly annotated 19 of these; acceptance of a match uses the same 0.1 probability cutoff that yielded 81.5% correct substructure annotation at the spectrum level.

## Related tools

- **MS2LDA** (Produces the Mass2Motif pseudo-spectra that are represented as spectra and subjected to matching; also provides the LDA probabilities used as match weights.) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Style of substructure/spectral matching applied to motif spectra for automatic annotation (contrast class for the unsupervised approach).)

## Examples

```
python motif_matching.py --motif-spectra beer3_pos_mass2motifs.ms2 --strip-neutral-losses --prob-threshold 0.1 --out motif_annotation_agreement.csv
```

## Evaluation signals

- Tally of automatically annotated motifs against the structurally characterized positive-mode beer motifs must equal 19 (per SI Appendix section S2.13).
- All neutral-loss entries must be removed from motif spectra before matching; verify no loss masses survive into the matched input.
- Every reported substructure annotation must have an associated probability ≥ 0.1 (the paper's validation cutoff).
- Per-motif agreement table must contain the 30–40 characterized motifs per file with a manual annotation, an automated annotation, a probability, and an agreement flag.
- Cross-check on GNPS/Mass Bank data: decomposing 5,670 spectra into 500 Mass2Motifs should still allow the beer-characterized motifs (kept fixed during inference) to be re-annotated consistently.

## Limitations

- Neutral losses are not currently supported by spectral matching tools and had to be omitted from automated annotation, losing loss-derived substructure evidence.
- Automated annotation was validated only on characterized positive-mode beer motifs; behavior in other ionization modes or matrices is not reported.
- MS2LDA works best for complex mixtures with information-rich MS2 spectra (ramped or stepped collision energy); motif quality, and hence annotation, degrades on simple mixtures or sparse spectra.
- Matching yields putative, not definitive, annotations — at probability threshold 0.1 only 81.5% and 63.3% of spectrum-level substructure annotations were correct.

## Evidence

- [discussion] MS2LDA decomposition scale: [evidence span withheld — anchor status: not_found]
- [discussion] Information-rich spectra requirement: [evidence span withheld — anchor status: not_found]
