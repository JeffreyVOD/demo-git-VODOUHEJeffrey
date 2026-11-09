# journal.md — Module CI/CD : Git, GitHub Actions & Docker

Auteur : Jeffrey — ESAIP Bachelor 3 IA
Dépôt de travail : `inventaire`

> Ce fichier regroupe les trois TP. La partie TP 01 correspond au livrable `poste.md` (recopiez-la dans ce fichier si votre enseignant veut deux fichiers distincts).
> Les empreintes (hash), versions et chemins ci-dessous sont des exemples cohérents entre eux : remplacez-les par les sorties réelles de votre machine avant de rendre.

---

# TP 01 — Installer & configurer Git (contenu de `poste.md`)

## Étape 2 — Prédiction et version réelle

- Version prédite : `git version 2.43.0`
- Version réelle (`git --version`) : `git version 2.47.1.windows.1`
- Les deux lignes diffèrent, je garde les deux : mon estimation venait de la version livrée par défaut sur les distributions de l'école, pas de celle que `winget` installe.

## Étape 3 — Terminal neuf

Le terminal ouvert avant l'installation ne trouvait pas `git` (`PATH` non rechargé). Correction : fermer toutes les fenêtres de terminal puis en ouvrir une neuve. `git --version` répond alors normalement.

## Étapes 4 et 5 — Identité et fichier global

Fichier de configuration global : `C:/Users/jeffr/.gitconfig`

## Étape 8 — Quelle valeur de `user.email` est utilisée ?

`git config --list --show-origin | grep user.email` affiche deux lignes :

```
file:C:/Users/jeffr/.gitconfig	user.email=jeffreyvodouhe@gmail.com
file:.git/config	user.email=moi@autre-domaine.fr
```

Git utilise `moi@autre-domaine.fr` (la dernière ligne), ce que confirme `git config user.email`.
**Règle :** le niveau le plus spécifique l'emporte : `--local` > `--global` > `--system`.

Après `git config --unset user.email` dans le dépôt, `git config user.email` renvoie à nouveau la valeur globale `jeffreyvodouhe@gmail.com`.

## Étape 10 — `core.autocrlf`

Réglage : `core.autocrlf true` (Windows).
Effet : Git convertit les fins de ligne LF en CRLF à l'écriture dans le répertoire de travail (checkout) et ramène les CRLF en LF au moment de l'indexation, donc le dépôt reste en LF alors que les fichiers sur le disque sont en CRLF.

## Sortie de `git config --global --list --show-origin`

```
file:C:/Users/jeffr/.gitconfig	user.name=Jeffrey VODOUHE
file:C:/Users/jeffr/.gitconfig	user.email=jeffreyvodouhe@gmail.com
file:C:/Users/jeffr/.gitconfig	init.defaultbranch=main
file:C:/Users/jeffr/.gitconfig	core.editor=code --wait
file:C:/Users/jeffr/.gitconfig	core.autocrlf=true
```

(Les deux alias `st` et `lg` s'y ajoutent à l'étape 18 du TP 03.)

---

# TP 02 — Comprendre les zones Git

## Section 1 — Un dépôt neuf

**Étape 1.** Avant tout commit, `git status` affiche `On branch main` puis `No commits yet`. Pour `src/`, Git n'affiche pas les fichiers mais le dossier entier : `src/` figure dans la liste des « Untracked files ».

**Étape 2.** `ls -a .git` : 7 entrées utiles (hors `.` et `..`) : `HEAD`, `config`, `description`, `hooks`, `info`, `objects`, `refs`. `HEAD` indique la branche courante, `objects` stocke le contenu. Le nombre varie légèrement selon la version de Git.

## Section 2 — Prédire puis vérifier (paires index / répertoire de travail)

| Étape | Commande | Paire prévue | Paire réelle | Commentaire |
|---|---|---|---|---|
| 3 | `git status` (départ) | untracked/untracked | untracked/untracked | `README.md`, `pom.xml` et `src/` sont tous non suivis. |
| 4 | `git add README.md` | staged/unmodified | staged/unmodified | `README.md` : « Changes to be committed : new file ». |
| 5 | modification de `README.md` sans `git add` | staged/unmodified ❌ | **staged/modified** | **Prédiction fausse, corrigée** : je pensais que la modification serait fusionnée dans l'index. |
| 6 | `git commit -m "Ajoute le README"` | unmodified/unmodified ❌ | **unmodified/modified** | Prédiction fausse, corrigée : le commit contient la version indexée, pas celle du disque. |
| 7 | `git add README.md` puis `git commit -m "Complète le README"` | unmodified/unmodified | unmodified/unmodified | `README.md` est propre. |

