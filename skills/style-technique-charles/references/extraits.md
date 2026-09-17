# Extraits de messages de Charles

Messages Discord (2022-2026), classés par règle. Ils montrent des mouvements de raisonnement et une attitude ; la surface se transpose (`registre-article.md`). Les extraits déjà cités dans `SKILL.md` ou dans les paires ne sont pas repris ici.

---

## Règle 1 - Entrer par le frottement

- « J'ai un bug assez gênant » + la stack trace.
- « Je vois trois points qui peuvent poser problème : »
- « Question con, mais je me rends pas compte de ce qu'on est censé logger » + le code actuel.
- « J'ai un exemple : » + une fonction.

## Règle 1c - Nommer la notion générale

```
C'est une version simplifiée et moins puissante d'une gestion des erreurs qui prend de plus en plus d'importance en Java ou JavaScript et qui est standard en Scala, Kotlin ou Rust
```

```
comme on savait pas le faire au début et qu'on a de la complexité accidentelle (généralement mal nommée dette technique)
```

## Règle 2 - Comparer sur le même problème

Tester une query, deux voies, deux coûts (2025-04-04) :

```
1) […] More of a white box approach, it's easier to do, but might be less accurate
2) […] this is closer to the real usage of the query, but this is harder to do, and the test will have a slower feedback loop so this can be frustrating
```

Pas de fausse neutralité :

- « Absolument pas » (sur `null`).
- « They are not good examples of tests, so I would rather explain what a good test should look like » (2025-06-08).
- « ❌ Discord est trop fermé à mon goût. ❌ Les forums Eternaltwin ne sont pas encore au point imo » : options écartées avec leur raison, puis proposition (2023-01-29).

---

## Règle 3 - Statut épistémique

Logs : hypothèse, doute, aveu (2023-02-04) :

```
Ce que je veux savoir éventuellement c'est
(1) un snapshot de l'user, du player, du daedalus etc à l'instant du bug
(2) la stack trace (qui a appelé quoi)

Pour la (1) je suppose que je peux passer l'entité dans les paramètres directement, au pire JSONifiée
Mais je me suis dit que ça serait peut être trop lourd / illisible ?
Pour la (2), je ne sais pas ?
```

Changement d'avis visible (2023-03-06) :

```
1) quand il y a un bug sur un vaisseau, je veux pouvoir m'y rendre pour voir la situation de mes propres yeux
    -> en vrai je ne sais pas si ça pourrait aider, au final (pas plus que les logs ?)
2) quand il y a un bug, je veux pouvoir avoir un tableau de bord du vaisseau et des joueurs pour investiguer plus facilement
[…]
Mais en vrai le 2) ça mériterait qu'on y réfléchisse bien plus longuement […]
Et peut être pas aussi prioritaire que je pensais
```

Source de la connaissance dite (2024-06-09) :

```
Je découvre aussi au fur et à mesure
[…]
(1) j'ai lu des ressources qui montrent que ça se fait avec des évènements, c'est même apprécié par certains sur des projets complexes (comme celui-ci?), mais c'est plus lourd et complexe à mettre en place surtout si c'est pas pensé dès le début
```

Correction ferme, fin honnête (2025-07-03) :

```
You confuse the global configuration of the difficulty with its "mode" (or threshold, if you want a better name).
[…]
The exact modalities on how to do that are still blurry to me, and is a future-you problem (if you want it)
```

Hypothèses concurrentes, sans trancher (2025-07-07) :

```
Je pense que la personne qui a programmé ça ne connaissait pas cette règle, par ce que c'est bien documenté mais pas très connu
Ou bien effectivement ça ne concerne que les incendies post-propagation ("beta") […] (j'ai pas le code en tête)
```

Statut de la preuve corrigé (2024-04-21) :

```
D'après ~~le code source~~ l'intention du code source de Mush ce n'est pas censé en donner à chaque cycle
```

