# FrenchoLinguo — Curriculum outline (A1, draft v0.1)

Base language: English. Target: CEFR A1, with a sketch of A2 at the end. Variety: metropolitan French.
Sources: CEFR descriptors and *Niveau A1 pour le français* (reference only) for coverage; *Français interactif* (CC BY 4.0) for explanations to adapt; Tatoeba fra–eng for example sentences; Lexique for frequencies. See [RESEARCH.md](RESEARCH.md).

---

## 1. Design principles

1. **One grammar point per lesson.** Each rule card can be read in under 60 seconds.
2. **Explain, then practise.** Every lesson follows the same order: rule card → 3–5 worked examples → practice with *explained* feedback.
3. **Gender is part of the word.** A noun is never shown without an article or a gender badge: *la table*, *l'arbre (m.)*. Learners memorise *le livre*, not *livre*.
4. **Spelling and sound, side by side.** French writes letters it doesn't pronounce. Every card can show silent letters in grey and liaisons with ‿ (*les‿amis*, *ils parlent* with grey *-ent*). Listening items test what spelling hides (*il parle / ils parlent*, *le / les*).
5. **Verbs: stem + ending.** Verb forms are first shown split (`parl·ons`, `fin·iss·ez`) and then fused, so the learner sees the pattern behind each group.
6. **Small vocabulary loads.** About 8–10 new words per lesson, chosen so the grammar can be practised. Each unit's vocabulary is themed.
7. **Spiral review.** FSRS schedules vocabulary cards *and* grammar-pattern cards (e.g. "*de* after a negation", "*être* verbs in the passé composé"). Every unit ends with a checkpoint.

### Session sizing (3–10 min)
Same as TurkoLinguo:

| Session | Content | Length |
|---|---|---|
| **Learn** | Rule card + worked examples + 4–6 easy items | 3–4 min |
| **Practise** | 10–15 mixed items on the new point + previous points | 4–6 min |
| **Review** (daily) | Due cards chosen by FSRS | 3–10 min |

35 lessons × 2 core sessions + checkpoints ≈ **80 core sessions for A1**, plus daily reviews.

---

## 2. Notation used in the app (and in the content files)

| Mark | Meaning | Example |
|---|---|---|
| grey letters | written but not pronounced | *ils parl*ent (grey), *peti*t (grey) |
| ‿ | liaison (a silent final consonant is pronounced before a vowel) | *les‿enfants* [lez‿ɑ̃fɑ̃], *vous‿êtes* |
| · | stem/ending split | `parl·ez`, `fin·iss·ons`, `all·é·e` |
| **m.** / **f.** badge | gender, when the article doesn't show it | *l'eau (f.)*, *les livres (m.)* |
| *(e)* | agreement depends on the speaker or subject | *Je suis content(e).* |

In the content files, verb forms are generated from templates (`-er`, `-ir/-iss-`, `-re`, irregular tables), and the form engine uses the same rules for feedback: "*mangeons*, not *mangons*: before *o*, *g* keeps its soft sound with an *e*."

---

## 3. A1 lesson map

Format for each lesson: **Grammar point** · rule summary · examples · common pitfalls · exercise types · vocabulary theme.

### Unit 0 — Sounds (2 lessons)