**Étape 5 — combien de fois `README.md` apparaît-il ?** Deux fois : dans « Changes to be committed » (`new file`, la version indexée) et dans « Changes not staged for commit » (`modified`, la version modifiée sur le disque). Un même fichier a un contenu dans l'index et un autre dans le répertoire de travail.

**Étape 6 — que reste-t-il ?** `README.md` en « Changes not staged for commit » (la modification faite après le `git add`), ainsi que `pom.xml` et `src/` toujours non suivis.

## Section 3 — La vue courte

**Étape 8.** Dépôt mis dans l'état demandé : `pom.xml` indexé, `README.md` indexé puis modifié, `src/App.java` jamais indexé. `git status -s` :

```
MM README.md
A  pom.xml
?? src/
```

**Étape 9 — Lecture des deux colonnes**

- **1ʳᵉ colonne** : l'index comparé au dernier commit (`HEAD`), c'est-à-dire ce qui est prêt à partir au prochain commit.
- **2ᵉ colonne** : le répertoire de travail comparé à l'index, c'est-à-dire ce qui est modifié mais pas encore indexé.

Codes de mes trois fichiers :

| Fichier | Code | Signification |
|---|---|---|
| `README.md` | `MM` | modifié dans l'index ET modifié à nouveau dans le répertoire de travail |
| `pom.xml` | `A ` | ajouté à l'index, identique sur le disque |
| `src/App.java` | `??` | jamais indexé (Git affiche `?? src/` pour le dossier) |

## Section 4 — Sortir de l'index

**Étape 10.** Commande donnée par `git status` : `git restore --staged <fichier>`. Après cette commande, le fichier se trouve dans le **répertoire de travail seulement** : les modifications sont conservées, elles sont simplement « modifiées, non indexées » (`MM` redevient ` M`, et un fichier ajouté pour la première fois redevient `??`).

J'ai ensuite fini le TP : indexé `README.md` et commité `pom.xml` et `src/App.java` pour que les trois fichiers soient versionnés et que `git status` soit propre (`nothing to commit, working tree clean`).

**Étape 11.** `git log --oneline` : 4 commits, le plus récent est `c60d1b8`.

## Sortie de `git log --oneline` (fin du TP 02)

```
c60d1b8 (HEAD -> main) Ajoute App.java
91fe473 Ajoute pom.xml
4f2a9c1 Complète le README
8b7d3e0 Ajoute le README
```

---

# TP 03 — Opérations courantes (local)

## Section 1 — Ce qui n'entre pas dans l'historique

**Étapes 1 et 2.** `build/sortie.bin` et `config.local` créés, `.gitignore` écrit :

```
build/
*.local
```

`git status` : les deux fichiers ont disparu de la liste. Commit « Ajoute .gitignore ».

**Étape 3 — `git add -f`.** `git add -f build/sortie.bin` passe, malgré `.gitignore`. Retrait de l'index avec `git rm --cached build/sortie.bin`.

> `.gitignore` empêche uniquement les fichiers **non suivis** d'apparaître dans `git status` et d'être ajoutés par un `git add` ordinaire ; il n'empêche pas un `git add -f` et ne touche pas un fichier déjà suivi.

**Étape 4 — dossier vide.** J'ai créé `logs/.gitkeep` et commité « Ajoute logs/.gitkeep ».
Git versionne des **fichiers** (le contenu et leur chemin), pas des dossiers : un dossier vide ne contient rien à indexer, donc il n'apparaît jamais dans l'historique. La convention `.gitkeep` (nom sans signification particulière pour Git) place un fichier vide dans le dossier pour que Git le suive.

**Étape 5 — `.gitattributes`.**

```
*.sh text eol=lf
```

Après création et commit de `script.sh`, `git ls-files --eol script.sh` affiche :

```
i/lf    w/lf    attr/text eol=lf    script.sh
```

- `i/lf` : fins de ligne du contenu dans l'**index** = LF.
- `w/lf` : fins de ligne dans le **working tree** (répertoire de travail) = LF.

Malgré `core.autocrlf true` (Windows), l'attribut `eol=lf` l'emporte pour `*.sh`.

## Section 2 — Committer, et corriger un commit

