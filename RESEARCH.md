# FrenchoLinguo — Open resources research (2026-10-29)

Goal: a mobile app for learning French in short sessions (3–10 min). Unlike Duolingo, it explains the grammar.
Sister project of TurkoLinguo; the pedagogy section is shared with it.

Base language: **English** (decided 2026-10-29). Target variety: metropolitan French.

## 1. Pedagogy: the evidence supports explaining grammar
- Meta-analyses have repeatedly found explicit instruction (explaining the rule, then practising it) more effective than implicit exposure alone: Norris & Ortega 2000, Spada & Tomita 2010, Goo et al. 2015.
  Caveat: many of these studies measured explicit knowledge, so the app also needs plenty of meaningful practice.
- Session loop: **short rule card → guided examples → practice with feedback → spaced review**.
- Spaced repetition: **FSRS** via `ts-fsrs` (already used in TurkoLinguo).
- French-specific: for English speakers the hard parts are not word-building (as in Turkish) but **gender, agreement, verb forms, and the gap between spelling and sound** (silent endings, liaison, nasal vowels). Listening deserves more weight than in TurkoLinguo: *il parle / ils parlent* sound the same.

## 2. Grammar and curriculum sources
| Resource | Content | Licence | Use |
|---|---|---|---|
| ***Français interactif*** (UT Austin / COERLL) | Full first-year course (≈ A1–A2+): 13 chapters, grammar explanations in English, vocabulary lists, videos of native speakers, audio | CC BY 4.0 (Pressbooks and 2019 PDF editions; an older site page says CC BY 3.0) | Best open source found. Explanations and examples can be adapted with attribution |
| Wikibooks *French* | Grammar lessons, levels 1–4 | CC BY-SA | Can be adapted (share-alike) |
| CEFR descriptors (Council of Europe) | Can-do statements per level | Free to use | Defines what A1 and A2 mean |
| Beacco et al., *Niveau A1 / A2 pour le français* (Didier) | Official French reference levels: grammar and vocabulary inventories per level | © publisher | Reference only, to check our ordering and coverage; don't copy |
| DELF A1 / A2 exam specifications (France Éducation international) | What the exams test | © | Reference for checkpoints |

Copyrighted reference grammars to consult without copying: Grevisse, *Le Bon Usage*; Hawkins & Towell, *French Grammar and Usage*; *Bescherelle* (conjugation).

## 3. Language data
| Resource | Content | Licence |
|---|---|---|
| **Tatoeba** | About 2.2 million French sentences, a large share aligned with English | CC BY 2.0 FR (some CC0); audio licence varies per sentence |
| **Lexique 3.83 / 4** (lexique.org) | ≈ 140k forms: lemma, gender, number, part of speech, phonetic transcription, film-subtitle and book frequencies | CC BY-SA 4.0 (README of the Lexique 4 archive; check the 3.83 archive) |
| **Wiktionary via kaikki.org (wiktextract)** | French dictionary in JSONL with English glosses, inflections, IPA | CC BY-SA / GFDL |
| **Verbiste** | ≈ 7,000 verbs with conjugation templates (XML) | GPL-2.0+; converted data must stay GPL → **use only to test our engine, never ship** |
| Mozilla Common Voice French | One of the largest Common Voice languages (> 1,000 h) | CC0, but re-hosting is forbidden by the terms of use |

Frequency choice: Lexique's **subtitle frequency** (`freqfilms2`) is the best proxy for spoken everyday French, so it orders words inside each theme pack.

## 4. NLP tools (generating and checking forms)
French needs far less morphology than Turkish, so we write our own small TypeScript engine (conjugation templates, agreement, elision, contractions) and test it against Verbiste / Wiktionary tables at build time.
- **Lefff** (Sagot): large morphological lexicon, LGPL-LR. A second source for testing.
- **spaCy `fr_core_news_*`** (MIT): lemmatising Tatoeba sentences, so the pipeline can keep only sentences that use known words and grammar.
- Phonetics: Lexique's phonetic column, or Wiktionary IPA, for "silent letter" and liaison hints.

## 5. Audio / TTS
- **Lingua Libre** (Wikimedia Commons): French is one of its biggest languages (the project has 1.4M+ recordings in total). Coverage of our A1 theme list still needs counting (`Category:Lingua_Libre_pronunciation-fra`), as was done for Turkish.
- **Tatoeba audio:** many more French sentences have audio than Turkish ones; licence per sentence.
- **Piper** (MIT engine): French voices such as `fr_FR-siwis` (trained on the SIWIS database, CC BY 4.0) and `fr_FR-upmc`. Each voice has its own licence; check before commercial use.
- Placeholder: phone TTS through `expo-speech` with `fr-FR`.

## 6. Open points
- App licence / business model: as for TurkoLinguo. CC BY-SA content (Wikibooks, Wiktionary, Lexique) means releasing derived **content** under the same licence; *Français interactif* and Tatoeba (CC BY) only need attribution. Nothing GPL (Verbiste) is shipped.
- Confirm the licences of Lexique 3.83, the Piper FR voices and Lefff.
- Count the Lingua Libre coverage of the A1 word list once VOCABULARY.md exists.
- ~~Learner's base language~~ → **English** (decided 2026-10-29).

## Sources
- Lexique licence: [Lexique 4 README](https://lexique.org/databases/Lexique400/README-Lexique.txt), [pylexique docs](https://pylexique.readthedocs.io/en/latest/readme.html)
- *Français interactif*: [UT Austin Pressbooks](https://utexas.pressbooks.pub/francais-interactif), [Open Textbook Library](https://open.umn.edu/opentextbooks/textbooks/francais-interactif), [adoption page](https://www.laits.utexas.edu:443/fi/page/adopt)
- Tatoeba: [Wikipedia](https://en.wikipedia.org/wiki/Tatoeba)
- Lingua Libre: [Meta-Wiki](https://meta.wikimedia.org/wiki/Lingua_Libre)
- Piper Siwis voice: [Hugging Face listing](https://huggingface.co/Trelis/piper-fr-fr-siwis-medium)
- Verbiste: [Debian README](https://sources.debian.org/src/verbiste/0.1.49-1/README), [Mageia package](https://madb.mageia.org/package/show/application/0/name/verbiste)
