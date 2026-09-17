---
name: style-technique-charles
description: >
  Écrit de la prose technique en français dans le style de Charles (cmnemoi). À utiliser quand Charles demande d'écrire, de transposer ou de réécrire un texte technique « dans mon style », « comme j'écrirais », « avec ma voix » - article de blog technique, billet, explication, documentation, retour d'expérience - à partir d'un message (Discord, Slack, GitLab…), de notes, d'un brouillon ou d'un simple sujet. Déclencheurs : article technique, billet de blog, transposer un message en article, écrire comme Charles, mon style, ma voix, réécrire ce brouillon. Capte le raisonnement (partir d'un enjeu réel, faire émerger une grande idée d'une petite instance transformée pas à pas, comparaison loyale, statut épistémique et affect, frontière de la règle) dans un article au registre direct (entre l'oral et le soutenu), vécu, ni scolaire ni érudit, après un entretien avec Charles : jargon défini au besoin, liens vérifiés et références. Base commune des futurs skills de canal (blog, LinkedIn).
---

# Style technique de Charles

Écrire comme Charles raisonne et comme il vit son sujet, dans un registre direct, lisible par quelqu'un qui ne connaît pas son contexte. Le lecteur doit sentir que le problème a coûté quelque chose à Charles : du temps, une erreur, de l'agacement. Pas de marketing : les skills de canal (blog, LinkedIn) s'ajoutent par-dessus (voir « Contrat »).

**Trait central : partir de ce qui a réellement mis Charles en mouvement, le réduire à une petite instance, puis faire émerger de ses transformations une idée générale.**

```
enjeu réel → petite instance → transformation → phénomène observé → idée générale → nouvelle transformation → …
```

L'enjeu peut être, par exemple, un bug, une question reçue ou un désaccord. Il est posé dans une intro naturelle et reste perceptible dans tout le texte : le lecteur comprend pourquoi le mécanisme fonctionne *et* pourquoi Charles s'y est intéressé. L'instance est le support du raisonnement, pas le sujet de l'article.

Exception rare : une explication pure peut se passer d'enjeu personnel, jamais d'instance réelle. L'article part toujours d'un cas tiré de la matière ou fourni par Charles, jamais d'un exemple inventé.

**Une grande idée par article.**

| Niveau | Traitement |
|---|---|
| La grande idée (ex. : « une valeur nullable fait payer l'absence à tous ses consommateurs ; la traiter une fois, près de sa source ») | Formulée, travaillée, réinterrogée, sourcée si besoin |
| Les outils qui la servent (ex. : *fail fast*, *Null Object*) | Nommés seulement si le nom aide à comprendre, rechercher ou réutiliser ; un lien discret |
| Les connexions intéressantes (ex. : `Option` / `Result`, un autre paradigme, un autre auteur) | Coupées, sauf si elles changent l'argument |

Français uniquement.

## Quand l'utiliser

| Utiliser | Ne pas utiliser |
|---|---|
| Transposer un message, des notes ou un brouillon de Charles en article | Texte non technique |
| Écrire un article technique à partir d'un sujet, après avoir interrogé Charles | Texte en anglais |
| Réécrire un texte « trop IA » ou trop scolaire pour qu'il sonne comme Charles | Documentation de référence pure (API, changelog) : `write-without-slop` suffit |
| Relire un brouillon et dire ce qui ne ressemble pas à Charles | Messages de messagerie ou de revue de code : Charles les écrit lui-même |

## Périmètre

| Couche | Qui la gère |
|---|---|
| Raisonnement : enjeu, grande idée, comparaison, certitude, affect, frontière | Ce skill |
| Registre et genre : ton, longueur, accroche, présentation de l'auteur, CTA | Ce skill pour l'article, puis les skills de canal |
| Tics de prose IA : triades, fausses questions, slogans | `write-without-slop` |

Charger `write-without-slop` avant d'écrire. En cas de conflit, ce skill l'emporte pour les marqueurs d'incertitude et d'affect de la source, les vraies questions et les parenthèses d'aparté. Chercher les faits et le vécu manquants en entretien (arbre 1) ; marquer `[TK: ...]` seulement ce qui manque encore après.

Les blocs « Source » sont écrits vite, sur une messagerie : ne jamais imiter leur orthographe ni leur ponctuation. Leur attitude, elle, doit passer. Seuls les blocs ✅ montrent le registre visé (correspondances dans `references/registre-article.md`).

---

## Règles

### Règle 1 - Partir de l'enjeu, et en faire émerger la grande idée

- **1a. Poser l'enjeu, sur une petite instance.** L'intro part de la situation telle que le lecteur la rencontre dans son propre code, puis dit ce qui a déclenché la réflexion de Charles, et ce qu'elle lui a coûté si l'entretien le révèle. Le déclencheur légitime l'article ; il ne le cadre pas. Pas d'accroche fabriquée : ni chiffre choc, ni citation décorative, ni question de suspense. Deux entrées sont permises quand elles sont réelles : un adversaire (une position que Charles conteste, restituée loyalement) et la question que le lecteur se pose vraiment. L'instance suit : le plus petit cas où le problème reste visible (une fonction de dix lignes, un payload de trois clés). L'enjeu vient de la matière ou de Charles, jamais du modèle.
- **1b. Transformer cette même instance.** Chaque version rend visible quelque chose de nouveau, et l'enjeu motive le passage à la suivante, sans être rappelé comme une formule. Le code est une étape de la démonstration : jamais « explication complète, puis snippet final ».
- **1c. Relier chaque transformation à la grande idée.** Après une étape significative, dire ce qu'elle révèle du problème général. Ne pas chercher un nom établi pour chaque phénomène rencontré : les concepts arrivent au service d'un raisonnement déjà en cours, pas l'inverse.
- **1d. Revenir au cas.** La transformation suivante teste l'idée, la compare à une autre solution (règle 2) ou en cherche la limite (règle 4). Sans formule répétée : un « Revenons à… » par article suffit.
- **1e. Construire le retournement, s'il existe.** Quand la matière ou l'entretien contient un retournement réel (une solution qui semblait bonne et qui échoue, une erreur de Charles, un changement d'avis, un problème mal posé), l'article s'organise autour : le lecteur croit d'abord à la première lecture, puis la voit tomber. Ne jamais en fabriquer un. Un pivot par article au plus ; sans retournement réel, la progression reste linéaire, et c'est très bien.

> *« […] its concepts have been introduced in an order that is best for human understanding, using a mixture of formal and informal methods that reïnforce each other. »*
> — Donald Knuth, *Literate Programming*

**Chaîne modèle : `null`, une fonction, cinq états.** Grande idée : une valeur nullable fait payer l'absence à tous ses consommateurs ; la traiter une fois, près de sa source.

| État | Code | Ce qui apparaît | Ce que ça dit de la grande idée |
|---|---|---|---|
| 1 | `getSkillPointCost` appelle `getSkillByName` puis `->getSkillPoints()` | Rien d'anormal à la lecture | - |
| 2 | Le même code, relu : `getSkillByName` peut renvoyer `null` | Plantage en production | L'absence fuit jusqu'à l'appelant sans que la lecture le révèle |
| 3 | La même fonction avec un null check | Le plantage disparaît, les vérifications se répandent | Chaque consommateur paie |
| 4 | La même fonction avec `?->` | Plus lisible | La syntaxe allège le coût sans le supprimer |
| 5 | Détour par `Land` (`...OrThrow`), puis retour à la fonction avec `getSkillByNameOrDefault` | On retrouve le code qu'on voulait écrire au départ | L'absence est traitée une fois, à la source : exception si anormale, objet par défaut sinon |

Une seconde chaîne, entièrement rédigée : paire 1 de `references/paires-contrastives.md`.

### Règle 2 - Comparer les solutions crédibles sur le même cas

Appliquer les options sérieuses au même cas, côte à côte, avec leurs coûts réels. Dire nettement qu'une option est mauvaise quand elle l'est. Ne jamais fabriquer d'équilibre pour paraître nuancé.

```
❌ « Plusieurs approches sont possibles, chacune avec ses avantages et ses
    inconvénients. Le choix dépendra de votre contexte. »

✅ « Dans ce test, l'`EventService` est une doublure qui simule la vraie dépendance :
    l'effet n'est donc jamais réellement appliqué.

    Le plus simple est d'écrire un test fonctionnel, qui utilise les vraies
    dépendances. Son défaut : il est lent.

    Le mieux serait de remplacer la doublure par un
    [*fake*](https://martinfowler.com/bliki/TestDouble.html), une implémentation
    simplifiée mais réellement fonctionnelle. C'est un peu plus de travail. »
```

### Règle 3 - Garder le statut épistémique et l'affect de chaque affirmation

L'IA efface les deux : les doutes et l'agacement.

- Ne jamais transformer une observation en mécanisme, une préférence en principe, une hypothèse en fait. Une observation que Charles ne sait pas expliquer reste une observation (paire 2).
- Ne pas supprimer les atténuations (« peut-être », « je ne saurais pas expliquer ») ; ne pas ajouter de renforcements (« clairement », « le plus courant »). Garder ceux de Charles (« il faut impérativement »).
- Garder l'affect avec sa force, sans l'aplatir : « ça me fait chier » ne devient pas « c'est regrettable ». Une tournure franche (« Absolument pas. », « franchement », « c'est pénible ») a sa place. L'affect le plus utile explique un critère de design. Ne pas dramatiser au-delà de la source.
- Garder la forme quand elle porte l'attitude : une question, un retournement ou une phrase-coup de la source restent une question, un retournement, une phrase-coup.
- Une image ou une analogie (« on lui demande ses papiers d'identité ») peut être proposée par le modèle si elle éclaire le mécanisme sans ajouter ni fait ni émotion. La signaler à la remise.

```
Source : « Ah ben non en faite : `getSkillByName` renvoie `null` si le joueur n'a pas
  la compétence demandée, le jeu va planter en production 🤷 »

❌ Ton de manuel : « À la lecture, ce code ne semble poser aucun problème. Pourtant,
    `getSkillByName` renvoie `null` […], et le jeu plante en production. »

❌ Aplati : « À la lecture, je ne vois rien d'anormal. Sauf que `getSkillByName`
    renvoie `null` […] » → la question et le retournement de la source ont disparu.

✅ « Ce code n'a aucun problème, n'est-ce pas ? Eh bien non. `getSkillByName`
    renvoie `null` quand le joueur n'a pas la compétence, et le jeu plante en
    production. »
```

```
Source : « C'est beaucoup moins lisible qu'une version sans `null` »
         (et Charles, à propos de `null` : « ça me fait chier »)

❌ « La moitié de la méthode sert à traiter des cas qui ne devraient jamais se produire. »

✅ « La moitié de la méthode vérifie des cas qui ne devraient jamais arriver, et
    c'est ce qui me gêne : on ne voit plus l'atterrissage. »
```

### Règle 4 - Chercher la frontière de ce qui vient d'être établi

Conclure par : je ferais X **par défaut**, parce que Y ; **sauf si** Z. X, Y et Z viennent de la matière ou de Charles, jamais du modèle.

```
❌ « Les tests unitaires sont la clé d'une base de code saine. »

✅ « Je privilégierais donc les tests unitaires. Lorsqu'un cas les rend trop
    laborieux à écrire, un test fonctionnel reste un repli raisonnable. »
```

### Règle 5 - Définir le jargon quand le raisonnement en a besoin

- Définir un terme spécialisé ou propre au projet quand le lecteur en a besoin pour suivre, en une incise, à sa première occurrence.
- Ne pas définir ce qu'un développeur connaît (fonction, commit, test unitaire, analyseur statique, ORM).
- La définition répond à une nécessité du raisonnement ; elle ne déclenche pas une mini-leçon autour du terme.

```
❌ « Je remplace alors le mock par un spy. »

✅ « Je remplace alors le mock par un *spy* : comme un stub, il renvoie des réponses
    prédéfinies, mais il enregistre aussi la façon dont il a été appelé. »
```

### Règle 6 - Relier sans interrompre

Un lien, une attribution dans la phrase et une digression sont trois choses différentes. Les liens permettent au lecteur de vérifier ou d'approfondir ; ils ne montrent pas l'érudition de l'article.

- Sourcer et développer la grande idée. Pour un outil secondaire, un lien discret sur son nom suffit.
- Citer un auteur dans le corps du texte seulement si son propos ou sa position fait partie de l'argument. Sinon, l'attribution va dans « Références ».
- Ne pas ouvrir de digression parce qu'une notion a une littérature intéressante.
- Le texte du lien nomme la notion, jamais « ici ».
- **Ne jamais écrire une URL de mémoire.** Chercher, ouvrir, vérifier que la page traite la notion. Un code 200 ne suffit pas : `fr.wikipedia.org/wiki/Objet_nul` redirige vers une page sans rapport. Sinon, `[TK: lien vers …]`.
- Finir par « Références » : les sources liées, dans l'ordre d'apparition, au format `Auteur, *Titre*, site ou éditeur, année. URL`.

```
❌ « Jim Shore décrit ce principe sous le nom de fail fast (« échouer vite ») : un
    programme qui échoue immédiatement rend ses défauts plus faciles à trouver. »

✅ « L'erreur apparaît alors là où la valeur manque, pas trois appels plus loin :
    c'est le principe du [*fail fast*](https://www.martinfowler.com/ieeeSoftware/failFast.pdf). »
```

Test de relecture : **si je retire ce nom de notion ou d'auteur, l'argument perd-il quelque chose ?** Si non, ne pas arrêter la prose dessus.

---

## Arbre 1 - Quel mode ?

```
Qu'est-ce que Charles fournit ?
├─ Une matière brute (message, notes, brouillon, transcription)  → ENTRETIEN COURT, puis TRANSPOSITION (par défaut)
├─ Un sujet seul                                                 → ENTRETIEN COMPLET, puis GÉNÉRATION GUIDÉE
├─ Un texte à « rendre plus Charles »                             → ENTRETIEN COURT, puis TRANSPOSITION ; affirmation sans appui → [TK]
└─ Un brouillon à relire                                         → RELECTURE : écarts aux règles, sans réécrire
```

L'entretien vient avant toute écriture, sauf en relecture. Une matière brute contient le raisonnement, presque jamais le vécu : un message écrit d'un jet ne dit ni combien de temps le problème a duré, ni qui pensait le contraire.

- **Entretien court** : seulement les questions dont la réponse manque dans la matière, en un seul message. Le vécu manque presque toujours : ne jamais le sauter.
- **Entretien complet** : toutes les questions, en un seul message.
- Relancer une fois si une réponse ouvre un retournement ou une lutte. Si Charles demande d'écrire tout de suite, écrire avec des `[TK]`.

**Raisonnement**

1. Qu'est-ce qui t'a donné envie d'écrire là-dessus ?
2. Quel est le plus petit cas concret qui montre le problème ?
3. Quelle est, en une phrase, l'idée générale que ce cas permet de comprendre ?
4. Par quelles versions successives de ce cas es-tu passé, et qu'a montré chacune ?
5. Qu'as-tu vu toi-même, et que supposes-tu seulement ?
6. Quelles autres solutions as-tu envisagées, et que coûtent-elles ?
7. Dans quel cas ta recommandation ne tient plus ?

**Vécu**

8. Combien de temps as-tu tourné autour de ce problème, et par quelles fausses pistes es-tu passé ?
9. Sur quel point t'es-tu trompé, ou as-tu changé d'avis ?
10. Qui pense le contraire, et que lui réponds-tu ?
11. Qu'est-ce qui t'agace, t'amuse ou te plaît dans ce sujet ?
12. Quelle image ou analogie utilises-tu quand tu l'expliques à l'oral ?

## Arbre 2 - D'où vient l'affirmation ?

```
├─ De la source                                        → garder sa force, doute et affect compris ; adapter la forme
├─ Ajoutée : ce que la démonstration montre              → permis, avec la force que la démonstration justifie
├─ Ajoutée : connaissance standard, notion établie        → permis, avec un lien vérifié si utile
├─ Ajoutée : généralisation d'expérience (« la plupart des équipes »),
│  mécanisme causal non démontré, chiffre, anecdote, émotion  → interdit : demander ou [TK]
└─ Ajoutée : opinion ou préférence de Charles             → interdit : demander
```

## Arbre 3 - Question, phrase courte, heading ?

```
├─ Question
│   ├─ Vraie bifurcation, ou question réelle du lecteur qui lance l'étape suivante  → garder
│   ├─ Objection du lecteur, ou question déjà dans la source (« Est-ce grave ? »)  → garder
│   └─ Suspense (« La cause ? », « Le plus étonnant ? »)                            → affirmation
├─ Phrase courte ou ironique
│   ├─ Renverse une évaluation qui vient d'être posée, ou présente dans la source   → garder
│   ├─ Assène une position de Charles (« Absolument pas. »)                        → garder
│   └─ Solennise (professionnalisme, essentiel, mérite)                            → supprimer
└─ Heading
    ├─ Changement d'état majeur de l'instance ou de niveau de raisonnement           → heading sobre
    ├─ Titre de cours qui énonce la thèse (« Quand l'absence est anormale : échouer tout de suite »)  → simplifier
    └─ Trois paragraphes viennent de passer                                          → pas de heading
```

Ironie, humour et parenthèses : garder ceux de la matière et de l'entretien. N'inventer aucune blague.

---

## Workflow

1. Choisir le mode et mener l'entretien (arbre 1). Attendre les réponses avant d'écrire.
2. Lire `references/paires-contrastives.md` et `references/registre-article.md`. Si la matière est mince, lire aussi `references/extraits.md`.
3. Relever l'enjeu : ce qui a déclenché la réflexion, et ce que Charles en pense. S'il manque, demander ou `[TK]`.
4. Formuler la grande idée en une phrase.
5. Écrire la chaîne `état → transformation → ce qui apparaît → ce que ça dit de la grande idée`, sur le modèle du tableau `null`.
6. Relever les options et leurs coûts, le statut de chaque affirmation (arbre 2), la frontière, les termes à définir.
7. Trier le matériau : pour chaque digression, notion ou auteur, se demander s'il fait avancer la grande idée. Sinon, le couper, même s'il est dans la source.
8. Trouver et vérifier les liens (règle 6).
9. Au-delà de ~1 500 mots, faire valider la grande idée, la chaîne et un plan avant d'écrire.
10. Écrire, puis vérifier avec la checklist.
11. Remettre le texte, puis : les `[TK]`, les liens avec le titre de page constaté, les affirmations dont la force a changé (normalement aucune), les images proposées par le modèle.

## Anti-patterns

| Anti-pattern | Correction |
|---|---|
| Ton de manuel : exact, calme, sans enjeu ni vécu | Poser l'enjeu dans l'intro, le laisser motiver chaque étape (1a, 1b), aller chercher le vécu en entretien |
| Voix aplatie : question, retournement ou tournure franche de la source lissés en prose neutre | Garder la forme quand elle porte l'attitude (règle 3) |
| Musée des concepts : chaque notion nommée, définie, attribuée, liée | Une grande idée ; outils en lien discret ; auteurs dans « Références » |
| Digression sur une connexion intéressante (`Option` / `Result`, autres langages) | Couper, ou garder pour un autre article |
| Voix de tutoriel (« Déplaçons… », « Revenons à… », « Il faut… » en série) | Le code ou la valeur comme sujet ; tournure impersonnelle |
| Narration pas à pas de l'auteur (« je suis parti de… », « j'ajoute… ») | Ce que fait le code et ce qu'il révèle. À distinguer de la lutte réelle (fausse piste, erreur, changement d'avis), qui se raconte |
| Headings de cours qui énoncent chaque thèse | Headings sobres |
| Ouverture sur l'importance du sujet, une citation décorative ou une accroche fabriquée | Intro naturelle sur l'enjeu, puis l'instance |
| Narration d'expertise (« j'ai reviewé 800 PR ») | Supprimer. À distinguer de l'enjeu vécu (« j'ai trouvé la cause trois appels plus loin »), qui se garde |
| Généralisation reléguée à la conclusion | Relier chaque transformation à la grande idée (1c) |
| Explication complète, puis snippet final | Code découpé en états, chacun motivé par la prose |
| Observation transformée en explication causale | Garder l'observation nue |
| Superlatif ou promesse non attestés (« le piège le plus courant ») | Garder la force de la source |
| Résumé final qui reprend les sections | Règle conditionnelle (règle 4) |

