# SOAR-EDR-Project
# Introduction
J'ai décidé de construire ce lab dans le but de m'initier au SOAR, un outil devenu un incontournable dans les infrastructures d'un service SOC.
De ce fait, l'infrastructure de ce lab est composé en premier lieu d'un poste informatique infecté par un Malware (credential stealer) et avec l'EDR LimaCharlie installé sur ce même ordinateur. L'EDR enverra une alerte directement sur Tines, la solution SOAR choisi dans mon cas. Tines va à son tour envoyer certaines informations de l'alerte sur Slack et par Email. Enfin, Tines donnera la possibilité à l'analyste d'isoler la machine qui est à l'initiative de l'alerte. Vous trouverez ci-dessous le schéma résumant ce que je viens d'expliquer.

# Installation de l'EDR LimaCharlie

Nous allons tout d'abord mettre en place l'EDR sur une machine windows. Après avoir créer un compte sur la plateforme LimaCharlie, on peut désormais créer une clé d'installation 

![installation-key](https://github.com/user-attachments/assets/204ce5a7-f65b-4c1e-8fd6-f0455e08b516)

Une fois cette chose faite, nous pouvons désormais installer l'exécutable de l'EDR, qui nous demandera de renseigner la clé précédemment créer.

SCREEN SUCCESS

A partir de là, on peut deja observer à travers la console LimaCharlie, les possibilités de supervision qui s'offre à nous, comme le moyen d'isoler la machine, voir les processus en cours, les logiciels configurés en autorun sur le poste, exécuter des commandes à distance et bien d'autres choses encore.

SCREEN SPLIT 4

# Éxécution et visualisation du Malware

J'ai choisi d'installer le malware LaZagne, qui permet de récupérer des identifiants et mot de passes stockées sur la machine en local.

Une fois le malware installé, je l'éxécute avec l'option "all" pour qu'on puisse facilement par la suite retrouver sa trace dans les logs de l'EDR.

SCREEN EXECUTION ALL

Dans l'onglet Timeline de LimaCharlie, on peut retrouver la trace d'éxécution de LaZagne, qui comprend tout un tas d'informations qui va nous êtres utile pour construire notre règle de détection et de réponse. Parmi ces informations, les plus pertinentes sont les champs suivants: 
   * COMMAND_LINE
   * FILE_PATH
   * HASH
SCREEN TIMELINE LAZAGNE

# Création de la règle de détection et de réponse.

Pour la règle de détection, on va devoir identifier tout ce qui peut concerner l'éxecution de LaZagne. Dans la timeline, on apprend que l'event "NEW_PROCESS" est celui choisit lorsque LaZagne se lance.
Ensuite, il faut également spéicifier le système d'exploitation, dans notre cas Windows.
Maintenant, si on regarde le champ COMMAND_LINE, LaZagne apparait systématiquement dans ce champ lorsqu'il est éxécuté.
Dans le champ FILE_PATH, LaZagne intervient toujours à la fin du chemin désigné.
Et enfin, le champ HASH, permet de retrouver la signature de l'éxecutable.

Ce qui nous donne au final cette règle de détection : 
SCREEN DETECT

Cette règle peut être taduite de la manière suivante : Le type d'event doit être NEW_PROCESS et détecté sur un endpoint Windows. La fin du champ FILE_PATH doit se terminer par "LaZagne.exe" OU le champ COMMAND_LINE doit contenir la valeur "LaZagne" OU le HASH du fichier doit être celui spécifié dans la valeur.

Il ne nous reste plus qu'à compléter la partie réponse de la règle, en indiquant que l'on souhaite utilisé l'action "report", ce qui va permettre d'envoyer une notification lorsque l'alerte est déclenchée.
SCREEN RESPONSE

Si je relance LaZagne sur la machine victime, je peux désormais retrouver dans l'onglet Detections de LimaCharlie, une alerte qui me notifie de l'éxécution de LaZagne.
SCREEN DETECTION RESULT.
