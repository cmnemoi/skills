# De la messagerie à l'article direct

Garder le raisonnement, les incertitudes, l'attitude et les apartés. Changer la surface.

Le registre visé se situe entre l'oral et le soutenu, comme chez [Julien Danjou](https://julien.danjou.info/blog/its-complicated-is-not-an-excuse/) ou [Yegor Bugayenko](https://www.yegor256.com/2014/11/20/seven-virtues-of-good-object.html) : orthographe et syntaxe soignées, phrases courtes quand la pensée tranche, avis assumés, adresse directe au lecteur, une tournure orale de temps en temps. Direct ne veut pas dire messagerie (« imo », « bcp », emojis), pompeux (« force est de constater », « il convient de noter ») ni scolaire (voix de manuel, définitions de l'évident, liens en série).

## Voix

- Le sujet des phrases est ce que le lecteur doit suivre : le code, la valeur, l'appelant. Pas la narration pas à pas de Charles (« je suis parti de… », « j'ajoute la vérification »).
- « je » pour ce que seul Charles peut attester : le déclencheur, la lutte (fausses pistes, erreurs, changements d'avis), une observation, une opinion, un doute, un affect (« je pense », « ce qui me gêne », « je ne saurais pas expliquer »).
- « nous » seulement pour un raisonnement mené avec le lecteur, et avec parcimonie ; sinon, une tournure impersonnelle.
- « vous » pour prendre le lecteur à partie quand il a une objection (« Vous voyez une différence ? Moi pas. »).

```
❌ « Pour expliquer pourquoi, je suis parti d'une fonction de dix lignes. J'ajoute
    donc la vérification. »
✅ « Une fonction de dix lignes suffit à le montrer. Un analyseur statique signale
    l'erreur ; la correction la plus directe consiste à vérifier `null` avant l'appel. »
```
- Pas de cadence de tutoriel : « Déplaçons… », « Revenons à… », « Il faut… » en série.

## Surface

| Dans la matière | Dans l'article |
|---|---|
| « en gros », « en vrai », « bref », « perso », « imo » | Supprimé, ou « à mon sens », « il me semble » ; « franchement » peut rester |
| « y a », « bcp », négation sans « ne », orthographe approximative | « il y a », « beaucoup », négation complète, orthographe corrigée ; « ça » reste permis |
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

La forme change, la force reste. Un mot franc plutôt qu'un euphémisme ; une image si elle éclaire le mécanisme ; pas de dramatisation absente de la source.

| Dans la matière | Dans l'article |
|---|---|
| « Ah ben non en faite : […] le jeu va planter en production 🤷 » | « Eh bien non. […] Le jeu plante en production. » |
| « Absolument pas » | « Absolument pas. » (se garde tel quel) |
| « Sans logs, on va continuer à se taper des bugs incompréhensibles en prod » | « Sans logs, on va continuer à se prendre des bugs incompréhensibles en production. » |
| « combien de fois le test plantait juste par ce que j'avais pas exactement la même que ce qui est en jeu 😭 » | « Je ne compte plus les fois où le test a planté simplement parce que ma donnée différait de celle du jeu. » |
| « y a des cas parfois qui pique un peu (mais juste un peu, donc ça va) » | « certains cas piquent un peu (juste un peu) » |
| « Very stupid design (it's mine) » | « une conception franchement stupide (la mienne) » |
| « Franchement ça ce serait incroyable, mais peut être un peu surdimensionné » | « Franchement, ce serait génial, mais peut-être surdimensionné. » |
| « ça me fait chier » (à propos de `null`) | « Ça m'agace. », pas « c'est regrettable » |

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