**Étape 6.** Trois commits successifs sur `src/App.java`, dont le dernier avec un message volontairement mauvais :

```
52b7c0e Ajoute la méthode main
d4e81a6 Ajoute la liste des produits
c91e4a7 wip
```

**Étape 7 — `git commit --amend`.** `git log --oneline` avant : `c91e4a7 wip`. Après : `3d02f8b Ajoute l'affichage du stock`.

Les deux empreintes du dernier commit : **`c91e4a7` → `3d02f8b`**.

**Que s'est-il passé ?** Un commit est immuable : son empreinte est calculée à partir de son contenu, de son parent et de son message. Changer le message produit donc un **nouveau commit** (nouvelle empreinte) qui remplace l'ancien au bout de la branche. L'ancien commit `c91e4a7` n'est pas modifié : il n'est plus atteignable depuis `main` (il reste un temps dans le `reflog`, puis sera supprimé par le ramasse-miettes). C'est une réécriture d'historique : à ne jamais faire sur un commit déjà publié.

**Étape 8.** `git log --oneline --graph --decorate --all` : historique linéaire, une seule branche `main` avec `HEAD`. `git diff HEAD~1 HEAD` montre les lignes ajoutées par le dernier commit dans `src/App.java`.

## Section 2 bis — Supprimer, déplacer, voir la différence

**Étape 9 — suppression avec `rm`.** `doc/brouillon.md` créé et commité. Après suppression à l'explorateur, `git status` indique `deleted: doc/brouillon.md` dans « Changes not staged for commit ». Pour que la suppression entre dans le prochain commit, il faut **l'indexer** : `git add doc/brouillon.md` (ou `git add -u`), puis commiter.

**Étape 10 — `git rm`.** `doc/notes.md` créé, commité, puis supprimé avec `git rm doc/notes.md` : le statut devient `D ` (suppression déjà indexée). `git rm` fait en une seule commande ce que fait `rm` + `git add` à l'étape 9 : il supprime le fichier du disque **et** enregistre la suppression dans l'index.

**Étape 11 — arrêter de suivre sans supprimer.** Rejeu du cas : `git add -f build/sortie.bin` puis commit (« Ajoute build/sortie.bin »).
Option : **`git rm --cached build/sortie.bin`**, puis commit (« Arrête de suivre build/sortie.bin »). `git status` : le fichier n'apparaît plus (non suivi, donc ignoré par `.gitignore`).
Ce que l'option laisse derrière elle : le **fichier sur le disque** reste intact, mais il **reste dans l'historique** (les anciens commits le contiennent toujours).

**Étape 12 — `git mv`.** `git mv src/App.java src/Inventaire.java`, puis `git status -s` :

```
R  src/App.java -> src/Inventaire.java
```

Git parle d'un **renommage** (`R`). Il ne stocke pas les renommages : il les détecte en comparant le contenu supprimé et le contenu ajouté (ici 100 % identique).

**Étape 13 — `git diff` et `git diff --staged`.** Après modification de `src/Inventaire.java` : `git diff` montre la modification. Après `git add`, `git diff` n'affiche **plus rien** (le répertoire de travail est identique à l'index).

- `git diff` compare le **répertoire de travail** à l'**index** (ce qui n'est pas encore indexé).
- `git diff --staged` compare l'**index** au **dernier commit** (ce qui partira au prochain commit).

La modification est ensuite commitée (« Ajoute le commentaire de classe »).

## Section 3 — Annuler, selon la zone

**Étape 14.** Modification non indexée de `src/Inventaire.java` annulée avec : `git restore src/Inventaire.java`.

**Étape 15.** Modification indexée, puis annulation de l'indexation seulement (modification conservée) : `git restore --staged src/Inventaire.java`.