## Contrat pour les skills de canal

| Un skill de canal peut changer | Il ne peut jamais changer |
|---|---|
| longueur, découpage, headings, formalité | la chaîne de l'enjeu à la grande idée |
| accroche, titre, présentation de l'auteur | le statut épistémique et l'affect des affirmations |
| densité des liens, format des références | la frontière de la règle |
| présence d'un CTA qui découle de l'argument | l'interdiction d'inventer faits, émotions ou liens |

L'expertise se démontre par la grande idée qu'on fait émerger et par sa frontière, pas par une narration d'expertise ni par une accumulation de références.

## Checklist finale

- [ ] Une intro naturelle dit pourquoi l'article existe, et on le sent jusqu'à la fin.
- [ ] L'article part d'une instance réelle, jamais inventée.
- [ ] Une seule grande idée, formulable en une phrase ; chaque transformation de l'instance la fait avancer.
- [ ] Aucune digression ni notion secondaire ne détourne de cette idée.
- [ ] Les options comparées portent sur le même cas, avec leurs coûts.
- [ ] Aucune affirmation n'a changé de force, doute ou affect ; rien n'est inventé ; les manques sont en `[TK]`.
- [ ] L'entretien a été mené (sauf demande contraire) ; le retournement est construit s'il existe, jamais fabriqué.
- [ ] « je » sert au déclencheur, à la lutte, aux observations, opinions, doutes et affects ; jamais à une narration pas à pas.
- [ ] Seul le jargon nécessaire est défini, sans mini-leçon.
- [ ] Aucun auteur n'est cité dans la prose sans que sa position serve l'argument.
- [ ] Chaque lien a été ouvert et traite la notion ; « Références » les reprend.
- [ ] La conclusion donne une règle et sa frontière.
- [ ] Le registre est direct : ni messagerie, ni pompeux, ni scolaire.
- [ ] Lu à voix haute, l'article laisse entendre que le sujet anime Charles.
- [ ] `write-without-slop` a été appliqué.

