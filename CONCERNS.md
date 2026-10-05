# Concerns, Flagged Errors and Risks

This file records weaknesses found while scoping the project. Each item is labelled:

- **Verified:** checked directly against the source data with a script.
- **Inference:** a reasoned judgement, not tested.
- **Unverified:** an open question that needs checking.

The data audit was run on the `ENTRIES` array in the uploaded *Fallacies & Biases — A Reference* HTML file.

## 1. Data defects

### 1.1 Dangling `related` links (Verified)
- **The problem:** 227 of the 1,170 `related` references (about 19%) point to slugs that don't exist in the dataset. For 4 entries, every related link is broken.
- **Why it went unnoticed:** the original page hides this. It filters unresolved slugs before rendering (`.filter(s => bySlug[s])`), so readers never see the gap.
- **The cause is sometimes a naming mismatch, not a missing concept.** For example, `affirming-a-disjunct` links to `false-dilemma`, but that concept exists under the slug `black-white-fallacy`. Other targets, such as `regret-aversion` and `modus-ponens`, have no entry at all.
- **Impact:** the "Related chains" game mode and any concept graph would be incomplete or misleading.
- **Fix:** build a list of alias slugs for renamed concepts, and decide case by case whether to drop each missing target or add it as an entry.

### 1.2 Inconsistent field types (Verified)
- **The problem:** `example` and `source` are strings in 369 entries but arrays in 15. The arrays are the 15 `alreadyTaught` entries, whose examples are split into "Everyday" and "Real-world" variants.
- **Impact:** any code expecting a string will break or display something wrong.
- **Fix:** normalise both fields to arrays during extraction.

### 1.3 Mostly empty fields (Verified)
- **The problem:** `latin` holds a real value in only 9 entries; the rest use the placeholder `—`. `etymology` is filled in for only 15.
- **Impact:** game modes or UI built around these fields would have almost no content.
- **Fix:** treat both fields as optional decoration.

### 1.4 Overlapping categories and types (Verified)
- **The problem:** three entries are tagged both fallacy and bias, and each also sits in two categories: base-rate fallacy, conjunction fallacy and gambler's fallacy.
- **Impact:** "Category sort" needs a rule for these entries, otherwise a correct answer could be marked wrong.

### 1.5 Undocumented field: `alreadyTaught` (Inference)
- **What the data suggests:** the 15 entries flagged `true` appear to be ones the file's author has already covered in some course or context. The field is not documented anywhere in the file.
- **Open question:** whether these should be treated as known (skipped or used for warm-ups) or taught again is a design decision for the file's author.

### 1.6 Citations not checked (Unverified)
- **The problem:** 354 of the 384 `source` fields cite works in prose with no URL. The other 30 point mainly to plato.stanford.edu, en.wikipedia.org and fallacyfiles.org.
- **What I have not done:** check whether any citation is accurate. This includes the specific journal references in the `mechanism` texts, such as Patt & Zeckhauser (2000) and Bar-Eli et al. (2007).
- **Why it matters:** the content may have been compiled with AI help. If so, fabricated or garbled citations are a known risk.
- **Fix:** before public release, spot-check at least the `core` tier (107 entries).

### 1.7 Fringe and popularised concepts (Verified count; Inference on risk)
- **The data:** 12 entries are marked `fringe` and 40 `popularization`. The fringe ones include "Goomba fallacy", "Motonormativity", "G.I. Joe fallacy" and "Traitorous critic fallacy".
- **The risk:** a quiz that rewards correct naming makes every term look equally legitimate.
- **Mitigation:** always show the evidence status, and consider excluding `fringe` entries from scored modes.

## 2. Scope concerns

### 2.1 "Argumentation" vs. the dataset (Inference)
- **The mismatch:** 241 of 384 entries (about 63%) are cognitive biases. Biases are about judgement, memory and decision-making, not the structure of arguments. Examples include "Recall & memory" (61 entries) and "Opinion reporting" (24).
- **The risk:** a site billed as teaching argumentation that is mostly about memory biases has a mismatch between its stated purpose and its content.
- **Status:** open scope decision, ROADMAP.md §3.1.

