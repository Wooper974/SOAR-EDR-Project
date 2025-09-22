# SOAR-EDR-Project
# Introduction
J’ai décidé de construire ce lab dans le but de m’initier au SOAR, un outil devenu incontournable dans les infrastructures d’un service SOC.
De ce fait, l’infrastructure de ce lab est composée, en premier lieu, d’un poste informatique infecté par un malware (credential stealer), sur lequel est installé l’EDR LimaCharlie. L’EDR enverra une alerte directement vers Tines, la solution SOAR que j’ai choisie dans mon cas. Tines transmettra ensuite certaines informations de l’alerte sur Slack. Enfin, Tines offrira à l’analyste la possibilité d’isoler la machine à l’origine de l’alerte.

Vous trouverez ci-dessous le schéma résumant ce que je viens d’expliquer.

# Installation de l'EDR LimaCharlie

Nous allons tout d’abord mettre en place l’EDR sur une machine Windows. Après avoir créé un compte sur la plateforme LimaCharlie, on peut désormais créer une clé d’installation :

![installation-key](https://github.com/user-attachments/assets/204ce5a7-f65b-4c1e-8fd6-f0455e08b516)

Une fois cela fait, nous pouvons installer l’exécutable de l’EDR, qui nous demandera de renseigner la clé précédemment créée.

<img width="1422" height="136" alt="Success" src="https://github.com/user-attachments/assets/1cbe75e6-f611-4184-94c5-870b7f6ead63" />

À partir de là, on peut déjà observer, via la console LimaCharlie, les possibilités de supervision qui s’offrent à nous : le moyen d’isoler la machine, voire de voir les processus en cours, les logiciels configurés en autorun sur le poste, d’exécuter des commandes à distance, et bien d’autres choses encore :

SCREEN SPLIT 4

# Éxécution et visualisation du Malware

J’ai choisi d’installer le malware LaZagne, qui permet de récupérer des identifiants et mots de passe stockés localement sur la machine.
Une fois le malware installé, je l’exécute avec l’option « all » afin que l’on puisse facilement retrouver sa trace dans les logs de l’EDR :

<img width="1073" height="370" alt="EXECUTION ALL" src="https://github.com/user-attachments/assets/35d46930-cb35-4b12-bb9c-00f592fd67a8" />  

Dans l’onglet *Timeline* de LimaCharlie, on peut retrouver la trace d’exécution de LaZagne, qui comprend tout un tas d’informations qui vont nous être utiles pour construire notre règle de détection et de réponse. Parmi ces informations, les plus pertinentes sont les champs suivants : 
   * COMMAND_LINE
   * FILE_PATH
   * HASH
<img width="1558" height="306" alt="timeline lazagne" src="https://github.com/user-attachments/assets/7ea2183c-9c47-4a2c-a589-80385f952c7d" />


# Création de la règle de détection et de réponse

Pour la règle de détection, nous devons identifier tout ce qui concerne l’exécution de LaZagne. Dans la *Timeline*, on apprend que l’événement NEW_PROCESS est celui utilisé lorsque LaZagne se lance.
Ensuite, il faut également spécifier le système d’exploitation, dans notre cas Windows.
Si nous regardons le champ COMMAND_LINE, LaZagne apparaît systématiquement dans ce champ lorsqu’il est exécuté.
Dans le champ FILE_PATH, LaZagne figure toujours à la fin du chemin indiqué.
Enfin, le champ HASH permet de retrouver la signature de l’exécutable.

Ce qui nous donne au final cette règle de détection : 
![SCREEN DETECT](https://github.com/user-attachments/assets/7a5c588b-33ce-4425-a89e-53054ac8de89)

Cette règle peut être traduite de la manière suivante :
Le type d’événement doit être NEW_PROCESS et détecté sur un endpoint Windows. La fin du champ FILE_PATH doit se terminer par LaZagne.exe OU le champ COMMAND_LINE doit contenir la valeur LaZagne OU le HASH du fichier doit correspondre à celui spécifié dans la règle.

Il ne nous reste plus qu’à compléter la partie « réponse » de la règle, en indiquant que nous souhaitons utiliser l’action *report*, ce qui permettra d’envoyer une notification lorsque l’alerte est déclenchée.

![SCREEN RESPONSE](https://github.com/user-attachments/assets/31aa803a-bca9-4549-bfa6-8c180e2d3270)

![SCREEN DETECTION RESULT](https://github.com/user-attachments/assets/8041d6a7-2e58-4c84-962d-a810f2652a73)


Si je relance LaZagne sur la machine victime, je peux désormais retrouver dans l’onglet *Detections* de LimaCharlie, une alerte m’indiquant l’exécution de LaZagne.
![detetc detce](https://github.com/user-attachments/assets/b48ddf6c-ce47-4f1c-97a0-542b46a7414f)


# Initialisation de Tines et Slack

Tout d’abord, nous devons faire en sorte que les alertes générées par LimaCharlie soient réceptionnées par Tines. Nous allons dans un premier temps, créer un Webhook sur Tines :

![Tines webhook](https://github.com/user-attachments/assets/8618c6e7-e4cb-4642-9f76-f30ec9ede7d8)

L’URL du Webhook peut ensuite être renseignée dans l’onglet Outputs de LimaCharlie. Pour vérifier que la liaison est opérationnelle, nous allons exécuter à nouveau LaZagne sur la machine infectée et observer si l’alerte est bien transmise :

![tines lima succ](https://github.com/user-attachments/assets/fcc9818b-d75c-466b-b62a-7feb4f1e638a)

Nous pouvons également retrouver l’alerte sur l’interface web de Tines : 

![tines alert success](https://github.com/user-attachments/assets/efdfc1b7-a402-4820-ad62-27773a9afbf8)

Maintenant, Tines doit être en mesure d’envoyer l’alerte reçue vers notre canal Slack. Pour commencer, nous allons simplement générer un message de test afin de confirmer que l’envoi de messages depuis Tines fonctionne correctement.
Tines offre la possibilité d’utiliser des *Templates* pour plusieurs solutions, dont Slack. Une fois la *template* Slack sélectionné, nous choisissons le Build « Send a message » et indiquons, dans les paramètres de ce build, l’ID du canal Slack ainsi qu’un message de test : 

![tines canad id msg test](https://github.com/user-attachments/assets/accd99a2-f068-4888-ba7c-da55b611bc85)

En lançant ce test, nous pouvons confirmer la réception du message de test sur notre canal Slack :

![Slack msg test](https://github.com/user-attachments/assets/5c842607-1170-4c0d-8fb3-fd4ec3d000fb)


# Création du Playbook

La prochaine étape consiste cette fois-ci, à envoyer de réelles informations issues de l’alerte reçue par Tines sur notre canal Slack.
Il convient d’abord de rappeler les champs qui nous intéressent : 
   * Nom de l'alerte 
   * Horaire
   * Nom de la machine
   * Adresse IP de la machine
   * File Path
   * Command Line
   * Utilisateur de la machine
   * Lien LimaCharlie

Il faut dorénavant retrouver ces champs dans l’alerte reçue par Tines et les renseigner dans le message que l’on souhaite envoyer à Slack : 

![tines champ msg](https://github.com/user-attachments/assets/5aafe25c-6c11-4d64-a1b0-30db32853273)

Si nous relançons une alerte, nous pouvons désormais observer sur notre canal Slack les informations remontées, essentielles pour qu’un analyste comprenne au mieux la situation : 

![slack msg alert](https://github.com/user-attachments/assets/67fe99f8-ca2f-498a-b186-79fce6dbf15e)

Ensuite, nous allons créer une page sur Tines pour donner la possibilité à l’analyste d’isoler la machine : 

![page tines](https://github.com/user-attachments/assets/1a341a4b-0218-4354-acb4-d5d110945ac3)

Une fois la page créée, nous nous intéressons au premier cas : lorsque l’analyste décide de ne pas isoler la machine.
On va de ce fait créer un *Trigger* lié à notre page, en veillant à ce que ce *Trigger* corresponde au champ souhaité (False dans notre cas) : 

![trigger false](https://github.com/user-attachments/assets/e2a95dfc-e19e-44a5-bda3-438f10a50829)

Le *Trigger* est ensuite lié à un autre message slack qui enverra l'information sur notre canal : 

![slack trigger 1](https://github.com/user-attachments/assets/863bc59c-ff92-4112-8e23-1d17aa64782e)
![slack trigger 2](https://github.com/user-attachments/assets/cb0fd69d-ac26-400d-a8fe-8056c0e394e1)

Passons maintenant au cas où l’analyste décide d’isoler la machine. Après avoir mis en place un nouveau *Trigger* pour le cas True, nous allons utiliser la template LimaCharlie pour Tines.
L’un des Builds de ce template permet d’isoler directement l’endpoint en fonction du SID qu’il reçoit. 

Avant d’appliquer ce Build, il faut relier LimaCharlie à Tines. Pour cela, nous allons créer un nouveau credential nommé « LimaCharlie » sur Tines, qui contiendra l’API de mon organisation sur LimaCharlie : 

![screen api tines lima](https://github.com/user-attachments/assets/e961a854-ce5c-4ff1-89be-091d6cd75772)

Revenons au Build d’isolation, sur lequel il ne faut pas oublier de renseigner le chemin exact de l’emplacement du SID lorsqu’une alerte est remontée sur Tines : 

![screen build isolate](https://github.com/user-attachments/assets/420551b6-559f-4c8e-880c-ced2a5a50569)

Il ne reste plus qu’à ajouter, à la fin, un nouveau message Slack pour signaler dans notre canal que la machine a été isolée : 

![screen slack isolate](https://github.com/user-attachments/assets/3418e821-7ba5-4e34-94c4-1505048bdb70)

Si nous lançons une simulation où l’analyste décide d’appuyer sur l’option « Oui », nous pouvons observer dans les détails de l’endpoint sur LimaCharlie, qu’il est désormais isolé du réseau : 

![final isolation limacharlie](https://github.com/user-attachments/assets/e76e761c-0ae4-49ea-875e-d47e258a5b91)

Nous obtenons à la fin cette arborescence dans Tines : 

![screen arbo tines](https://github.com/user-attachments/assets/6f79b457-c2de-4bcf-8fda-84ead51e9ce8)


En résumé, suite à la réception d'une alerte, Tines va : 
   * Envoyer un message contenant des informations de l'alerte sur Slack.
   * Créer une page résumant les informations et donnant la possibilité à l’analyste d’isoler la machine.
   
   * Selon le choix de l'analyste :
     * Ne pas isoler la machine et envoyer une notification sur Slack.
     * Isoler la machine et envoyer une notification sur Slack.