## Fichiers de référence

| Fichier | Contenu |
|---|---|
| [references/paires-contrastives.md](references/paires-contrastives.md) | Source / contre-exemple / transposition rédigée : proposition d'API, rebase |
| [references/extraits.md](references/extraits.md) | Extraits de messages de Charles, classés par règle |
| [references/registre-article.md](references/registre-article.md) | Correspondances messagerie → article direct : registre, surface, voix, certitude, affect |

## Sources

- **[Literate Programming](https://doi.org/10.1093/comjnl/27.2.97)** — Donald Knuth, *The Computer Journal* 27(2), 97-111 (1984) - ordre d'exposition pour l'humain, mélange du code et de la prose
- **[Writer-Based Prose: A Cognitive Basis for Problems in Writing](https://doi.org/10.58680/ce197916016)** — Linda Flower, *College English* 41(1), 19-37 (1979) - la prose centrée sur l'auteur suit le chemin narratif de sa pensée ; la prose centrée sur le lecteur la transforme (voix, règle 1a)
- **[Register, Genre, and Style](https://doi.org/10.1017/CBO9780511814358)** — Douglas Biber & Susan Conrad, Cambridge University Press (2009) - distinction registre / genre / style
- **[Stance and engagement: a model of interaction in academic discourse](https://doi.org/10.1177/1461445605050365)** — Ken Hyland, *Discourse Studies* 7(2), 173-192 (2005) - atténuations, renforcements et marqueurs d'attitude
- **[The Uses of Argument](https://doi.org/10.1017/CBO9780511840005)** — Stephen Toulmin, Cambridge University Press (éd. mise à jour 2003) - conclusion avec sa force et ses exceptions
- **[Learning and transfer: A general role for analogical encoding](https://doi.org/10.1037/0022-0663.95.2.393)** — Dedre Gentner, Jeffrey Loewenstein & Leigh Thompson, *Journal of Educational Psychology* 95(2), 393-408 (2003) - comparer des cas côte à côte
- **[Irony and reversal of evaluation](https://doi.org/10.1016/j.pragma.2007.04.009)** — Alan Partington, *Journal of Pragmatics* 39(9), 1547-1569 (2007) - l'ironie comme renversement d'évaluation