---

## Règle 4 - Frontière

```
J'ai tendance à privilégier la seconde, par ce que pour moi c'est ce qui est importe
L'utilisateur s'en fiche qu'on utilise un modificateur, il veut juste que son point spécialiste soit bien utilisé dans la bonne situation
[…]
Mais dans certains cas tu peux pas faire autrement que de tester plus proche de l'implémentation, quand tu as besoin de vérifier des contraintes prurement techniques : les tests sur les changements de cycles multiples par exemple
```

```
Perso je pense que c'est un peu sur-ingénisé pour le moment, mais que ça portera ses fruits quand on aura les autres actions de drone
```

```
Franchement ça ce serait incroyable, mais peut être un peu surdimensionné
```

---

## Règle 5 - Définir au moment où le terme sert (2025-05-07)

```
Les stubs et les mocks sont deux types de *test doubles* (ou "doublures de test", en français)

Le stub renvoit une valeur prédéfinie lorsqu'il est appelé

Le mock c'est pareil, mais il est aussi capable d'analyser comment il est utilisé et généralement c'est utilisé pour définir si cette utilisation est anormale
[…]
(Ce qui dans la littérature est défini par le terme "Dummy" mais c'est un cas particulier de Stub)
```

```
Une transaction ça permet de dire à la base de données : "ok, là je fais des changements risqués, garde ça en mémoire"
Et si y a un bug de lui dire "ok, annule tout"
```

## Règle 6 - Relier et qualifier les sources

Liens non revérifiés : appliquer la règle 6 avant toute réutilisation.

```
This is used when a class is only meant to be used as a blueprint to others through [inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming))
```

```
je pose ça là du coup, c'est une bonne introduction (même s'il élude certaines explications de base) : [lien]
Je regardais ça aussi côté Symfony : https://symfony.com/doc/current/messenger.html
```

---

## Vécu et affect (règles 1a et 3)

Sur Discord, la douleur est dite en passant, brièvement, souvent avec autodérision. Dans un article, elle peut prendre plus de place quand l'entretien la confirme : c'est elle qui montre que le sujet a coûté quelque chose. Autodérision plutôt que pathos.

- « J'ai un bug assez gênant » + la stack trace.
- « le jeu va planter en production 🤷 »
- « Mais sans logs, on va continuer à se taper des bugs incompréhensibles en prod »
- « (combien de fois le test plantait juste par ce que j'avais pas exactement la même que ce qui est en jeu 😭) »
- « y a des cas parfois qui pique un peu (mais juste un peu, donc ça va) »
- « J'ai un peu mal à la tête donc je vais développer très sommairement »
- « Je vais vraiment pas t'embêter par ce que les bases sont mauvaises et c'est pas un sujet si simple »

## Ironie

Elle relativise toujours quelque chose qui vient d'être posé.

| Extrait | Fonction |
|---|---|
| « 4) Profit? » puis « Je crois que je suis pas sûr de ce qui va se passer si l'action n'a pas de Hunter sélectionné » | Relativise son propre plan |
| « C'est quand même ballot que c'est en s'inspirant des langages fonctionnels qu'on commence à s'en débarasser » | Signale une absurdité |
| « /!\ Very stupid design (it's mine) /!\ » | Autocritique |
| « (*) je laisse la définition d'à priori en exercice » | Aparté en note |
| « mais mieux vaut un tiens que deux tu l'auras 🙂 » | Assume une solution partielle |
| « future-you problem » | Ferme un sujet non résolu sans le masquer |

## Parenthèses

Nuance, certitude, vocabulaire, source :

- « (non cloisonné, j'y reviens après) »
- « (ou threshold, if you want a better name) »
- « (après quelques tests je ne vois que les objets non primitifs […] ou `null` est obligatoire, à cause de l'ORM, bref) »
- « (dont la pertinence est à discuter) »
- « (j'ai pas le code en tête) »
