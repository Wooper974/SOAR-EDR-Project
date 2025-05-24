# SOAR-EDR-Project
# Introduction
J'ai décidé de construire ce lab dans le but de m'initier au SOAR, un outil devenu un incontournable dans les infrastructures d'un service SOC.
De ce fait, l'infrastructure de ce lab est composé en premier lieu d'un poste informatique infecté par un malware et avec l'EDR LimaCharlie installé sur ce même ordinateur. L'EDR enverra une alerte directement sur Tines, la solution SOAR choisi dans mon cas. Tines va à son tour envoyer certaines informations de l'alerte sur Slack et par Email. Enfin, Tines donnera la possibilité à l'analyste d'isoler la machine qui est à l'initiative de l'alerte. Vous trouverez ci-dessous le schéma résumant ce que je viens d'expliquer.

# Installation de l'EDR LimaCharlie

Nous allons tout d'abord mettre en place l'EDR sur une machine windows. Après avoir créer un compte sur la plateforme LimaCharlie, on peut désormais créer une clé d'installation 

![installation-key](https://github.com/user-attachments/assets/204ce5a7-f65b-4c1e-8fd6-f0455e08b516)

Une fois cette chose faite, nous pouvons désormais installer l'exécutable de l'EDR, qui nous demandera de renseigner la clé précédemment créer.

SCREEN SUCCESS

A partir de là, on peut deja observer à travers la console LimaCharlie, les possibilités de supervision qui s'offre à nous, comme le moyen d'isoler la machine, voir les processus en cours, les logiciels configurés en autorun sur le poste, exécuter des commandes à distance et bien d'autres choses encore.

SCREEN SPLIT 4

 

