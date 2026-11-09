---
name: mass-spectral-database-retrieval
description: Use when you have a curated compound list with labels or properties of
  interest and need the corresponding experimental mass spectra (EI-MS or ESI-MS/MS,
  positive ion mode) to run a spectrum-based prediction model, and the compounds'
  spectra are known to be deposited in MassBank or DrugBank.
when_to_use_negative:
- When the chemical structure of each compound is already known and a structure-based
  classifier (e.g., BitterPredict) is sufficient — the article's discussion notes
  the spectrum-to-structure-to-bitterness route is 'the most problematic stage' and
  covers fewer compounds
- When the compounds of interest have no deposited spectra in MassBank or DrugBank
  — retrieval will return nothing and the downstream spectrum-based model cannot be
  applied
- When only negative-ion-mode spectra are available — the article's BitterMasS implementation
  makes predictions based on spectra 'obtained in positive ion mode'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassBank
  role: Primary database queried for experimental EI-MS and ESI-MS/MS positive-ion-mode
    spectra of the curated bitter/nonbitter compounds
- name: DrugBank
  role: Supplementary spectral source for the medication and vitamin compound categories
    when MassBank coverage is insufficient
- name: BitterMasS
  role: Downstream Random Forest classifier that consumes the retrieved spectra to
    predict bitterness
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:0a9522065fe34116d7cda168711cc0eb1b8bc3a19cb4ce3f204f5125cfd92d4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  - DrugBank
  - MassBank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# mass-spectral-database-retrieval

## Summary

Retrieve experimental positive-ion-mode EI-MS and ESI-MS/MS spectra for a curated compound list from the MassBank database (supplemented by DrugBank spectra for medication and vitamin categories), so that spectra can be fed to a downstream classifier such as BitterMasS. This is the data-acquisition step that turned sensory-labeled compounds into 759 spectra covering 106 compounds for BitterMasS external testing.

## When to use

Use when you have a curated compound list with labels or properties of interest and need the corresponding experimental mass spectra (EI-MS or ESI-MS/MS, positive ion mode) to run a spectrum-based prediction model, and the compounds' spectra are known to be deposited in MassBank or DrugBank.

## When NOT to use

- When the chemical structure of each compound is already known and a structure-based classifier (e.g., BitterPredict) is sufficient — the article's discussion notes the spectrum-to-structure-to-bitterness route is 'the most problematic stage' and covers fewer compounds
- When the compounds of interest have no deposited spectra in MassBank or DrugBank — retrieval will return nothing and the downstream spectrum-based model cannot be applied
- When only negative-ion-mode spectra are available — the article's BitterMasS implementation makes predictions based on spectra 'obtained in positive ion mode'

## Inputs

- Curated compound list with names/identifiers (e.g., bitter and nonbitter compounds with sensory taste-panel labels extracted from literature publications)
- Compound category annotations (e.g., 'medication', 'vitamin') used to decide when DrugBank supplementation is needed

## Outputs

- Per-compound bundles of experimental positive-ion-mode EI-MS and/or ESI-MS/MS spectra retrieved from MassBank (and DrugBank for medication/vitamin categories)
- A spectral test/training set with compound-level coverage counts (e.g., 759 spectra for 106 compounds)

## How to apply

Start from the curated compound list (e.g., compounds with sensory taste-panel bitterness labels from literature publications), then query the MassBank database for experimental positive-ion-mode EI-MS and ESI-MS/MS spectra matching each compound. For specific compound categories — medications and vitamins — supplement MassBank results with DrugBank spectra to avoid missing records. Collect every available spectrum per compound rather than a single representative: in the BitterMasS external test set this yielded 759 spectra covering 106 compounds, which supports per-spectrum prediction followed by majority-vote aggregation downstream. Only positive ion mode is retained, matching the model's training domain; if a compound has no deposited spectrum in either source, it is simply excluded from the spectral set (this is why only ~30% of training compounds had spectra). Output a per-compound bundle of spectra ready for direct spectral classification or, alternatively, structure annotation via tools like CFM-ID 4.0 or MassGenie followed by structure-based prediction.

## Related tools

- **MassBank** (Primary database queried for experimental EI-MS and ESI-MS/MS positive-ion-mode spectra of the curated bitter/nonbitter compounds)
- **DrugBank** (Supplementary spectral source for the medication and vitamin compound categories when MassBank coverage is insufficient)
- **BitterMasS** (Downstream Random Forest classifier that consumes the retrieved spectra to predict bitterness)

## Evaluation signals

- Every retrieved spectrum is recorded as positive ion mode (EI-MS or ESI-MS/MS) — negative-mode spectra must not enter the set
- Compound-level coverage is reported alongside spectrum counts (e.g., 759 spectra across 106 compounds), so the many-to-one spectrum-to-compound mapping is explicit
- Compounds in medication and vitamin categories are checked for DrugBank supplementation, not just MassBank
- The resulting spectral set feeds per-spectrum predictions that can be aggregated by majority vote per compound — verify that at least the spectra needed for voting were retrieved per compound
- Coverage of the original compound list is documented; a large gap (e.g., only ~30% of compounds having spectra) is an expected, reportable outcome, not a silent failure

## Limitations

- Only a small fraction of known compounds have deposited mass spectra — in the BitterMasS training set only ~30% of initial bitter and nonbitter compounds had at least one mass spectrum identified, so retrieval may leave most of a compound list unspectra'd
- MassBank coverage is incomplete for certain categories, requiring DrugBank supplementation for medications and vitamins
- Most articles reporting experimental mass spectra do not include the actual spectra; standardized mass spectral deposition is lacking, which limits what any database retrieval can recover
- Retrieval is restricted to positive ion mode, excluding negative-mode data that might otherwise be available

## Evidence

- [discussion] Retrieval from MassBank as primary source with DrugBank supplementation: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
- [discussion] Retrieval is restricted to positive ion mode: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
- [discussion] Sparse spectral coverage is the norm, not an error: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
- [discussion] Lack of standardized spectral deposition limits what retrieval can recover: [evidence span withheld — anchor status: not_found]
- [abstract] External test set scale achieved by this retrieval workflow: "BitterMasS exhibited 67% precision and 93% recall for the first"
- [discussion] Alternative strategy the retrieval step enables or preempts: "annotating the structure from the spectrum (which is the most problematic stage)"
- [readme] README context for the broader tool family consuming retrieved spectra: "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