### 2.2 Missing argumentation basics (Inference)
- **The gap:** the dataset catalogues *errors* only. It doesn't teach the positive concepts needed to recognise them, such as premises and conclusions, validity vs. soundness, burden of proof, steelmanning or argument mapping.
- **The risk:** learning a list of named fallacies without these foundations can lead to "fallacy-spotting" as a debating tactic (sometimes called the fallacy fallacy, or *argumentum ad logicam*).
- **Fix:** a short foundations module is probably needed.

### 2.3 Too many items for one game (Inference)
- **The problem:** 384 items is a lot for one game. Without a clear progression path (core first, category by category), learners may be overwhelmed.

## 3. Game-design risks

### 3.1 Examples give away the answer (Verified for the obvious case; Inference for the wider problem)
- **The obvious case (verified):** in 13 entries, the example text contains the concept's own name, e.g. `ad-hominem`, `ad-populum`, `goomba-fallacy`.
- **The wider problem (inference):** many more examples likely contain a giveaway keyword from the definition.
- **Impact:** "Spot the fallacy" built on the raw examples would test keyword matching, not understanding.
- **Mitigation:** mask the name in the example, write fresh quiz-specific examples (a content effort), or use mechanism-based questions.

### 3.2 Distractor quality (Inference)
- **The problem:** random wrong answers are too easy, but answers drawn from the same category can be *arguably correct*. Many fallacies overlap; ad hominem variants are one example.
- **The risk:** players get marked wrong for defensible answers, which erodes trust in the game.
- **Mitigation:** blacklist each entry's `related` items as distractors, and allow players to report a disputed question.

### 3.3 Gamification can backfire (Inference)
- **The risk:** points and streaks can push players to optimise for score (guessing fast, memorising surface cues) rather than understanding.
- **Evidence:** research on extrinsic rewards and motivation is mixed, and I have not reviewed it for this file. Treat this as a design risk to test, not a finding.

### 3.4 Some concepts can't be tested by recognition alone (Inference)
- **The problem:** several biases describe statistical or population-level effects, such as base-rate neglect or regression effects. A single short vignette may not illustrate them well.

## 4. Legal and licensing (Unverified, needs a decision before public release)

- **Stated provenance:** the source file describes itself as "A personal reference … compiled from standard taxonomies (Wikipedia's List of Fallacies / List of Cognitive Biases)".
- **Why that matters:** Wikipedia text is licensed CC BY-SA. If any definitions or examples are copied or closely adapted from it, public reuse requires attribution and share-alike licensing of that content.
- **Not checked:** I have not checked how much of the text is derived from Wikipedia.
- **Other cited sources:** e.g. Stanford Encyclopedia of Philosophy and Fallacy Files have their own terms. Quoted passages from them would need review.
- **Repository licence:** none chosen yet.
- **Choices to make:** a licence for the code (e.g. MIT) and a licence for the content (e.g. CC BY-SA, if derived from Wikipedia).

## 5. Technical risks

### 5.1 Size of the source file (Verified size; Inference on impact)
- **The size:** the source HTML is about 850 KB, almost all of it data.
- **Impact:** loading all of it upfront is fine on desktop but noticeable on slow mobile connections.
- **Mitigation:** lazy-load the long fields (`mechanism`, `statusNote`) or split the data per category.

### 5.2 Progress lost with browser data (Verified as general browser behaviour)
- **The risk:** progress stored only in the browser (localStorage or IndexedDB) can be lost when the user clears site data, uses private browsing, or switches devices.
- **Mitigation:** tell users this in the interface, and offer progress export/import.

### 5.3 No single source of truth yet (Inference)
- **The risk:** if fixes are made to `data/entries.json` while the reference HTML keeps evolving separately, the two will drift apart.
- **Fix:** decide which file is canonical once Phase 0 lands.

## 6. Repository state
- **Branch:** the repository had no commits before these planning files were added.
- **Uploaded file not committed:** the source HTML has not been added to the repo yet; that happens in Phase 0. Until then the data exists only as the uploaded file.
