# Paires contrastives

Chaque paire montre une matière de Charles, une version à éviter et une transposition conforme. Les sources sont écrites vite : seul le ✅ montre le registre visé. Les liens des ✅ ont été vérifiés ; ceux d'un nouvel article doivent l'être aussi. Les exemples viennent d'eMush, mais les mouvements valent pour n'importe quel projet.

---

## Paire 1 - Une proposition d'API (règle 1)

### Source (2023-09-27)

```
Pour moi, l'actuel attribut `parameter` des actions devrait un truc du genre `holder`. C'est l'entité qui "porte" l'action
Ensuite, les actions devraient accepter un `array` (un dictionnaire côté front) de string `parameters` arbitraires passées dans la requête.

ça permettrait de traiter n'importe quel input venant du front (exemple aussi des contenus des posts its par exemple) de façon simple et flexible imo
La validation de l'input se ferait dans la logique de l'action.

Par exemple pour orienter le vaisseau, le front end pourrait renvoyer :
{ action: 'orientate', holder: 'command_terminal', parameters : { orientation: 'south' } }

Et côté back end on récupérerait les paramètres :
[OrientateDaedalus qui lit $this->parameters["orientation"]]

On a en faite sûrement envie de valider la réponse du front avant de l'utiliser :
[même classe, avec orientationIsValid() et]
throw new \LogicException("The {$chosenOrientation} orientation is not valid, you little hacker!");
```

### À éviter

```
❌ « Pour gagner en flexibilité, les actions pourraient accepter un dictionnaire de
    paramètres arbitraires. Cette approche présente trois avantages : elle simplifie
    le traitement des saisies, elle unifie l'interface des actions et elle renforce
    la sécurité. Voici un exemple d'implémentation : »
    [la classe finale complète, en un seul bloc]
```

- Le raisonnement est fini avant le code ; l'étape naïve a disparu (1b).
- Aucune transformation n'est reliée à une idée générale (1c).
- « renforce la sécurité » inverse la source : l'approche naïve *oblige* à ajouter la validation (R3).
- « sûrement » et « imo » ont disparu (R3).

### Transposition

> ## Accepter des paramètres génériques, et les valider côté serveur
>
> Dans eMush, un jeu multijoueur open source, chaque action d'un joueur est envoyée au serveur sous forme de requête. Dans le code actuel, l'attribut `parameter` d'une action désigne en réalité l'entité qui « porte » l'action. Je propose de le renommer `holder`, puis de permettre à chaque action de recevoir un dictionnaire de paramètres arbitraires.
>
> Il me semble que cela permettrait de traiter simplement n'importe quelle saisie du client, par exemple le contenu d'un post-it. La validation se ferait dans la logique de l'action. [TK: quel besoin concret a déclenché cette proposition ?]
>
> Prenons l'action qui oriente le vaisseau. Le client enverrait :
>
> ```json
> {
>   "action": "orientate",
>   "holder": "command_terminal",
>   "parameters": { "orientation": "south" }
> }
> ```
>
> Côté serveur, l'action n'aurait qu'à lire le paramètre :
>
> ```php
> class OrientateDaedalus extends AbstractAction
> {
>     protected function applyEffect(ActionResult $result): void
>     {
>         $daedalus = $this->player->getDaedalus();
>         $daedalus->setOrientation($this->parameters['orientation']);
>     }
> }
> ```
>
> Rien, dans cette version, n'est propre à l'orientation d'un vaisseau. En remplaçant un contrat précis par un dictionnaire libre, nous avons déplacé une responsabilité : le client peut envoyer n'importe quelle valeur, et c'est au serveur de décider si elle est acceptable. Or cette implémentation accepte tout.
>
> Revenons donc à `OrientateDaedalus`. En pratique, nous voudrons sûrement valider la valeur reçue :
>
> ```php
> class OrientateDaedalus extends AbstractAction
> {
>     protected function applyEffect(ActionResult $result): void
>     {
>         $chosenOrientation = $this->parameters['orientation'];
>         if (!$this->orientationIsValid($chosenOrientation)) {
>             throw new \LogicException("The {$chosenOrientation} orientation is not valid, you little hacker!");
>         }
>
>         $daedalus = $this->player->getDaedalus();
>         $daedalus->setOrientation($chosenOrientation);
>     }
>
>     private function orientationIsValid(string $orientation): bool
>     {
>         return isset(OrientationEnum::$availableOrientations[$orientation]);
>     }
> }
> ```
>
> C'est la règle générale que cette proposition rend incontournable : une donnée venant du client se [valide côté serveur](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html), quelle que soit la validation faite dans l'interface.
>
> [TK: dans quel cas un dictionnaire de paramètres libres serait-il une mauvaise idée ?]
>
> ## Références
>
> - OWASP, *Input Validation Cheat Sheet*, OWASP Cheat Sheet Series. https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html

Ce qui a changé :

- Chaîne conservée : idée → requête → version naïve → même classe validée.
- Grande idée reliée à la version naïve (un contrat souple déplace la validation vers le serveur), puis retour au cas (1c, 1d). Le lien vers l'OWASP reste discret, sans attribution dans la phrase (règle 6).
- « front-end » et « back-end » deviennent « client » et « serveur » ; une incise situe le projet.
- « imo » → « il me semble » ; « sûrement » conservé ; la blague reste dans le code.
- Code corrigé et signalé : `boolean` → `bool`, `protected` dans les deux versions.

---

## Paire 2 - Le rebase (règle 3)

### Source (2024-01-28, complétée le 6 juin 2025)

