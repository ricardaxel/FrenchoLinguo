# FrenchoLinguo — Project plan (draft v0.1, 2026-10-29)

Goal: a mobile app for learning French in short sessions (3–10 min). Unlike Duolingo, it **explains the grammar**.
It follows the same approach as [TurkoLinguo](../TurkoLinguo): rule card → worked examples → practice with
*explained* feedback → FSRS spaced review, plus vocabulary theme packs and optional talk tasks with a native speaker.

Base language: **English** (decided 2026-10-29). Target variety: metropolitan (France) French.

---

## 1. What we reuse from TurkoLinguo

TurkoLinguo is the template. Most of it is language-independent and can be copied, then renamed:

| Part | Reuse | Notes |
|---|---|---|
| App shell (Expo SDK 57, RN 0.86, TS, Expo Router) | Upgrade to SDK 58 | Screens: home, session, grammar, checkpoint, talk. Target device: **iPhone** (see [docs/IOS.md](docs/IOS.md)) |
| `session/` (SessionRunner, CardView, ItemView, grade.ts) | As is | Exercise types are generic |
| `progress/`, `review/` (ts-fsrs) | As is | Vocabulary + grammar-pattern cards |
| `talk/` + docs/TALK_TASKS.md | As is | Partner card frame rewritten in French |
| Content pipeline (YAML → validator → content.json) | Adapt | Schema keeps its shape; morphology part changes |
| `scripts/validate_content.py` | Adapt | NFC check, "used before taught" check, French typography check |
| CI + signed APK release workflow | As is, + EAS iOS workflow | Rename package id; add `release-ios.yml` (EAS Build → TestFlight); Android stays a secondary target |
| `engine/` (turkish.ts, morph.ts, check.ts, feedback.ts) | **Rewrite** | French needs a different engine (§3) |
| Curriculum, vocabulary, sentences, media | **New** | French content written from scratch |

## 2. Research (RESEARCH.md)

Same survey as TurkoLinguo, for French:
- **Grammar / curriculum:** CEFR A1–A2 descriptors for French (Référentiel / *Niveau A1 pour le français*), Wikibooks *French*, open FLE courses (e.g. *Français interactif*, UT Austin / COERLL, CC BY).
- **Sentences:** Tatoeba `fra`–`eng` (very large set, many with audio; CC BY 2.0 FR).
- **Dictionary / frequency:** Wiktionary via kaikki.org; Lexique 3/4 (frequencies, gender, lemma ↔ form; CC BY-SA).
- **Conjugation data:** Verbiste / Wiktionary conjugation tables (check licences) — used to *test* our engine, not shipped.
- **Audio:** Lingua Libre (`fra`, very large), Tatoeba audio, Common Voice FR, Piper FR voices (per-voice licence); `expo-speech` `fr-FR` as placeholder.
- Decide the content licence given CC BY-SA sources.

## 3. French engine (replaces the Turkish suffix engine)

Turkish feedback is about vowel harmony and suffixes; French feedback is about **conjugation, agreement and sound/spelling rules**.

1. **Conjugation:** regular -er / -ir (finir) / -re groups, spelling changes (*mangeons, commençons, appelle, achète*), the core irregulars (*être, avoir, aller, faire, pouvoir, vouloir, venir, prendre…*). Tenses for A1–A2: présent, impératif, passé composé (avoir/être + participle agreement), futur proche, imparfait, futur simple, conditionnel de politesse.
2. **Nouns & agreement:** gender stored on every lexicon entry; plural (-s, -x, -aux); adjective agreement and position (*un grand homme / une voiture rouge*), *beau/bel/belle* type forms.
3. **Determiners & contractions:** *le/la/l'/les*, *un/une/des*, partitive *du/de la/de l'*, *de* after negation, *au/aux/du/des*.
4. **Elision & typography:** *je → j'*, *le → l'*, *ne → n'*, *que → qu'*, *si → s'il*; French spacing before `? ! : ;` (accepted, not required, in typed answers).
5. **Answer checking:** accent-tolerant mode with explained feedback ("*é*, not *e*") rather than a hard fail; accepted variants (*est-ce que* / inversion / intonation questions; *on* / *nous*).
6. **Feedback diagnosis** names the rule: "*allée*, not *allé*: with *être*, the participle agrees with *elle*."
7. **Spoken French:** dropped *ne* (*je sais pas*), liaison and *e muet* shown on cards, like TurkoLinguo's spoken variants.

