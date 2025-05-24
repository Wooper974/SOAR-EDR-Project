# SOAR-EDR-Project
# Introduction
J'ai décidé de construire ce lab dans le but de m'initier au SOAR, un outil devenu un incontournable dans les infrastructures d'un service SOC.
De ce fait, l'infrastructure de ce lab est composé en premier lieu d'un poste informatique infecté par un malware et avec l'EDR LimaCharlie installé sur ce même ordinateur. L'EDR enverra une alerte directement sur Tines, la solution SOAR choisi dans mon cas. Tines va à son tour envoyer certaines informations de l'alerte sur Slack et par Email. Enfin, Tines donnera la possibilité à l'analyste d'isoler la machine qui est à l'initiative de l'alerte. Vous trouverez ci-dessous le schéma résumant ce que je viens d'expliquer.

# Installation de l'EDR LimaCharlie

 