```
Je vois trois points qui peuvent poser problème :
- Rebaser la bonne branche, au bon endroit. Exemple : il faut lancer `git rebase develop` sur la branche cible, pour que la branche cible recupère les commits de `develop`
- Rebaser à partir d'une version non mise à jour de `develop`. Il est très fort possible que la version de `develop` sur ton PC ne soit pas à jour. Tu peux utiliser `git rebase origin/develop` pour toujours utiliser la version sur GitLab
- Mise à jour du 6 juin 2025 : Après avoir rebasé la branche, tenter d'envoyer ces changements avec une interface graphique (VSCode, PHPStorm..). Je ne saurais pas expliquer exactement ce qu'il fait dans ces cas, mais ça ne fonctionne pas et vous vous retrouvez avec un historique chaotique.
Il faut impérativement lancer `git push --force` (ou équivalents plus safe) dans un terminal.

En faisant :
git fetch
git checkout ma-branche
git rebase origin/develop
git push --force-if-includes --force-with-lease

J'ai jamais de problèmes
```

Un autre message (2025-06-06) : sur une branche avec beaucoup de commits, Charles a d'abord tout squashé avec `git rebase -i` « so there are less conflicts to handle ».

### À éviter : l'article assisté par IA

| Passage | Problème |
|---|---|
| « C'est sans doute l'une des commandes les plus puissantes de Git — mais aussi l'une des plus faciles à rater. » | Emphase avant le problème |
| « une branche avec plus de 200 commits », « des centaines de pull requests » | Faits absents de la source : à demander |
| « Le plus étonnant ? Le piège le plus courant n'est pas celui qu'on croit. » | Question de suspense, superlatif non attesté |
| « Or ces interfaces graphiques gèrent mal les pushs forcés […] recréer des commits en double. » | **Le plus grave** : « je ne saurais pas expliquer » devient un mécanisme assuré |
| « La méthode inratable » | La source dit « J'ai jamais de problèmes » |
| « Maintenir un historique clair est un signe de professionnalisme » | Slogan |
| « En résumé » + CTA « bénéficier de mon expertise » | Résumé mécanique, CTA sans lien avec l'argument |

### Transposition

> # Trois façons de rater un rebase (et la séquence que j'utilise)
>
> Un [rebase](https://git-scm.com/docs/git-rebase) rejoue les commits d'une branche par-dessus une autre, pour intégrer ses nouveautés sans commit de fusion. Les trois erreurs qui suivent concernent tout projet où les branches de fonctionnalité sont rebasées sur une branche principale : `develop` ici, souvent `main` ailleurs.
>
> Prenons une branche `ma-branche`, créée à partir de `develop`. Pendant que nous travaillons dessus, d'autres merge requests sont fusionnées dans `develop`, et nous voulons les intégrer avant la relecture. [TK: le cas réel qui a motivé l'article ?]
>
> ## Rebaser la bonne branche
>
> `git rebase develop` rejoue les commits de la branche *courante* par-dessus `develop`. La commande doit donc être lancée depuis `ma-branche`. Lancée depuis `develop`, elle n'a aucun effet utile.
>
> ## Rebaser sur une version à jour de `develop`
>
> Il est très possible que la copie locale de `develop` ne corresponde plus à celle du serveur. Il est plus sûr de rebaser sur `origin/develop`, une *branche de suivi distante* (*remote-tracking branch*) : la copie de l'état du serveur que le dernier `git fetch` a récupérée.
>
> ## Pousser depuis un terminal
>
> J'ai ajouté ce point en juin 2025. Lorsque l'on pousse une branche rebasée avec le bouton d'un IDE (VSCode, PHPStorm…), je ne saurais pas expliquer précisément ce qu'il se passe. Je constate seulement que cela ne fonctionne pas : l'historique devient chaotique. Il faut donc impérativement pousser depuis un terminal.
>
> ## La séquence que j'utilise
>
> ```bash
> git fetch
> git checkout ma-branche
> git rebase origin/develop
> git push --force-if-includes --force-with-lease
> ```
>
> D'après la [documentation de `git push`](https://git-scm.com/docs/git-push), `--force-with-lease` refuse la mise à jour si quelqu'un a poussé sur la branche distante depuis notre dernier `fetch`, et `--force-if-includes` exige en plus que l'état distant ait été intégré localement. Avec cette séquence, je n'ai jamais rencontré de problème.
>
> Lorsqu'une branche contient beaucoup de commits, il m'est arrivé de les regrouper d'abord en un seul avec un rebase interactif (`git rebase -i`), pour avoir moins de conflits à résoudre. [TK: à partir de quand ferais-tu cela ?]
>
> ## Références
>
> - *git-rebase*, documentation officielle de Git. https://git-scm.com/docs/git-rebase
> - *git-push*, documentation officielle de Git. https://git-scm.com/docs/git-push

Ce qui a changé :

- Portée générale dès l'ouverture ; instance ajoutée pour un lecteur qui n'a pas le problème en tête.
- Jargon défini : rebase, branche de suivi distante, options de `git push`, rebase interactif ; documentation liée.
- Incertitude sur l'IDE conservée mot pour mot, et **pas généralisée** : Charles ne sait pas l'expliquer (R3).
- Frontière tirée d'un autre message de Charles ; son seuil reste en `[TK]`.
- Limite de cette transposition : elle reste un cours. La version publiée a trouvé son pivot après entretien : « pendant près de trois ans, j'ai vu des contributeurs expérimentés obtenir un historique incohérent après un rebase sans comprendre pourquoi ». Ce vécu manquait à la matière ; aucune règle de transposition ne pouvait le produire.
- Ni slogan, ni résumé, ni CTA.
