---
name: blog-technique-charles
description: >
  Transforme une matière technique brute (message, notes, brouillon, fil de revue de code) en article de blog technique qu'on a envie de cliquer, lire jusqu'au bout, retenir et partager, sans sacrifier la justesse. Skill de canal « blog » posé sur style-technique-charles : il choisit l'angle, le lecteur, la question ouverte et la preuve maîtresse, délègue l'écriture au skill de style, puis travaille titre, intro, intertitres, rétention, conclusion, CTA, packaging (description, slug) et messages de partage. Déclencheurs : article de blog, billet, rendre un article bankable, marketing éditorial, titre d'article, accroche, packaging, partager mon article, post LinkedIn / Hacker News / Reddit pour un article, pourquoi personne ne lit mon article. Français uniquement.
---

# Article de blog technique (Charles)

Faire un article dont la valeur technique fait le marketing de son auteur. Le marketing reste invisible : pas d'accroche fabriquée, pas de promesse non tenue, pas de hot take.

```
impression → clic → lecture → « ah, c'est pour ça » → confiance → souvenir → partage
   titre      intro   progression      preuve          compromis    idée       envoi
```

Un bon contenu peut échouer à chaque étape. Ce skill cherche **l'étape qui bloque**, pas à tout optimiser.

## Quand l'utiliser

| Utiliser | Ne pas utiliser |
|---|---|
| Matière brute à transformer en article de blog | Réécrire la voix ou le raisonnement : `style-technique-charles` |
| Article déjà rédigé à rendre publiable (titre, intro, rétention, packaging) | Documentation, changelog, README |
| Messages de partage d'un article publié | Texte en anglais |
| « Pourquoi cet article ne marche pas ? » | Matière sans instance réelle : demander le cas, ne rien inventer |

## Contrat avec `style-technique-charles`

Charger `style-technique-charles` (et donc `write-without-slop`) avant toute écriture. Ce skill respecte son « Contrat pour les skills de canal » : il peut changer longueur, découpage, intertitres, accroche, titre, présentation de l'auteur, densité des liens, CTA. Il ne touche **jamais** à la chaîne enjeu → grande idée, au statut épistémique et à l'affect, à la frontière, ni à l'interdiction d'inventer.

Arbitrages entre marketing et style :

