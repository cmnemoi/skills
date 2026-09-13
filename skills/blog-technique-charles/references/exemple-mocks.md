# Exemple : passe blog sur « Ce que vérifie un test rempli de mocks »

Mode PASSE BLOG : l'article est déjà rédigé avec `style-technique-charles`. Tout ce qui suit vient du texte ; rien n'est ajouté.

## Fiche éditoriale

| Champ | Contenu |
|---|---|
| Lecteur | Développeur qui écrit ou relit des tests unitaires avec des mocks, sans avoir tranché entre les deux écoles |
| Déclencheur | Février 2025, revue d'une merge request d'eMush : un contributeur ne comprend pas ce qu'un test vérifie |
| Question ouverte | Que vérifie ce test, au juste ? |
| Grande idée | Un test mocké vérifie ses mocks : quand on améliore le code sans changer son comportement, il faut aussi réécrire le test, alors que c'est lui qui devait garantir que rien n'a changé |
| Preuve maîtresse | Le même refactoring : le test de Londres échoue, le test de Chicago passe. Puis le bug inverse, que les deux détectent |
| Signal | Comparaison loyale (le vrai bogue), défauts de l'école de Chicago, *fakes* |
| Longueur | ~1 250 mots : adaptée, rien à couper hors intro |
| Goulet | **Intro** : elle annonce un résumé de cours au lieu de la question que le lecteur partage |

## Diagnostic, étape par étape

| Étape | État | Action |
|---|---|---|
| Titre | Sujet explicite, question ouverte | Garder |
| Intro | Phrase méta (« Il y a beaucoup à dire là-dessus, de quoi remplir un cours entier ») ; promesse d'un résumé des deux écoles | Goulet : réécrire (ci-dessous) |
| Premier « ah » | « Le test ne contient aucune assertion » arrive dans la 2e section | Bon |
| Diagonale | Intertitres = étapes du cas ; seul « Le test » est faible, mais il ouvre l'instance | Garder |
| Preuve maîtresse | « Modifier le code sans changer son comportement », à ~60 % | Rendue désirable par l'intro réécrite |
| Idée transmissible | Présente, formulée au bon endroit (fin de « Modifier le code… ») | Garder |
| Conclusion | Frontière vague (« à moins d'avoir une très bonne raison ») + `[TK]` | Demander à Charles le cas où un mock serait justifié |
| CTA | `[TK]` | Demander à Charles |

## Intro

```
❌ Actuelle : « […] il ne comprenait pas ce que ce test vérifiait. Il y a beaucoup à
   dire là-dessus, de quoi remplir un cours entier. Je lui ai répondu en résumant
   l'opposition entre deux écoles de test, et je reprends ici cette réponse sur le
   test qui l'a provoquée. »
   → phrase méta ; la promesse est un cours ; rien ne rend la suite désirable.

✅ « Pendant trois ans, j'ai maintenu eMush, un jeu multijoueur open source. En février
   2025, en relisant une merge request, j'ai vu un contributeur buter sur un test
   unitaire : il ne comprenait pas ce que ce test vérifiait. Il avait de bonnes raisons.
   Ce test n'affirme rien sur le message produit, et il passerait avec n'importe quel
   prénom. Je reprends ici la réponse que je lui ai faite, en opposant deux écoles de
   test sur ce test précis. »
   → chaque fait vient de l'article ; la question devient celle du lecteur.
```

## Grande idée : ce qu'il ne faut pas faire

```
❌ « Les mocks ne rendent pas automatiquement un test fragile ; vérifier son
   implémentation, si. »
   → plus consensuelle que Charles, qui écrit : « Les mocks sont à éviter, à moins
     d'avoir une très bonne raison de les utiliser ; sur eMush, je n'en vois pas. »

✅ « Pour améliorer le code sans changer son comportement, il faut aussi réécrire le
   test, alors que c'est précisément lui qui devait garantir que rien n'a changé. »
   → déjà dans l'article, à la force de Charles.
```

## Titres

1. **Ce que vérifie un test rempli de mocks** (recommandé : sujet + question)
2. Ce test passerait avec n'importe quel prénom (artefact fort, mais sujet implicite)
3. Tests unitaires : l'école de Londres contre l'école de Chicago, sur un vrai test (explicite, moins de tension)

## Packaging

| Élément | Proposition |
|---|---|
| Description | Un test unitaire d’eMush sans assertion sur son résultat, réécrit sans mocks : ce que chaque version détecte quand on modifie le code. |
| Slug | `test-rempli-de-mocks` |
| Auteur, CTA | `[TK]` |

## Partage

LinkedIn :

```
❌ « Nouvel article sur les mocks ! Londres ou Chicago ? Dites-moi en commentaire 👇 »

✅ « En relisant une merge request d'eMush, j’ai vu un contributeur buter sur un de nos tests
   unitaires : il ne comprenait pas ce qu’il vérifiait. Il n’affirmait rien sur le
   message produit.

   Je l'ai réécrit sans mocks, puis j'ai modifié le code sans changer son
   comportement. L'ancien test échoue, le nouveau passe.

   [lien] »
```

Hacker News : titre original, soumission normale. Article en français : ne le soumettre que si Charles le souhaite.
