---
title: "Open Source French Drone Identification"
date:
  created: 2020-05-25
  updated: 2020-05-25
draft: false
categories: 
    - Various
tags:
    - remoteID
    - linux
authors:
  - khancyr
---
Cher·e·s dronistes français, (This news is mostly for french people, you will find **english translation at the end of the post**)

Vous n'êtes pas sans savoir que l'Arrêté du 27 décembre 2019 définissant les caractéristiques techniques des dispositifs de signalement électronique et lumineux des aéronefs circulant sans personne à bord va rentrer en vigueur prochainement. Cela implique que tous les drones pesant 800 grammes ou plus au décollage doivent être équipés d’un dispositif permettant de signaler électroniquement sa présence.
<!-- more -->

Ce signalement électronique repose sur l'envoi de trames "beacon" avec un équipement Wifi en utilisant une structure et un protocole définis dans le texte de loi. La difficulté d'implémentation du signalement électronique vient de la technique d'envoi des trames.
Contrairement à certaines idées reçues, l'utilisation de la trame "beacon" ne nécessite pas de créer un point d'accès wifi sur lequel on doit se connecter pour récupérer les informations. La trame "beacon" est justement une trame qui ne nécessite pas d'appairage d'équipement Wifi. Typiquement, lorsque vous utilisez votre gestionnaire réseau pour détecter et afficher les points d'accès Wifi présents autour de vous, ce sont ces mêmes trames "beacon" qui s'affichent, indiquant les informations émises par les différents points d'accès Wifi autour de vous.
![Exemple_trame_beacon|690x388](upload://vlvmbkhJgzpPR9JghZ4vUCwYB4y.png)

La difficulté repose donc sur les connaissances nécessaire pour pouvoir faire transmettre cette fameuse trame par votre puce Wifi.
L'encodage des informations ne présente pas de réelles difficultés pour peu qu'on sache un minimum coder.

Il existe peu de solutions actuellement sur le marché, notamment pour les budgets modestes.
C'est pourquoi nous vous proposons une solution open source que j'ai réalisé avec le soutien d'Airbot Systems afin de vous permettre de faire vous même votre module de signalement électronique pour moins de 40€ sans savoir coder, ni souder !

Quatre dépôts ont été mis en ligne :
- une librairie C++ servant d'implémentation de référence. https://github.com/khancyr/droneID_FR
- un code d'exemple en python pour envoyer des trames d'identification à partir d'un ordinateur et un décodeur de trames. https://github.com/khancyr/signalement_drone_python
- un code d'exemple sur une carte bas coût avec module GNSS intégré. https://github.com/khancyr/TTGO_T_BEAM
- un exemple de boitier imprimable en 3D pour la carte TTGO T-BEAM. https://github.com/khancyr/TTGO_T_BEAM_STL

- Le pack complet est téléchargeable sur le site d'Airbot Systems : https://www.airbot-systems.com/free-downloads/


-------
Infos :
-------

La librairie C++ est un simple fichier .h qui permet de faire l'encodage des informations de la bonne façon.


Le code python est un exemple de code utilisable sous Linux pour générer et envoyer des trames d'identification avec un système MAVLink (comme le simulateur d'ArduPilot). Le code python est séparé en deux. Un script permet de récupérer les informations MAVLink et d'envoyer la trame beacon. Le second script scanne les trames wifi reçu par votre ordinateur pour décoder les trames de signalement électronique drone. Ce code n'utilise que des outils standard et fonctionne sur Rasberry Pi. Vous trouverez aussi dans la documentation de ce dépôt comment utiliser un logiciel pour voir toutes les trames beacons qui sont captées par votre ordinateur.


Enfin, le dernier dépôt propose la création d'un module indépendant et autonome permettant de faire votre propre signalement électronique. La carte utilisée en exemple est une carte TTGO T-Beam que vous trouverez facilement pour moins de 40€ chez votre vendeur favoris. Privilégiiez une version 1.0 ou 1.1 de la carte si possible pour plus de performance GPS. Cette carte est basée sur une puce Wifi ESP32, bien connu des communautés de développeurs et bricoleurs, et possède un GNSS ainsi qu'un support de batterie pour compléter le tout. Nous avons réalisé une documentation sur comment programmer la carte, et pour plus de facilité, le code a été réalisé avec le framework Arduino. Il ne vous reste qu'a changer le nom de point d'accès Wifi que va créer la carte et l'ID drone, avec celui que vous a fourni AlphaTango ! (La DGAC a précisé que ce serait bientôt disponible sur leur site)
Attention, si vous ne modifiez pas les valeurs par défauts vous aurez un réseau Wifi nommé "ILLEGAL_DRONE_AP" et un ID drone : "ILLEGAL_DRONE_APPELEZ_POLICE17" ....
Évidemment, ce code est portable à d'autres cartes basées sur l'ESP32. La carte TTGO T-BEAM a été choisie, car elle possède directement tout l'équipement pour être autonome pour le signalement électronique. Nous proposerons plus tard un autre exemple avec juste une puce ESP32 branché sur une carte type Pixhawk ou Cube avec ArduPilot. En combinant, les informations de l'exemple python et de l'ESP32 vous ne devriez pas avoir de mal à faire vous-même la modification pour avoir un module d'émission encore moins chère et/ou plus intégré dans votre drone.
Malheureusement, nous ne connaissons pas les protocoles utilisés par les autopilotes BetaFlight et autres et n'avons pas le temps de proposer d'implémentation d'exemple. N'hésitez pas à proposer d'autres exemples qui utilisent les protocoles LTM, MSP, etc. comme entrées de la librairie pour envoyer les bonnes données dans la trame beacon.

