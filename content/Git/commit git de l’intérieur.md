---
title: C’est quoi au final un commit dans git ?
description: 
tags:
  - git
draft: false
date: 2024-02-18
created: 2024-02-18T18:31
updated: 2024-02-19T22:29
---
J’ai toujours cru qu’un commit git c’était ce que j’en voyais dans les merge request Gitlab : un ensemble de différences que j’introduisais dans un code existant.

Et puis un jour j’ai commencé à me documenter sur git et ma vision de ce qu’est un commit a radicalement changé.

Ça vous dirait de partir explorer les entrailles du dossier `.git` avec moi pour essayer de découvrir ce que renferme vraiment un commit ? Alors c’est parti !

# Comment on fait pour créer un repository git ?

Déjà pour bien comprendre il faut savoir qu’un repository git n’a pas du tout besoin de remote pour fonctionner, tout peut marcher uniquement en local. 
Alors aujourd’hui pas de Gitlab ni de GitHub, juste un simple repository vide en local pour repartir de zéro.

Il suffit de créer un dossier et d’en faire un repository git avec la commande :
```shell
git init
```
 Voilà c’est tout, on a un repo git local fonctionnel. C’est aussi simple que cela !

Et du coup ça fait quoi un `git init` alors ?

Git a simplement créer un dossier `.git` à la racine de mon dossier et l’a initialisé avec une arborescence et les quelques fichiers dont il a besoin pour fonctionner.

On peut observer son contenu avec la commande :
```shell
tree -a
```

![[init-tree.png|400]]

Pour le moment on ne va pas détailler tous les fichiers et dossiers créés, on va plutôt essayer de voir ce qu’il va se passer si on crée notre premier commit.

#  Naissance d’un commit

Alors on y va, on crée nos premiers fichiers et on va les commiter : 

```Shell
echo "# My new great projet" > README.md
echo "CC-BY-SA-4.0" > LICENSE.md
git add README.md LICENSE.md
git commit -m "init README and LICENSE"
```

Voilà, ça y est, notre premier commit du projet est là, on peut vérifier qu’il a bien été créé avec la commande :
```shell
git log
```

![[first-git-log.png]]

**On peut voir plusieurs choses** :
- Qui a créé le commit (Author)
- Quand est-ce qu’il a été créé (Date)
- Le message du commit
- Et un ensemble de caractères incompréhensibles : il s’agit d’un hash unique que git a affecté à ce commit (plus exactement il s’agit d’un SHA-1)

Bon ok, vous allez me dire rien d’extraordinaire pour le moment... En effet. 

# Il est stocké où mon commit ?

Vous vous rappelez le dossier `.git` ? Qu’est-ce qui a changé dedans maintenant qu’on a créé notre premier commit ?

![[tree-after-one-commit.png]]

On a des nouveaux fichiers, 4 pour être précis. Tous dans le dossier `objects`. 

Si vous avez été attentif vous avez peut-être même remarqué qu’un des objets porte le même nom que le hash de notre commit. Plus précisément le nom du sous-dossier correspond aux 2 premiers caractères du hash et le nom du fichier contient les 38 autres.

On va donc s’intéresser à ce fichier en premier.
Il s’agit d’un fichier binaire que nous n’allons pas pouvoir lire simplement, on va devoir utiliser une commande git méconnue : `git cat-file`.

`git cat-file` Permet de nous indiquer de quel type d’objet il s’agit pour git (avec le flag -t) et également de nous afficher son contenu de manière compréhensible(avec le flag -p).