Shared expected-forms test file (`content/morphology/tests.yaml`) as in TurkoLinguo.

## 4. Curriculum (CURRICULUM.md, then CURRICULUM_A2.md)

Same design principles: one grammar point per lesson, rule card readable in < 60 s, Learn (3–4 min) + Practise (4–6 min) sessions, a checkpoint per unit. Draft A1 map (~35 lessons):

- **Unit 0 — Sounds:** French sounds for English speakers (nasal vowels, *u/ou*, *r*, silent final letters); accents and what they do; liaison and elision.
- **Unit 1 — Who & what:** *c'est / ce sont*, *un/une*, gender; *le/la/les*; subject pronouns + *être*; negation *ne… pas*; questions (intonation, *est-ce que*, question words); *il y a*; numbers 0–20; adjective agreement & position.
- **Unit 2 — Having & doing:** *avoir* (+ age, *avoir faim*…); -er verbs présent; *de* after negation; partitive (food); *aller* + *à* contractions; *faire* + weather.
- **Unit 3 — Where & when:** prepositions of place, countries & cities (*en / au / à*); time, days, dates; numbers to 100; *venir de* + contractions.
- **Unit 4 — Mine & yours:** possessives *mon/ma/mes*; demonstratives *ce/cet/cette/ces*; -ir and -re verbs; *pouvoir / vouloir / devoir* + infinitive.
- **Unit 5 — Daily life:** reflexive verbs (routine); imperative; *futur proche*; object pronouns *le/la/les* (intro).
- **Unit 6 — Past:** passé composé with *avoir*; with *être* (+ agreement); negation in the passé composé; time expressions.
- **A2 sketch:** imparfait vs passé composé, *y / en*, indirect pronouns, comparatives/superlatives, futur simple, relative *qui/que/où*, conditionnel de politesse, *depuis / il y a*.

## 5. Vocabulary (VOCABULARY.md)

Same model: 8-word packs placed just before the lesson that uses them, each word with a sentence frame using known grammar, core vs elective themes, frequency-first (Lexique). Every noun card shows **gender with its article** (*la table*, never *table* alone). A1 target ≈ 650 words.

## 6. Milestones

1. **M0 — Setup (now):** git repo, this plan.
2. **M1 — Research & design docs:** RESEARCH.md, CURRICULUM.md (A1), VOCABULARY.md, CONTENT_FORMAT.md adapted.
3. **M2 — Skeleton:** copy the TurkoLinguo app, pipeline and CI; rename (package id, title, `fr-FR` TTS); strip Turkish content and engine; **upgrade to SDK 58** (the App Store Expo Go runs only SDK 58); build passes with an empty course and runs on the **iPhone via Expo Go**. Steps M2.1–M2.6 in [docs/IOS.md §7](docs/IOS.md).
4. **M3 — French engine:** conjugation, agreement, elision, answer checking + feedback, with tests.
5. **M4 — Vertical slice:** one full lesson (e.g. *être* + subject pronouns) with grammar, Learn/Practise items, a pack, review cards, a talk task; playable on the phone through Expo Go.
6. **M5 — A1 content:** units 0–6 + checkpoints + packs, written in parallel per unit, each reviewed by a French-expert pass (as in TurkoLinguo).
7. **M6 — Audio & polish:** Lingua Libre / Tatoeba audio coverage, **TestFlight build (iOS)** + signed APK (Android). The Apple Developer Program ($99/yr) starts here, not earlier.
8. **M7 — A2.**

## 7. Open decisions

- ~~Learner's base language~~ → **English** (decided 2026-10-29).
- **Variety of French:** metropolitan France (default, used for now); show Québec/Belgian/Swiss variants as notes?
- **Shared code:** copy the TurkoLinguo app (simplest, chosen for now) vs. extracting a shared core both apps use.
- **Licence / business model:** same question as TurkoLinguo (CC BY-SA content means share-alike content).
- **iPhone** (details in [docs/IOS.md §8](docs/IOS.md)):
  - when to buy the Apple Developer Program (recommended: M5/M6);
  - the home-screen name ("FrenchoLinguo" may be cut off; "Frencho"?);
  - iPad support (recommended: off);
  - the accent bar layout;
  - beta for other learners vs straight to the App Store;
  - where to host the privacy policy.
