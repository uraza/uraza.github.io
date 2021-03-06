---
layout: post
title: "Sauvegarde automatique sur WD My Book Live sous Windows"
description: "Une marche à suivre détaillée sous Windows pour mettre en place des sauvegardes automatiques de vos données sur un disque réseau WD My Book Live."
category: mybooklive
redirect_from:
  - /post/sauvegarde-my-book-live-windows/
  - /sauvegarde-my-book-live-windows/
---

Il y a quelques temps, je vous parlais de la mise en place de [sauvegarde automatique sur un disque réseau WD My Book Live]({% post_url 2013-10-16-sauvegarde-my-book-live-ubuntu %} "Tutoriel sur la sauvegarde sur My Book Live sous Ubuntu") sous Unix.

Dans cet article, je vous propose de faire la même chose sous Windows.

## Sauvegarde des données sur My Book Live avec robocopy

[robocopy](http://fr.wikipedia.org/wiki/Robocopy "Page Wikipedia sur robocopy") est une commande de réplication de contenu disponible sous Windows.

Nous allons utiliser cette commande dans un fichier batch que je nommerai *sync\_data\_nas.bat* dans cet article :

```sh
rem Ouverture d'une connexion sur le disque
net use \\mybooklive\jeremy /user:jeremy MOT_DE_PASSE

rem Création d'une copie de D: vers le My Book Live en ignorant certains répertoires
rem Les fichiers qui n'existent plus sur le répertoire local seront aussi supprimés du disque réseau
robocopy D:\ \\mybooklive\jeremy\ /MIR /Z /XD "Desktop" "Programmes" "Downloads" ".*" "$*" "System Volume Information"

rem Fermeture de la connexion
net use \\mybooklive\jeremy /delete
```

On commence par ouvrir une connexion vers le disque avec *net use* : cette commande reçoit le chemin vers le répertoire destination sur le My Book Live (*\\\\mybooklive\\jeremy* dans mon cas) ainsi que mes identifiants (nom d'utilisateur et mot de passe utilisés pour se connecter au disque réseau, dans le cas d'un répertoire protégé).

Ensuite on lance robocopy de *D:\\* (mon disque de données) vers *\\\\mybooklive\\jeremy*. Les options utilisées sont les suivantes :

* /MIR => Copie miroir, tout ce qui n'est plus présent dans la source sera aussi supprimé dans la destination.
* /Z => La copie recommence en cas d'échec.
* /XD => Exclut un ensemble de fichiers / répertoires : dans mon cas, je ne copie pas les répertoires "Desktop", "Programmes", "Downloads" et "System Volume Information" ainsi que tous ceux dont le nom commence par un point ou un dollar.

D'autres options existent, je vous invite à consulter la documentation de l'outil au besoin.

Le script est prêt, je vous invite à le mettre à un endroit fixe : sur ma machine, je le mets dans le répertoire *D:\scripts*.

## Mise en place d'une tâche planifiée

Plus qu'à mettre en place une tâche planifiée (l'équivalent Windows des tâches Cron) pour exécuter notre script de sauvegarde périodiquement.

L'outil de planification de tâches est disponible dans Panneau de configuration > Outils d'administration > Planificateur de tâches.

Il faut alors "Créer une tâche de base" (voir menu de droite).

Un assistant vous permettra de définir facilement votre tâche planifiée :

* Sur le premier écran, il suffit de nommer la tâche : je l’appellerai "SyncDataNAS".
* Ensuite, il est possible de choisir la fréquence de la tâche : dans mon cas, je choisis de l'exécuter "Tous les jours".
* Il m'est alors demandé de choisir une date de début et une heure d'exécution : 20h30, ce sera parfait, l'ordinateur est généralement allumé et je suis rarement devant à ce moment, ça ne gênera pas mon utilisation.
* Puis il faut sélectionner l'action : il s'agit de "Démarrer un programme", je pointe alors vers "D:\scripts\sync\_data\_nas.bat.", pas besoin d'argument.

Une fois la tâche créée, vous pouvez la voir en cliquant sur "Bibliothèque du Planificateur de tâches dans le menu de gauche". Plus tard, vous pourrez revenir à cet endroit pour voir si tout se passe bien : sur l'image ci-dessous, on peut voir que la dernière exécution a réussi.

![Liste des tâches planifiées Windows pour synchronisation My Book Live](/img/uploads/tache_planifiee_liste.png)

Et voilà ! Votre My Book Live recevra maintenant des sauvegardes régulières de vos données !
