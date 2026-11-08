---
name: precursor-mass-tolerance-matching
description: Use when you have experimental MGF-format MS/MS spectra (e.g., human
  blood metabolomics) plus one or more reference MS/MS libraries, and you need to
  count how many spectra can be annotated by library search under cosine similarity
  thresholds.
when_to_use_negative:
- Queries lack precursor m/z metadata (e.g., MS1-only data), since the 10 ppm precursor
  tolerance cannot be evaluated
- The goal is de novo structure elucidation or ML-based MS/MS prediction rather than
  reference spectral library search
- Input spectra have not undergone basic quality filtering (low-resolution or zero/uniform-intensity
  fragments), which the Spectraverse curation pipeline removes before matching
tools:
- name: matchms
  role: Loads MGF spectra/libraries, applies cleaning filters, and performs cosine-similarity
    library search with precursor m/z tolerance
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides cosine score calculation used by the Spectraverse preprocessing/search
    code
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:773dbaba21d7537312651bb99b74cbf08bb98c6cfd404ee5f5c28d7c6b72f5f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# precursor-mass-tolerance-matching

## Summary

Annotates experimental MS/MS spectra by matching them against reference spectral libraries (e.g., Spectraverse, MassSpecGym, NPLIB1) using matchms cosine similarity with a precursor m/z tolerance constraint and a minimum matched-fragment-ion requirement.

## When to use

Use when you have experimental MGF-format MS/MS spectra (e.g., human blood metabolomics) plus one or more reference MS/MS libraries, and you need to count how many spectra can be annotated by library search under cosine similarity thresholds.

## When NOT to use

- Queries lack precursor m/z metadata (e.g., MS1-only data), since the 10 ppm precursor tolerance cannot be evaluated
- The goal is de novo structure elucidation or ML-based MS/MS prediction rather than reference spectral library search
- Input spectra have not undergone basic quality filtering (low-resolution or zero/uniform-intensity fragments), which the Spectraverse curation pipeline removes before matching

## Inputs

- Experimental MS/MS spectra in MGF format (e.g., human blood metabolomics spectra)
- Reference MS/MS spectral libraries in MGF (Spectraverse v1.0.1, MassSpecGym, NPLIB1)
- Python environment with matchms (and optionally SpectralEntropy) installed

## Outputs

- Per-library, per-cosine-threshold annotation-count table
- Set of annotated experimental spectra with matched reference IDs and scores

## How to apply

Load the experimental MGF spectra and each reference library (Spectraverse v1.0.1 from Zenodo, MassSpecGym, NPLIB1) into matchms spectrum objects, applying matchms metadata cleaning and peak filters first. For each library independently, run a matchms spectral library search using cosine similarity scoring with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions, so results are directly comparable across libraries. Repeat the search at multiple cosine similarity thresholds and count annotated experimental spectra per library per threshold. Assemble per-library, per-threshold counts into a single table; in the source comparison, Spectraverse annotated the most human blood MS/MS spectra at every threshold evaluated, consistent with Spectraverse containing 488,630 spectra from 44,237 unique small molecules. Note that matchms supports sparse score storage so comparisons of several hundred thousand spectra remain tractable.

## Related tools

- **matchms** (Loads MGF spectra/libraries, applies cleaning filters, and performs cosine-similarity library search with precursor m/z tolerance) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides cosine score calculation used by the Spectraverse preprocessing/search code) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
from matchms import calculate_scores; from matchms.similarity import CosineGreedy; scores = calculate_scores(references, queries, CosineGreedy(tolerance=0.01, matched_peaks=3))
```

## Evaluation signals

- Annotation-count table covers all three libraries at all cosine similarity thresholds evaluated
- Spectraverse shows the highest annotation count at every threshold, consistent with the paper's reported comparison against MassSpecGym
- Every reported match satisfies the search criteria: precursor within 10 ppm and at least 3 matched fragment ions
- Searches were run independently against each library with identical settings so counts are comparable
- All input spectra and libraries were successfully loaded as matchms spectrum objects after cleaning filters

## Limitations

- Curation of the underlying Spectraverse library relied on manual review of only several thousand MS/MS spectra; pitfalls may remain beyond what was manually reviewed
- Near-duplicate spectra removal was only reported for MassSpecGym comparisons; completeness of de-duplication for Spectraverse itself is not stated
- Count depends on the cosine thresholds chosen; only a fraction of MS/MS spectra can typically be annotated because libraries are limited in size and biased toward well-studied, commercially available compounds

## Evidence

- [methods] 10 ppm tolerance with 3 matched peaks: [evidence span withheld — anchor status: not_found]
- [readme] matchms cosine library search: [evidence span withheld — anchor status: external_doc]
- [other] Spectraverse comparison finding: [evidence span withheld — anchor status: not_found]
