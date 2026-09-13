# De la messagerie à l'article soutenu

Garder le raisonnement, les incertitudes, l'attitude et les apartés. Changer la surface. Soutenu ne veut pas dire pompeux (« force est de constater », « il convient de noter ») ni scolaire (voix de manuel, définitions de l'évident, liens en série).

## Voix

- « je » pour ce que Charles a vécu, fait, pense : « j'ajoute la vérification », « je pense ».
- « nous » seulement pour un raisonnement mené avec le lecteur, et avec parcimonie.
- Pas de cadence de tutoriel : « Déplaçons… », « Revenons à… », « Il faut… » en série.

## Surface

| Dans la matière | Dans l'article |
|---|---|
| « en gros », « en vrai », « bref », « perso », « imo » | Supprimé, ou « à mon sens », « il me semble » |
| « y a », « bcp », « ça », négation sans « ne », orthographe approximative | « il y a », « beaucoup », « cela », négation complète, orthographe corrigée |
| « tu » | « vous », « nous » ou tournure impersonnelle, selon la voix (voir ci-dessus) |
| « Question con, mais… », « je dis n'importe quoi » | La formule disparaît, l'incertitude reste : « je n'ai pas vérifié cette piste », « idéalement » |
| Emojis, spoilers, « /!\ » | Supprimés ; l'attitude qu'ils portent passe dans les mots (voir « Affect ») |
| Parenthèses nombreuses, notes (1) | Gardées si elles portent un sens ; une note peut devenir une parenthèse |
| « 1) », « (1) » au fil du texte | Liste Markdown pour une vraie énumération |
| Liens vers des messages, tickets, merge requests | Remplacés par leur contenu utile |
| Code brut, commentaires mêlés | Même chaîne de versions ; erreurs évidentes corrigées et signalées |

## Marqueurs de certitude

La forme change, la force reste.

| Statut | Dans la matière | Dans l'article |
|---|---|---|
| a observé | « ça marche pas », « j'ai vu que » | « je constate que », « en pratique, cela ne fonctionne pas » |
| pense | « je pense », « pour moi », « imo » | « je pense », « à mon sens » |
| préfère | « perso je ferais », « j'ai tendance à privilégier » | « je privilégierais », « je ferais plutôt » |
| soupçonne | « peut être », « j'ai l'impression », « je suppose » | « peut-être », « il me semble », « je suppose » |
| ne sait pas | « je sais pas », « je dis n'importe quoi » | « je ne sais pas », « je n'ai pas vérifié cette piste » |
| a changé d'avis | « et peut être pas aussi prioritaire que je pensais » | « et, à la réflexion, peut-être moins prioritaire que je ne le pensais » |

## Affect

La forme change, la force reste. Rester sobre : un mot juste, pas une image ni un adjectif appuyé ; pas de dramatisation absente de la source.

| Dans la matière | Dans l'article |
|---|---|
| « Ah ben non en faite : […] le jeu va planter en production 🤷 » | « Sauf que non. […] et le jeu plante en production. » |
| « Absolument pas » | « Absolument pas. » (se garde tel quel) |
| « Sans logs, on va continuer à se taper des bugs incompréhensibles en prod » | « Sans logs, nous continuerons à subir des bugs incompréhensibles en production. » |
| « combien de fois le test plantait juste par ce que j'avais pas exactement la même que ce qui est en jeu 😭 » | « Je ne compte plus les fois où le test a planté simplement parce que ma donnée différait de celle du jeu. » |
| « y a des cas parfois qui pique un peu (mais juste un peu, donc ça va) » | « certains cas piquent un peu (juste un peu) » |
| « Very stupid design (it's mine) » | « une conception franchement mauvaise (la mienne) » |
| « Franchement ça ce serait incroyable, mais peut être un peu surdimensionné » | « Ce serait vraiment formidable, mais peut-être surdimensionné. » |

## Exemple

Source :

```
Une mécanique qui implique le hasard est parfois un peu compliquée à tester
ça dépend de ton implémentation
[…]
||je dis n'importe quoi, mais idéalement il te faudrait une fonction qui prend des probabilités en entrée et tu vérifies dans ton test si elle retourne bien les bonnes probabilités en sortie||
```

```
❌ Trop lissé : « Tester une mécanique aléatoire nécessite une approche rigoureuse.
   La meilleure pratique consiste à isoler la génération de probabilités. »
   → « idéalement » a disparu ; « la meilleure pratique » est un renforcement ajouté.

❌ Trop messagerie : « ça dépend de ton implémentation. ||je dis n'importe quoi, mais…|| »

✅ « Une mécanique qui dépend du hasard peut être compliquée à tester, et dépendre
   beaucoup de l'implémentation : idéalement, nous devrions avoir une fonction qui
   prend des probabilités en entrée, et le test vérifierait qu'elle retourne les
   bonnes probabilités en sortie. »
```