| Tentation marketing | Arbitrage |
|---|---|
| Créer un *knowledge gap* dans l'intro | Oui, mais la question vient du déclencheur réel. Jamais de question rhétorique, de chiffre choc ni de citation d'autorité |
| *Open loops* entre sections | Le contenu crée le besoin (une limite, un contre-exemple, une objection). Jamais une formule de suspense (« Le plus étonnant ? ») |
| Intertitres informatifs | Un intertitre nomme l'étape ou l'état du cas, pas la thèse en titre de cours |
| Humour, digressions, voix forte | N'en ajouter aucun. Garder ceux de la matière |
| Phrase finale mémorable | L'idée transmissible vit dans le corps, là où la démonstration la justifie. La conclusion donne la règle et sa frontière |
| Signal d'expertise | Une phrase de contexte qui situe l'enjeu (« j'ai maintenu eMush pendant trois ans ») ; jamais de chiffres d'expertise (« 800 PR relues ») |
| Idée partageable plus tranchée ou plus consensuelle | Interdit. Elle garde la force exacte de la position de Charles |

---

## Arbre 1 - Quel mode ?

```
Qu'est-ce que Charles fournit ?
├─ Matière brute                → COMPLET : tri → fiche → validation → rédaction → passe blog → packaging
├─ Sujet seul                   → entretien de style-technique-charles + questions « Lecteur » et « Preuve » de la fiche
├─ Article rédigé               → PASSE BLOG : fiche déduite du texte, goulet, corrections ciblées, packaging
└─ Article publié               → PARTAGE : messages par canal
```

## Arbre 2 - La matière tient-elle un article ?

```
La matière contient-elle…
├─ une instance réelle (test, bug, diff, décision) ?   non → pas d'article : demander le cas
├─ une idée non évidente pour le lecteur visé ?        non → c'est une note ou un message, le dire
├─ une preuve observable (code, sortie, mesure) ?      non → demander, ou [TK] si c'est secondaire
├─ deux grandes idées ?                                → deux articles : garder celle qui a la meilleure preuve
└─ oui partout                                         → fiche éditoriale
```

## Fiche éditoriale

Remplir **avant** d'écrire, depuis la matière seulement. Case vide → question à Charles ou `[TK]`.

| Champ | Question | Exemple (article sur les mocks) |
|---|---|---|
| Lecteur | Qui a déjà vécu ce problème ? | Développeur qui maintient des tests unitaires pleins de mocks |
| Déclencheur | Qu'est-ce qui a réellement lancé la réflexion ? | Un contributeur ne comprend pas ce qu'un test vérifie |
| Question ouverte | Quelle question le lecteur possède-t-il assez pour vouloir la réponse ? | Que vérifie ce test, au juste ? |
| Grande idée | La réponse, en une phrase que le lecteur pourrait répéter à un collègue | Un test mocké vérifie ses mocks : il casse quand on améliore le code, alors qu'il devait le protéger |
| Preuve maîtresse | L'artefact qui fait dire « ah, c'est pour ça » | Le même refactoring : le test mocké échoue, le test par résultat passe |
| Signal | Quel compromis ou quelle frontière montre le jugement ? | Défauts de l'école de Chicago, *fakes* pour les dépendances volatiles |
| Titres | 3 à 5 candidats, recommandation en premier | Voir règle 1 |
| Longueur | Minimum pour tenir la promesse | ~1 250 mots |
| Goulet | L'étape du tunnel la plus faible | Intro qui annonce un cours au lieu de la question |

Au-delà de ~1 500 mots, ou si l'angle est discutable, faire valider la fiche avant de rédiger.

Exemple complet (fiche, diagnostic, packaging, partage) : [references/exemple-mocks.md](references/exemple-mocks.md).

---

## Règles

### Règle 1 - Le titre ouvre la question sans cacher le sujet

Spécifique, pertinent, avec une tension. Compréhensible sans connaître le projet. Il ne promet rien que l'article ne tient pas.

```
❌ « Tout comprendre aux mocks dans les tests unitaires »   → cours, aucune tension
❌ « Ce test va vous surprendre »                           → mystère sans information
❌ « Mon retour sur eMush »                                 → contexte privé, aucun sujet
✅ « Ce que vérifie un test rempli de mocks »                → sujet clair, question ouverte
```

### Règle 2 - L'intro pose la question en moins de ~200 mots

Dans l'ordre de `style-technique-charles` (règle 1a) : déclencheur, instance, question ouverte. À la fin de l'intro, le lecteur sait de quoi on parle, pourquoi c'est intéressant, pourquoi Charles en parle, et ce qui reste à comprendre.

- Couper les phrases méta qui n'apprennent rien (« il y a beaucoup à dire là-dessus »).
- Annoncer la question, pas le plan ni le cours (« je résume l'opposition entre deux écoles »).
- Si la preuve maîtresse arrive tard, l'intro laisse voir un fait de l'instance qui la rend désirable. Ne pas la dévoiler.

### Règle 3 - Un premier « ah » avant le premier tiers

Ne pas demander dix minutes d'attention avant le premier insight. Distribuer ensuite : distinction utile, sortie réelle, contre-exemple, formulation juste. Chaque section fait au moins une de ces choses : poser une question, apporter une preuve, produire un insight, faire avancer l'argument. Sinon, la couper.

### Règle 4 - Chaque section rend la suivante nécessaire

```
❌ « Maintenant que nous avons vu les mocks, intéressons-nous aux fakes. »
✅ « Le traducteur en mémoire n'est pas volatile. Pour une dépendance qui l'est, … »
```

La fin d'une section laisse une limite, une objection ou un cas non traité. La section suivante y répond.

### Règle 5 - La lecture en diagonale raconte déjà l'article

Parcours : titre → intro → intertitres → code et sorties → conclusion. Ce parcours seul doit montrer le problème, la progression, la preuve et la conclusion.

```
❌ « Une autre perspective »                                    → vide
❌ « Les tests mockistes sont couplés à l'implémentation »       → thèse en titre de cours
✅ « Modifier le code sans changer son comportement »            → étape du cas
✅ « Introduire un vrai bogue »                                  → étape du cas
```

Un intertitre marque un nouveau mouvement de l'argument, pas une pause visuelle.

### Règle 6 - Des preuves, pas des déclarations

- Code : les 5 à 20 lignes qui portent l'argument ; retirer imports, boilerplate et configuration sans rapport.
- Sorties d'erreur, diffs, mesures : réellement obtenues. Sinon `[TK: sortie réelle de …]`, jamais une sortie plausible.
- Un exemple fil rouge fort plutôt que cinq exemples. Un second exemple seulement s'il révèle une limite ou casse une lecture trop simple.
- Un visuel seulement s'il réduit l'effort de lecture (diff, trace, schéma). Aucune image décorative.
- La comparaison loyale (introduire aussi le bug qui donne raison à l'autre option) est le signal professionnel le plus fort. La mettre en valeur, pas la raccourcir.

### Règle 7 - La conclusion ferme la boucle

Retour au déclencheur, puis la règle et sa frontière (`style-technique-charles`, règle 4). Pas de résumé des sections, pas de slogan final.

CTA : discret, une phrase, qui découle de l'argument (identité, contact, article lié). Son contenu vient de Charles : sinon `[TK: CTA]`.

```
❌ « Vous aussi, transformez vos tests dès aujourd'hui ! »
✅ « [TK: phrase sur les problèmes sur lesquels Charles travaille + contact] »
```

---

## Packaging

| Élément | Contrainte |
|---|---|
| Titre | Une idée, compréhensible sans contexte, 80 caractères max (limite Hacker News) |
| Description | Une ou deux phrases, ~160 caractères : la promesse, sans répéter le titre |
| Slug | Court, stable, sans date ni mot vide superflu |
| Auteur | Bloc auteur et liens : `[TK]` si absent de la matière |

## Partage (sur demande ou mode PARTAGE)

Même article, enveloppe adaptée au canal. Le titre canonique ne change pas ; le message d'accompagnement peut mettre en avant le déclencheur, la preuve ou l'application pratique.

| Canal | Forme | À éviter |
|---|---|---|
| LinkedIn | Unité de valeur autonome : observation concrète → insight → lien | « Nouvel article ! N'hésitez pas à partager 🚀 » |
| Hacker News | Soumission normale (pas `Show HN`, réservé à ce qu'on peut essayer), titre original de l'article | Titre retouché pour attirer, demande de votes |
| Reddit | Seulement dans une communauté où le problème est déjà discuté, après lecture des règles ; contribution à une conversation, contre-exemples bienvenus | « Vos retours sont les bienvenus ! » |
| Discord, Slack | Seulement si le sujet vient d'y être discuté : « on en parlait l'autre jour » | Diffusion dans tous les salons |

Aucun message de partage n'ajoute un fait, un chiffre ou une position absents de l'article.

---

## Workflow

1. Choisir le mode (arbre 1).
2. Trier la matière (arbre 2).
3. Remplir la fiche éditoriale. Nommer le goulet.
4. Si l'article dépasse ~1 500 mots ou si l'angle est discutable, faire valider la fiche.
5. Rédiger avec `style-technique-charles`, en lui passant la fiche comme contraintes de canal.
6. Passe blog : règles 1 à 7, en commençant par le goulet. Corrections ciblées, dans la voix de Charles.
7. Packaging.
8. Messages de partage, si demandés.
9. Remettre : l'article, la fiche, le goulet et ce qui a été fait, 3 à 5 titres, les `[TK]`.

## Anti-patterns

| Anti-pattern | Correction |
|---|---|
| Tout optimiser à la fois | Nommer un goulet, le traiter d'abord |
| Intro qui annonce un cours ou un plan | Déclencheur → instance → question ouverte |
| Question rhétorique ou suspense pour tenir le lecteur | Le contenu crée la question (limite, contre-exemple) |
| Idée partageable qui adoucit ou durcit la position de Charles | Force exacte de la source |
| Sortie d'erreur ou métrique « plausible » | Sortie réelle, ou `[TK]` |
| Preuve maîtresse enterrée dans une section tardive sans être désirée | Un fait de l'instance, dans l'intro, la rend attendue |
| Intertitres décoratifs ou thèses de cours | Étapes du cas |
| Conclusion-résumé ou slogan | Règle et frontière |
| Article allongé pour « faire sérieux » | Longueur minimale pour tenir la promesse ; au-delà de ~3 000 mots, chercher deux articles |
| Même message copié sur tous les canaux | Enveloppe par canal, article identique |

## Checklist finale

- [ ] La fiche éditoriale est remplie depuis la matière ; les manques sont en `[TK]`.
- [ ] Le titre ouvre une question sur un sujet explicite, sans contexte privé.
- [ ] L'intro pose déclencheur, instance et question en moins de ~200 mots, sans phrase méta.
- [ ] Un premier insight arrive avant le premier tiers.
- [ ] Titre, intro, intertitres, code et conclusion racontent seuls l'article.
- [ ] Chaque section pose une question, apporte une preuve ou fait avancer l'argument.
- [ ] Toutes les sorties et mesures sont réelles.
- [ ] La grande idée apparaît dans le corps, à la force exacte de la position de Charles.
- [ ] La conclusion donne la règle et sa frontière ; le CTA est discret ou en `[TK]`.
- [ ] La checklist de `style-technique-charles` passe toujours.

Questions de relecture :

- Qu'est-ce que j'aurais aimé lire quand j'étais bloqué sur ce problème ? (Julia Evans)
- Quelle affirmation résiste mal si je cherche un contre-exemple ? (Dan Luu)
- Qu'est-ce qui rend ce problème intéressant dans un vrai système, pas dans un exercice ? (Stripe Engineering)
- Quelqu'un qui connaît déjà le concept aurait-il plaisir à lire ce passage ?

## Fichiers de référence

| Fichier | Contenu |
|---|---|
| [references/exemple-mocks.md](references/exemple-mocks.md) | Passe blog complète sur l'article « Ce que vérifie un test rempli de mocks » |

## Sources

- **[The psychology of curiosity: A review and reinterpretation](https://doi.org/10.1037/0033-2909.116.1.75)** — George Loewenstein, *Psychological Bulletin* (1994) - la curiosité naît d'un manque d'information identifié (règles 1, 2)
- **[Information foraging in information access environments](https://doi.org/10.1145/223904.223911)** — Peter Pirolli & Stuart Card, CHI '95 (1995) - *information scent* : le lecteur suit les indices d'une récompense (règles 1, 5)
- **[The role of transportation in the persuasiveness of public narratives](https://doi.org/10.1037/0022-3514.79.5.701)** — Melanie Green & Timothy Brock, *Journal of Personality and Social Psychology* (2000) - partir d'une situation vécue (règle 2)
- **[Processing Fluency and Aesthetic Pleasure](https://doi.org/10.1207/s15327957pspr0804_3)** — Rolf Reber, Norbert Schwarz & Piotr Winkielman, *Personality and Social Psychology Review* (2004) - ce qui se traite facilement plaît (règles 3, 6)
- **[What Makes Online Content Viral?](https://doi.org/10.1509/jmr.10.0353)** — Jonah Berger & Katherine Milkman, *Journal of Marketing Research* (2012) - utilité pratique et transmission (fiche : grande idée)
- **[The Elaboration Likelihood Model of Persuasion](https://doi.org/10.1016/S0065-2601(08)60214-2)** — Richard Petty & John Cacioppo, *Advances in Experimental Social Psychology* (1986) - un lectorat motivé est convaincu par les arguments, pas par l'emballage (règle 6)
- **[Job Market Signaling](https://doi.org/10.2307/1882010)** — Michael Spence, *The Quarterly Journal of Economics* (1973) - analogie : un signal coûteux (compromis, comparaison loyale) crédibilise plus qu'une déclaration (règle 6)
- **[Hacker News Guidelines](https://news.ycombinator.com/newsguidelines.html)** et **[Show HN Guidelines](https://news.ycombinator.com/showhn.html)** - titre original, pas de promotion, blog posts exclus de Show HN