![banana_scale|370x500](upload://dsytiou9FTtjH8uaOu6qis6Q8vl.jpeg)

-----
F.A.Q:
- Est-ce légal : OUI !
- Est-ce que c'est certifié : OUI ! La loi ne prévoit aucun mécanisme de certification ou de validation des données envoyées. Cette implémentation est donc conforme. Elle a été testée avec le code que propose la Gendarmerie pour la détection des trames : https://github.com/GendarmerieNationale/ReceptionInfoDrone
- Est-ce que je dois payer pour utiliser ce code : NON ! Le code que nous proposons est en libre accès.
- Est-ce que je peux vendre des modules basés sur ce code : OUI ! Dans le respect de la licence GPL qui impose que vous proposiez à votre acheteur un accès au code utilisé.
- Est-ce que vous vendez des modules tout fait: NON ! Pour ma part (Pierre Kancir), je ne vends rien. Veuillez voir avec Airbot Systems ou d'autres entreprises pour avoir une carte clé en main.
- Est-ce que je peux utiliser une autre carte avec ce code : OUI et NON. Si c'est une carte avec une puce ESP32, alors vous n'aurez simplement qu'à changer la définition des ports de votre carte pour faire marcher le tout. Avec une autre puce, la librairie C++ sera fonctionnelle, mais vous devrez faire appel aux bonnes API de votre puce Wifi pour émettre la trame beacon.
- Qu'est-ce que je risque à voler avec l'ID par défaut ou sans signalement : veuillez relire le texte de loi, c'est marqué dedans.
  https://www.legifrance.gouv.fr/eli/arrete/2019/12/27/ECOI1934044A/jo/texte
- Est-ce que je suis obligé d'utiliser votre librairie : NON ! Vous avez un code référence qui fonctionne, vous pouvez créer votre propre librairie et outils ou utiliser ceux d'autres personnes/entreprises.
- Je n'ai jamais codé, est-ce que c'est dur à utiliser ? Normalement NON. Mais cela dépend de votre capacité à suivre des informations techniques. N'hésitez pas à poser des questions sur vos forums drones favoris, voir vous déplacer sur un terrain de modélisme ou Fablab pour qu'on vous montre comment faire !

Pierre Kancir (Hivebotics)
Julien Queffélec (Airbot Systems)

Edit : liens vers des logiciels dérivés fait par la communauté :

* Code pour ESP1 : https://discuss.ardupilot.org/t/open-source-french-drone-identification/56904/28
* Decodeur de balise avec ESP32 : https://discuss.ardupilot.org/t/open-source-french-drone-identification/56904/113
* Version ESP8266 avec interface Web : https://discuss.ardupilot.org/t/open-source-french-drone-identification/56904/245
* Balise Compatible MSP : https://discuss.ardupilot.org/t/open-source-french-drone-identification/56904/162
* Balise avec IHM web intégré : https://discuss.ardupilot.org/t/open-source-french-drone-identification/56904/383

========================
English version
========================

As you may be aware, french law will soon request that all UAVs weighing 800 grams or more at takeoff must be equipped with a device that allows to electronically signal its presence.

This electronic signalling is based on the sending of "beacon" frames with Wifi equipment using a structure and protocol defined in the text of the law. The difficulty in implementing electronic reporting lies in the technique of sending the frames.
Contrary to some preconceived ideas, the use of the "beacon" frame does not require the creation of a wifi access point to which one must connect to retrieve the information. The "beacon" frame is precisely a frame that does not require the pairing of Wifi equipment. Typically, when you use your network manager to detect and display the Wifi access points present around you, it is these same "beacon" frames that are displayed, indicating the information emitted by the different Wifi access points around you.
![Example_beacon_frame|690x388](upload://vlvmbkhJgzpPR9JghZ4vUCwYB4y.png)

The difficulty is therefore based on the knowledge necessary to be able to transmit this famous frame by your Wifi chip.
The encoding of the information does not present any real difficulties as long as you know how to encode it.

There are few solutions currently on the market, especially for modest budgets.
That's why we propose you an open source solution that I realized with the support of Airbot Systems in order to allow you to make yourself your electronic signalling module for less than 40€ without knowing how to code or solder!

Four repositories have been put online :
- a C++ library serving as a reference implementation. https://github.com/khancyr/droneID_FR
- a python example code for sending identification frames from a computer and a frame decoder. https://github.com/khancyr/signalement_drone_python
- an example code on a low cost board with integrated GNSS module. https://github.com/khancyr/TTGO_T_BEAM
- an example of a 3D printable case for the T-BEAM TTGO board. https://github.com/khancyr/TTGO_T_BEAM_STL

- The complete pack can be downloaded from the Airbot Systems website: https://www.airbot-systems.com/free-downloads/


-------
Info :
-------

The C++ library is a simple .h file that allows you to encode the information in the right way.


The python code is an example of code that can be used under Linux to generate and send identification frames with a MAVLink system (like the ArduPilot simulator). The python code is separated in two. A script allows to retrieve the MAVLink information and to send the beacon frame. The second script scans the wifi frames received by your computer to decode the drone electronic signaling frames. This code uses only standard tools and works on Rasberry Pi. You will also find in the documentation of this repository how to use a software to see all the beacon frames that are captured by your computer.

Finally, the last repository proposes the creation of an independent and autonomous module allowing you to make your own electronic report. The card used as an example is a TTGO T-Beam board that you can easily find for less than 40€ at your favorite vendor. Prefer a version 1.0 or 1.1 of the board if possible for more GPS performance. This card is based on a Wifi ESP32 chip, well known to the developer and do-it-yourself communities, and has a GNSS and battery support to complete the package. We made a documentation on how to program the card, and for ease of use, the code was made with the Arduino framework. All you have to do is to change the name of the Wifi access point that will be created by the card and the drone ID, with the one provided by AlphaTango (French official website for drone matter)! (The DGAC has specified that it will soon be available on their website).
Attention, if you do not modify the default values you will have a Wifi network named "ILLEGAL_DRONE_AP" and a drone ID: "ILLEGAL_DRONE_APPLEZ_POLICE17" .... (ILLEGAL_DRONE_CALL_POLICE911 in english)
Obviously, this code is portable to other ESP32 based boards. The TTGO T-BEAM was chosen because it has directly all the equipment to be autonomous for electronic reporting. We will later propose another example with just one ESP32 chip connected to a Pixhawk or Cube type flightcontroler with ArduPilot. By combining the information from the python example and the ESP32 you should have no problem to make the modification yourself to have an even cheaper and/or more integrated transmitter module in your UAV.
Unfortunately, we don't know the protocols used by BetaFlight and other autopilots and don't have time to propose an example implementation. Feel free to propose other examples that use LTM, MSP, etc. protocols as library entries to send the right data in the beacon frame.


-----
F.A.Q:
- Is it legal: YES!
- Is it certified : YES! The law does not provide any mechanism for certification or validation of the data sent. This implementation is therefore compliant. It has been tested with the code proposed by the Gendarmerie (like Police but outside cities) for the detection of frames : https://github.com/GendarmerieNationale/ReceptionInfoDrone
- Do I have to pay to use this code: NO! The code that we offer is freely available.
- Can I sell modules based on this code: YES! In compliance with the GPL license which requires that you offer your buyer access to the code used.
- Do you sell ready-made modules: NO ! For my part (Pierre Kancir), I don't sell anything. Please check with Airbot Systems or other companies to get a turnkey card.
- Can I use another board with this code: YES and NO. If it's a board with an ESP32 chip, then you will just have to change the port definition of your card to make it work. With another chip, the C++ library will be functional, but you'll have to use the right API of your Wifi chip to emit the beacon frame.
- What do I risk to steal with the default ID or without reporting: please read the law text again, it's explain inside.
  https://www.legifrance.gouv.fr/eli/arrete/2019/12/27/ECOI1934044A/jo/texte
- Do I have to use your library: NO! You have a reference code that works, you can create your own library and tools or use those of other people/companies.
- I've never coded before, is it hard to use? Normally NO. But it depends on your ability to follow technical information. Feel free to ask questions on your favorite drone forums, see you move around a modeling field or Fablab to be shown how to do it!

Pierre Kancir (Hivebotics)
Julien Queffélec (Airbot Systems)