> Indice (étapes 14/15) : `git restore <fichier>` seul ne défait **pas** une indexation (il restaure le répertoire de travail à partir de l'index) ; et `git restore --staged` ne touche **pas** au contenu du disque. Pour supprimer à la fois l'indexation et la modification, il faut les deux commandes, dans cet ordre : `--staged` puis `restore` simple.

**Étape 16 — Tableau des annulations**

| Je veux annuler… | Commande | Ce que je perds |
|---|---|---|
| une modification non indexée | `git restore <fichier>` | La modification, **définitivement** (elle n'a jamais été enregistrée par Git, elle est irrécupérable). |
| une indexation | `git restore --staged <fichier>` | Rien : le contenu reste dans le répertoire de travail, seul le passage dans l'index est annulé. |
| le message du dernier commit | `git commit --amend` | L'ancienne empreinte : le commit est remplacé par un nouveau (l'ancien reste quelque temps dans le `reflog`). À ne pas faire si le commit est déjà publié. |

## Section 4 — Mettre de côté, et s'outiller

**Étape 17 — stash.** Modification commencée, puis `git stash` : le répertoire de travail est propre (`nothing to commit, working tree clean`). `git stash list` :

```
stash@{0}: WIP on main: e5b0d84 Renomme App en Inventaire
```

`git stash pop` restaure la modification et supprime l'entrée de la pile. J'ai ensuite abandonné la modification avec `git restore` pour terminer avec un dépôt propre.

**Étape 18 — alias.**

```
git config --global alias.st "status -s"
git config --global alias.lg "log --oneline --graph --decorate --all"
```

Sortie de `git lg` :

```
* a3f9c21 (HEAD -> main) Ajoute le commentaire de classe
* e5b0d84 Renomme App en Inventaire
* 1c7a6f3 Arrête de suivre build/sortie.bin
* 9d2e4b7 Ajoute build/sortie.bin
* 6f81a0c Supprime doc/notes.md
* b27c5e9 Ajoute doc/notes.md
* 04d9e31 Supprime doc/brouillon.md
* 7ac3f58 Ajoute doc/brouillon.md
* 3d02f8b Ajoute l'affichage du stock
* d4e81a6 Ajoute la liste des produits
* 52b7c0e Ajoute la méthode main
* f0a6d19 Ajoute .gitattributes et script.sh
* 8e3b2c4 Ajoute logs/.gitkeep
* 2a95f7d Ajoute .gitignore
* c60d1b8 Ajoute App.java
* 91fe473 Ajoute pom.xml
* 4f2a9c1 Complète le README
* 8b7d3e0 Ajoute le README
```

## Vérification finale

- `git status` : propre (`nothing to commit, working tree clean`).
- `build/sortie.bin` n'est plus suivi : il reste sur le disque et n'apparaît pas dans `git status` (ignoré par `.gitignore`).
- Le dépôt contient `.gitignore`, `.gitattributes`, `logs/.gitkeep` et 18 commits (minimum demandé : 6).
- L'historique garde la trace du commit fautif « Ajoute build/sortie.bin » (`9d2e4b7`) : `git rm --cached` arrête le suivi à partir de maintenant, il n'efface pas le passé.


Je rajoute maintenant une troisième ligne pour la branche feature-a ???

# TP 05 — Git distant avec Github

### Compréhension de `origin/main` et de l'option `-u`

La commande `git remote -v` permet d'afficher les dépôts distants configurés ainsi que leurs adresses de récupération (*fetch*) et d'envoi (*push*).

La commande `git branch -vv` affiche les branches locales et leurs relations de suivi avec les branches distantes.

`origin/main` désigne la branche de suivi distant correspondant à la branche `main` du dépôt distant nommé `origin` (généralement le dépôt GitHub). Elle représente la dernière position connue localement de cette branche distante.

L'option `-u` (`--set-upstream`), utilisée avec `git push -u origin main`, permet de définir la branche distante `origin/main` comme branche de suivi de la branche locale `main`.

Cette association est mémorisée par Git. Il n'est donc pas nécessaire de répéter `-u` à chaque envoi : les commandes `git push` et `git pull` peuvent ensuite être utilisées directement.


09/11/2026 - 15:44 :  git pull permettra de récupérer les chagements distants et mettra à jour le dépôt local en fast-forward car aucun changement n'a été effectué ici


### Expérience Git Fetch

**Manipulation :** J'ai effectué une seconde modification directement sur GitHub, puis j'ai exécuté `git fetch` depuis mon terminal.

**Observation :** Le nouveau commit a été récupéré et apparaît dans l'historique sous la référence `origin/main`. Cependant, ma branche locale `main` n'a pas été modifiée.

**Conclusion :** `git fetch` récupère les nouveaux commits du dépôt distant et actualise les références distantes locales, sans intégrer ces modifications dans la branche de travail. À l'inverse, `git pull` effectue une récupération suivie d'une intégration dans la branche locale.



$ git push
To https://github.com/JeffreyVOD/demo-git-VODOUHEJeffrey.git
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/JeffreyVOD/demo-git-VODOUHEJeffrey.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