**L1. Letters, accents & silent endings**
- The accents and what they change: *é* [e], *è / ê* [ɛ], *ç* = "s" before a/o/u, *à / ù* (distinguish words: *a / à*, *ou / où*). *h* is silent.
- Final consonants are usually silent (*petit, grand, les*), except often **c, r, f, l** ("**careful**"): *avec, bonjour, neuf, il*.
- Stress falls on the last pronounced syllable of a phrase.
- Exercises: hear → pick the spelling; "which letters are silent?"; minimal pairs (*le / les*, *dessus / dessous*).
- Vocabulary: greetings (*bonjour, bonsoir, salut, merci, oui, non, au revoir, s'il vous plaît*).

**L2. Nasal vowels, u / ou, liaison & elision**
- Nasal vowels *an/en*, *on*, *in/ain/un*: *enfant, bon, vin, pain*.
- *u* [y] vs *ou* [u]: *tu / tout*, *dessus / dessous*.
- Elision: *le, la, je, ne, de* lose their vowel before a vowel or mute *h*: *l'ami, j'habite*. Liaison: *les‿amis, vous‿avez*. (Both come back in every later lesson.)
- Exercises: nasal or not?; elide or not (*le / l'*); liaison yes/no by ear.
- Vocabulary: about 10 nouns used later (*l'ami(e), la maison, le livre, l'eau, le pain, le vin, l'enfant, la table, le chat, l'école*).

### Unit 1 — Who & what (7 lessons + checkpoint)

**L3. *C'est* + *un / une*: gender**
- *Qu'est-ce que c'est ? — C'est un livre. C'est une table.* Every noun is masculine or feminine.
- Gender hints, not rules: *-tion, -té, -ette* → usually f.; *-ment, -age* → usually m. (*la page* is an exception worth naming.)
- Pitfall: the gender is not about meaning (*la chaise*, *le bureau*). Learn the article with the word.
- Vocabulary: classroom and household objects.

**L4. *le / la / l' / les* and the plural**
- Definite articles; plural *-s* (silent!), so the article carries the number: *le livre / les livres* sound different only at the article.
- *-eau → -eaux*, *-al → -aux* as a preview, with *le journal / les journaux*.
- *C'est le livre de Paul.* vs *Ce sont les livres de Paul.*
- Exercises: build the plural; listen: one or many?

**L5. Subject pronouns + *être***
- *je suis, tu es, il/elle/on est, nous sommes, vous êtes, ils/elles sont*.
- *tu* (friend, family, child) vs *vous* (polite or plural). *on* = "we" in speech.
- Adjectives of state and professions: *Je suis fatigué(e). Elle est médecin.* (no article before a bare profession).
- Fixed phrases: *Je m'appelle… / Tu t'appelles comment ?* (reflexives explained in L25); *Je suis anglais(e) / américain(e)*.
- Pitfall: *Il est professeur* (no *un*), but *C'est un professeur.*
- Vocabulary: nationalities, professions, states (*content, fatigué, malade*).

**L6. Negation: *ne… pas***
- *ne* before the verb, *pas* after it: *Je ne suis pas fatigué.* Before a vowel: *Il n'est pas là.*
- Spoken French often drops *ne* (*Je suis pas fatigué*): shown as a spoken variant, but written answers need *ne*.
- Exercises: affirmative ↔ negative; place *ne* and *pas*.

**L7. Questions**
- Three ways: intonation (*Tu es français ?*), *est-ce que* (*Est-ce que tu es français ?*), inversion (*Êtes-vous français ?*, recognised only in A1).
- Question words: *qui, où, comment, qu'est-ce que*. *Comment ça va ? — Ça va bien.*
- Pitfall: *est-ce que* doesn't change the word order of the sentence that follows; before a vowel *est-ce qu'il*.
- Exercises: turn statements into questions (each form accepted); match questions and answers.

**L8. *il y a / il n'y a pas de***
- *Il y a un chat dans le jardin. Il y a des livres.* (*des* = plural "some").
- After a negation, *un / une / des* become ***de*** (*d'*): *Il n'y a pas de pain. Il n'y a pas d'eau.*
- Vocabulary: café and food (*le café, le thé, l'eau, le pain, le fromage, le croissant*).

**L9. Adjectives & numbers 0–20**
- Agreement: *+e* (f.), *+s* (pl.): *petit, petite, petits, petites*. Sound change only in some forms (*petit / petite*, but *joli / jolie* sound the same).
- Position: most adjectives come after the noun (*une voiture rouge*); a short list goes before (*BAGS*: beauty, age, goodness, size → *une belle maison, un grand livre, un petit chat*).
- Numbers 0–20, with liaison (*deux‿amis*, *six / dix* before a consonant).
- Exercises: agree the adjective; word ordering; listen to a number and type it.

**✓ Checkpoint 1.** Mixed review + a mini dialogue (meeting someone in a café).

### Unit 2 — Having & doing (4 lessons + checkpoint)

**L10. *avoir***
- *j'ai, tu as, il a, nous avons, vous avez, ils ont* (liaison: *ils‿ont* vs *ils sont*!).
- Where English uses "to be": *J'ai vingt ans. J'ai faim / soif / froid / chaud.*
- Negation: *Je n'ai pas de voiture.* (recall of L8)
- Vocabulary: family preview, things you own.

**L11. -er verbs in the present**
- Stem + *-e, -es, -e, -ons, -ez, -ent*. Four of the six forms sound the same: *parle, parles, parle, parlent*.
- Elision with *je*: *j'aime, j'habite*.
- Spoken *on* = *nous*: *On parle français.*
- Exercises: conjugate (split → fused); listen: which person?; dictation of silent endings.
- Vocabulary: everyday verbs (*parler, habiter, aimer, travailler, regarder, écouter, manger, chercher*).

**L12. Partitive *du / de la / de l' / des***
- "Some" of something uncountable: *Je mange du pain. Tu bois de l'eau ?* (*boire* as a phrase-form only).
- *aimer / adorer / détester* take *le / la / les* (general taste): *J'aime le fromage*, but *Je mange du fromage.*
- After a negation → *de*: *Je ne mange pas de viande.*
- Pitfall: the hardest A1 contrast for English speakers. Contrast-pair items ("which one means *some* cheese?").
- Vocabulary: food & drink.

**L13. -er spelling changes; *on***
- *manger → nous mangeons*, *commencer → nous commençons* (keep the soft sound before *o*).
- *acheter → j'achète*, *préférer → je préfère*, *appeler → j'appelle* (the stem changes when the ending is silent: "boot" verbs).
- Exercises: spot the error; conjugate with a "why?" hint.

**✓ Checkpoint 2.** Food shopping at the market (*Je voudrais du fromage, s'il vous plaît.*).

### Unit 3 — Where & when (5 lessons + checkpoint)

**L14. *aller* + *à*: *au, à la, à l', aux***
- *je vais, tu vas, il va, nous allons, vous allez, ils vont*.
- *à + le → au*, *à + les → aux*: *Je vais au cinéma, à la plage, à l'école, aux toilettes.*
- *Où ?* "where?"; *Comment allez-vous ?* (recall of L7).
- Vocabulary: places in town.

**L15. Prepositions of place; *chez***
- *dans, sur, sous, devant, derrière, entre*; *à côté de / près de / loin de* with *de + le → du*: *à côté du café*.
- *chez* + person: *chez moi, chez Marie, chez le médecin*.
- Exercises: picture → sentence (*Le chat est sous la table.*).

**L16. Countries & cities**
- *à* + city (*à Paris*); *en* + feminine country or one starting with a vowel (*en France, en Iran*); *au* + masculine (*au Canada*); *aux* + plural (*aux États-Unis*).
- Nationalities with agreement: *il est français / elle est française*.
- Vocabulary: countries, nationalities, languages (*le français, l'anglais*).

**L17. Time & days; numbers 21–69**
- *Quelle heure est-il ? — Il est trois heures et demie / moins le quart.* 24-hour time in schedules (*à 18 h 30*).
- Days and months, no capitals: *lundi, le lundi* (every Monday) vs *lundi* (this Monday).
- Numbers 21–69 (*vingt et un*, *trente-deux*).

**L18. *venir* + *de*; numbers 70–100**
- *je viens, tu viens, il vient, nous venons, vous venez, ils viennent*. *Je viens du Canada, de Paris, des États-Unis.*
- *de + le → du*, *de + les → des* (the twin of L14).
- Numbers 70–100: *soixante-dix, quatre-vingts, quatre-vingt-dix* (Belgian/Swiss *septante, nonante* as a note).

**✓ Checkpoint 3.** Asking the way and arranging a meeting time.

### Unit 4 — Mine & yours (6 lessons + checkpoint)

**L19. Possessives *mon / ma / mes, ton…, son…***
- The possessive agrees with the **thing owned**, not the owner: *son frère* = his **or** her brother; *sa sœur* = his **or** her sister.
- Before a vowel, *ma → mon*: *mon amie*.
- Vocabulary: family.

**L20. *notre / votre / leur***
- *notre maison, nos amis, leur voiture, leurs enfants*.
- Pitfall: *leur* (one thing) vs *leurs* (several); *votre* also polite.

**L21. Demonstratives *ce / cet / cette / ces***
- *ce livre, cet homme, cet ami* (vowel or mute *h*), *cette femme, ces enfants*. *-ci / -là* for "this / that one" (recognition).
- Vocabulary: clothes, colours (with agreement from L9).

**L22. -ir (*finir*) and -re (*attendre*) verbs**
- *je finis… nous finissons, vous finissez, ils finissent* (the *-iss-* in the plural).
- *j'attends, il attend, nous attendons*: no ending in *il attend*.
- Exercises: which group?; conjugate with split view.

**L23. *faire* and *prendre***
- *faire*: weather (*il fait beau / froid*), activities (*faire du sport, de la musique*: partitive recall), *faire les courses*.
- *prendre / apprendre / comprendre*: *je prends, nous prenons, ils prennent*. *Je prends un café.*
- Vocabulary: weather, sports and hobbies.

**L24. *vouloir, pouvoir, devoir* + infinitive**
- *je veux / peux / dois* + infinitive: *Je peux venir ? Tu dois travailler.*
- *Je voudrais* (polite "I would like") stays a set phrase until A2.
- Negation goes around the conjugated verb: *Je ne peux pas venir.*

**✓ Checkpoint 4.** Introducing your family and making plans for the weekend.

### Unit 5 — Daily life (5 lessons + checkpoint)

**L25. Reflexive verbs**
- *je me lève, tu te lèves, il se lève, nous nous levons, vous vous levez, ils se lèvent*. Elision: *je m'appelle, il s'habille*.
- *Je m'appelle* from L5 is now explained.
- Negation: *Je ne me lève pas tôt.*
- Vocabulary: daily routine, times of day.

**L26. The imperative**
- *Parle ! Parlons ! Parlez !* (-er verbs drop the *-s* in the *tu* form). *Sois / soyez, aie / ayez* as phrases.
- Negative: *Ne parle pas !* Polite alternatives: *Vous pouvez… ?*
- Vocabulary: classroom and kitchen instructions.

**L27. *futur proche***
- *aller* + infinitive: *Je vais manger. Nous allons partir demain.*
- Negation: *Je ne vais pas sortir.* Time words: *demain, ce soir, la semaine prochaine*.

**L28. Direct object pronouns *le / la / l' / les***
- Before the verb: *Je le vois. Tu la connais ? Je les aime.* With an infinitive: *Je vais le faire.*
- Pitfall: same forms as the articles but a different job; the position is the hard part.
- Exercises: replace the object; place the pronoun (word bank).

**L29. *quel*, *combien*, *pourquoi / parce que***
- *Quel / quelle / quels / quelles* agree with the noun: *Quelle heure ? Quels livres ?*
- *Combien de* + noun (*combien de frères ?*); *Combien ça coûte ?*
- *Pourquoi ? — Parce que…* (*parce qu'il*).

**✓ Checkpoint 5.** Describing your day and asking someone about theirs.

### Unit 6 — Yesterday (4 lessons + checkpoint)

**L30. Passé composé with *avoir***
- *avoir* (present) + past participle: *J'ai parlé. Tu as fini ? Il a attendu.* (-er → *-é*, -ir → *-i*, -re → *-u*).
- One past tense where English has two: *J'ai mangé* = "I ate" / "I have eaten".
- Time words: *hier, ce matin, la semaine dernière, il y a deux jours*.

**L31. Irregular past participles**
- *fait, pris, mis, eu, été, vu, bu, lu, voulu, pu, dû, dit, écrit*.
- Exercises: participle recall; spot the error (~~j'ai prendu~~).

**L32. Passé composé with *être***
- The movement verbs (*aller, venir, partir, arriver, entrer, sortir, monter, descendre, naître, mourir, rester, tomber, retourner, devenir*) and all reflexive verbs.
- The participle agrees with the subject: *Elle est allée. Ils sont partis. Nous nous sommes levé(e)s.*
- Pitfall: *je suis allé(e)* is not "I am gone".

**L33. Negation & questions in the past**
- *ne* and *pas* go around the auxiliary: *Je n'ai pas mangé. Elle n'est pas venue.*
- *Qu'est-ce que tu as fait hier ? Tu es allé(e) où ?*

**✓ Checkpoint 6.** Telling a friend about last weekend.

### Unit 7 — Plans & wishes (2 lessons + final A1 checkpoint)

**L34. *venir de* + infinitive; *être en train de***
- Recent past: *Je viens de manger.* "I've just eaten." Ongoing: *Je suis en train de travailler.*
- Contrast with L27 (*aller* + infinitive) and L30.

**L35. *savoir* vs *connaître***
- *savoir* + fact / infinitive (*Je sais nager. Tu sais où il habite ?*); *connaître* + person / place (*Je connais Paris.*).
- *pouvoir* vs *savoir* for abilities (*Je ne peux pas nager aujourd'hui* vs *Je ne sais pas nager*).

**✓ Checkpoint 7 (final A1).** A longer scenario: booking a weekend away (where, when, what you did last time, what you'll do).

---

## 4. A2 (sketch, to be written in CURRICULUM_A2.md)

Imparfait, then imparfait vs passé composé · *y* and *en* · indirect object pronouns *lui / leur* and pronoun order · comparatives and superlatives (*meilleur, mieux*) · futur simple · conditionnel (polite and hypothetical *si* + imparfait) · relative pronouns *qui / que / où* · *depuis / pendant / il y a* · adverbs in *-ment* · disjunctive pronouns (*moi, toi…*) beyond set phrases · negatives *ne… jamais / plus / rien / personne* · subjunctive after *il faut que* (as an introduction only).

---

## 5. Exercise type catalogue

| Type | Tests | Feedback |
|---|---|---|
| Pick the article (*le / la / l' / les / un / du…*) | Gender, number, partitive, elision | Names the deciding fact (gender, vowel, negation) |
| Agree the adjective / participle | Agreement | Highlights the noun or subject it agrees with |
| Conjugate (split → fused) | Verb forms | Diff + the rule (group, spelling change, irregular) |
| Type the form | Production | Diff + the rule that was broken |
| Translate EN → FR (word bank, later free typing) | Sentence production | Accepts the variants listed in the content |
| Translate FR → EN | Comprehension | — |
| Listen & choose / dictation | Listening; silent letters | Slowed-down replay; silent letters shown in grey |
| Singular or plural? (by ear) | *le / les*, *il / ils*, liaison | Points to the sound that decides |
| Spot the error | Rule awareness | Explains the error |
| Contrast pair (*du fromage / le fromage*, *il a / il est allé*) | Meaning of a form | Short semantic note |
| Mini dialogue (fill in the turns) | Use in context | — |

Every item is tagged with the **grammar points** it uses, so FSRS can schedule grammar-pattern reviews, and so the Tatoeba pipeline can find example sentences restricted to points already learned.

### Typing & answer checking (French-specific requirements)
- **Accent bar:** é è ê à â ç î ï ô û ù ë œ above the keyboard.
- **Accent near-misses:** an answer that is right except for an accent is marked "almost right" and the accent is named ("**é**, not e"). If the accent changes the word (*a / à, ou / où, du / dû, sur / sûr, la / là*), it is marked wrong and the two words are explained.
- **Normalise harmless differences:** ' and ’ apostrophes, *oe / œ*, capital letters, and French spaces before `? ! : ;` are all accepted.
- **Accept valid variants:** the three question forms (L7), *on / nous*, *tu / vous* when the prompt doesn't decide, and both genders when the speaker is unknown (*content / contente*). The spoken *ne*-drop is accepted only in items marked as spoken.

### Set phrases
Forms that come later appear early only as **set phrases** (*s'il vous plaît, je voudrais, je m'appelle, ça va, excusez-moi, je ne sais pas, de rien, à bientôt, enchanté(e)*). They:
- carry a **"phrase" badge** and are not split into stem and ending;
- are never used in "conjugate" or "build the form" exercises;
- are linked from the lesson that explains them (*je m'appelle* → L25, *je voudrais* → A2 conditionnel).

---

## 6. Open questions (to settle with a French-expert review)
- **Partitive placement:** L12 teaches the partitive right after -er verbs, contrasted with *aimer le / la*. Check that this is not too much for Unit 2.
- **Inversion:** recognition only in A1, production in A2?
- **Spoken French:** how much (*ne*-drop, *on*, *ouais*, *t'es*) to show beyond spoken-variant notes?
- **Object pronouns in A1** (L28): keep, or move to A2 with *y / en* and *lui / leur*?