![[content/Git/commit git de l'intérieur - files/first-commit.png]]

Il s’agit donc bien de notre commit. Contrairement à la sortie de `git log` on a une nouvelle information cette fois-ci : une ligne qui commence par `tree` et un nouveau hash.

Tout ce qui possède un hash dans git est un objet, on va donc pouvoir l’étudier avec la commande `cat-file` :

![[first-tree-content.png]]

Cette fois il s’agit d’un objet de type `tree`, grosse surprise, et dans son contenu encore des hash mais cette fois de type `blob`. On retrouve également les noms de nos fichiers `LICENSE.md` et `README.md`.

Allez, on continue avec cat-file :

![[first-blob-content.png]]

Voilà, on a obtenu le contenu de nos fichiers et on a enfin fait le tour des 4 objets créés par notre premier commit.

Si on résume ça donne ça :

![[First-commit.excalidraw.png]]
*Chaque flèche nous montre qui possède une référence vers qui.*
# Une chaîne de commit

On continue? 
Il nous faudrait un deuxième commit pour comprendre un peu plus ce qu’il se passe.

```Shell
echo "This project only illustrates a git demonstration" >> README.md
echo "juste something for illustration" > aFile.txt
git add README.md aFile.txt
git commit -m "add a file and update README"
```

Ok donc cette fois on modifie notre fichier README, on crée un nouveau fichier et on commit le tout.

![[second-git-log.png]]

D’après vous, que va contenir ce second commit ? Un nouveau fichier et la nouvelle ligne de notre README ? 

On vérifie dans le `.git` maintenant que l’on sait comment faire ?

![[second-tree.png]]

On remarque déjà que tous nos objets précédents sont toujours présents. Si on regarde leur contenu on constate qu’ils sont identiques à la première fois où on les a inspectés.

C’est une des clés du fonctionnement de git :
**Tous les objets stockés par git sous un hash ne seront jamais modifiés, on dit qu’ils sont immuables.**

Maintenant, intéressons-nous aux nouveaux objets. On a 4 nouveaux fichiers dans le répertoire `objects`. Que contiennent ils ?

Commençons par notre nouveau commit :

![[second-commit-content.png]]

Cette fois notre commit est un peu différent du premier. Il intègre une information supplémentaire, un **parent**, avec le hash de notre premier commit. 

C’est grâce à cette information que git connaît l’ordre de nos commits et qu’il peut reconstruire l’historique de notre repository. Le premier commit est donc une exception car il ne possède aucun prédécesseur.

Que contient notre tree ? 

![[secont-tree-content.png]]

Le fichier `tree` contient une référence vers les deux fichiers de notre commit mais également la référence vers notre premier fichier non modifié (le fichier `LICENSE.md`).

![[final-files-content.png]]

On remarque également que notre deuxième blob du fichier README.md contient nos deux lignes et pas seulement celle que l’on a rajoutée.

Si on résume on se trouve dans la situation suivante :

![[second-commit.excalidraw.png]]

# En résumé 

En fait, chaque commit de git contient une référence vers l’ensemble des blobs (fichiers) présents dans notre repository au moment du commit et pas seulement les différences introduites par ce commit !

**Un commit, au final, c’est une capture de l’état de notre dossier au moment du commit et pas seulement un ensemble de différences.**

On constate quand même que, si un fichier ne change pas, git ne s’embête pas à le stocker une deuxième fois et qu’il conserve la référence de son premier hash. 

En effet, c’est le principe même d’un hash, fournir la même clé pour un fichier qui ne change pas. Git se fie donc au hash pour déterminer si le contenu d’un fichier a changé.


> [!NOTE] Ce qu’il faut retenir
> - Git conserve une copie de chaque version de tous les fichiers jamais commités
> - Tous les objets git sont identifiables par un SHA-1 unique
> - **Un commit n’est ni plus ni moins qu’une liste de hash de tous les fichiers présents dans le dossier de travail au moment du commit**

J’espère que cet article vous aura aidé à mieux appréhender comment git fonctionne en interne. 

Pour ma part, j’ai toujours trouvé que comprendre le fonctionnement interne d’un outil aide à mieux l’utiliser au quotidien. 

# Références 
- La page [Git Internals - Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) du livre [pro git](https://git-scm.com/book/en/v2)
- La presentation Devoxx [Savez-vous vraiment comment fonctionne git](https://m.youtube.com/watch?v=uA2WZCQP4EI)
- La documentation de la commande [cat-file](https://git-scm.com/docs/git-cat-file)
