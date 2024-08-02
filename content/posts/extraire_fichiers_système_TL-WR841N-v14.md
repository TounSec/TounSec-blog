---
title: "👾 Extraction des fichiers système d'un TL-WR841N v14 👾"
date: 2024-05-26
draft: false
category: ["Hardware"]
tags: ["Hardware", "UART", "TL-WR841N_v14"]
language: fr
---

![Global](/images_extraire_fichiers_systeme_TL-WR841N-v14/IMG_20240524_131601_edit_759675902427830.jpg)

# Contexte

Cet article commence par une exploration de l'analyse du matériel (hardware) et de la console série, avant de se concentrer sur la rétro-ingénierie du firmware qui sera traité dans un prochain article, le produit ciblé est un routeur **TP-Link N300**. Les spécifications techniques du routeur seront présentées dans la partie suivante.

Ce travail est conçu pour tous ceux intéressés par le domaine de l'analyse du matériel et du hardware hacking. Le processus complet de recherche, depuis la **collecte d'informations** jusqu'à l'**extraction du firmware**, en passant par une autre méthode d'extraction des fichiers systèmes par l'UART sera décrit en détail.

L'objectif principal de ce première article est d'extraire les fichiers systèmes depuis l'UART du routeur **TP-Link N300**,. La seconde partie sera conssacré à **l'extraction du firmware** par la **mémoire flash** et de sa **rétro ingénierie**.

---
# Méthodologie

*Ci-dessous, la méthodologie suivit:*
- **Collecte d'Information**: Cette partie consiste à **rassembler des informations** sur le produit ainsi que ses composants maîtres, notamment les puces.
- **Interraction avec la Console Série**:  Cette étape implique l'interaction avec la **console série** via un protocole de communication série appelé **UART**, permettant ainsi d'accéder au système du microcontrôleur (MCU).

---
# Collecte d'Information

#### Information directement lisible sur le produit

![Dos produit](/images_extraire_fichiers_systeme_TL-WR841N-v14/IMG_20240524_141554_edit_761125477372921.jpg)

*Au dos du produit, on peut retrouver différente information intéressant tel que:*
- **Le Modéle**: TL-WR841N
- **La Version**: 14.0
- **Le Numéro de Série**: 22413A1000745
- **La puissance requise pour alimenter le produit**: 
	- 9V 
	- Courant Continu (DC) 
	- 0.6A

*Ainsi que des informations alternatives par rapport à l'objectif visé:*
- **MAC**: 74-FE-CE-4D-3D-68
- **Mot de passe réseau par défaut**: 13261313
- **SSID** : TP-Link_3D68

#### Information sur les puces composants le MCU

![PCB global](/images_extraire_fichiers_systeme_TL-WR841N-v14/Untitled-2024-05-24-1534.png)

Dans un premier temps, il existe un site vraiment utile pour la collecte d'information [HERE](https://openwrt.org/toh/tp-link/tl-wr841nd?s[]=tp&s[]=link&s[]=tl&s[]=wrn841n). Il rassemble souvent beaucoup d'information très utile, sur le hardware, les versions supportées, le firmware, console série, ... .

*Pour vérifier les informations collecter sur les puces à partir du site mentionné ci-dessus, récupérer l'acronyme/nom du fabricant ainsi que les références de chaques puces:*
- **SDRAM**: ESMT M13S2561616A-5T
- **CPU**: MEDIATEK M17628NN
- **Flash**: cFeon QH32B-104HIP

En faisant une simple recherche sur un navigateur, on trouve les datasheets des différentes puces. On se rend aussi compte que la seule bonne référence à collecter par le biais de ce site est celle du CPU, ce qui montre l'importance de vérifier manuellement les informations afin de les valider.

Enfin, la recherche de la **console série** est souvent l'une des premières choses à faire. Dans le cas de ce produit et de cette version, cela est relativement simple car les pins des **points de test** (TP) sont référencés. Je vais expliquer dans la partie **UART** comment trouver la signification de chaque pin manuellement. On verra d'ailleurs qu'il y a une petite spécificité qui n'est pas mentionnée sur le site.

- **SDRAM**: ESMT M13S2561616A-5T
- **CPU**: MEDIATEK M17628NN
- **Architecture**: MIPS
- **CPU bit**: 32-bit
- **Flash**: cFeon QH32B-104HIP
- **Bootloader**: uboot

Dans notre cas, le **MicroController Unit** (MCU) est composée d'un **CPU**, d'une **SDRAM** pour la mémoire temporaire (mémoire volatile) et d'une mémoire **flash** pour la mémoire permanente (non volatile), ainsi que de sa **console série** pour interagir avec le MCU à partir d'un protocole de communication série appelé **UART**.

---
# UART

#### C'est quoi ?

> Universal Asynchronous Receiver/Transmitter (UART) est un circuit intégré qui permet la communication série asynchrone entre deux dispositifs. Il convertit les données parallèles en une séquence de bits sérielle pour l'envoi et inversement pour la réception. Le UART est essentiel pour la communication entre ordinateurs et périphériques série.

#### Comment le trouver ?

Comme précisé précédemment, dans le cas de ce produit, il existe seulement une série de **4 points de test** et ils sont référencés. Cependant, sur une PCB plus imposante, il est plus compliqué de les trouver, on peut baser ces recherches sur plusieurs informations.

*Chercher **4 points de test successifs**, trouver une **masse sur le matériel**, placer votre sonde négative (noir) sur la masse que vous avez trouvé comme point de référence, puis à l'aide de la sonde rouge tester la tension (V) de ces 4 pins, normalement vous devriez trouver ces valeurs dans le cadre de notre cible:*
- **VCC** ≅ 3.3V
- **GND** = 0.0V
- **RX** = 0.0V
- **TX** ≅ 3.2V

**VCC** fournit une alimentation de **3.3V** et **GND** étant la **terre**, les valeurs sont facilement interprétables, mais pourquoi **RX** et **TX** ont-elles ces valeurs?

**RX** est la pin qui reçoit, n'ayant pas d'interruption, il est naturel de trouver sa valeur à **0**. Pour **TX**, c'est la pin qui transmet, donc sa **tension est stimulée** c'est une configuration du microcontroller.

Pour valider les valeurs du multimétre (on peut faire ce processus dans le sens inverse), il suffit de **souder 4 pins headers**.

![UART pin header](/images_extraire_fichiers_systeme_TL-WR841N-v14/IMG_20240524_130315.jpg)

Puis de faire le branchement du boitier de l'analyseur logique (noté que j'ai relié GND du boitié vers une masse comme pour le multimétre).
![Branchement logic analyser](/images_extraire_fichiers_systeme_TL-WR841N-v14/IMG_20240525_235615_edit_795696595506708.jpg)

Puis ouvrir le logic2 de Saleae
![logic2](/images_extraire_fichiers_systeme_TL-WR841N-v14/Untitled-2024-05-26-0015.png)
Le résultat obtenu sur **TX** montre qu'il y a bien des données qui transitent. Pour les décoder, il suffit de trouver la bonne **baud rate**. Pour la déterminer, il suffit de prendre la largeur d'un **bit positif** et de la diviser par 1 (par exemple : bd = 1 ÷ 8333µs). Le résultat vous donnera une valeur approximative, et vous pouvez vous référer à la liste suivante pour trouver la valeur exacte. Dans notre cas, c'est **115200** (comme souvent).
- 1200
- 2400
- 4800
- 9600
- 19200
- 38400
- 57600
- 115200
- 230400
- 460800
- 921600

![baud rate 1](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_003353.png)
![baud rate 2](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_003438.png)

Faire un clique droit puis afficher les valeurs en ascii
![decode to ascii](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_003755.png)

On reçoit bien des données de **TX**, ce qui est ce que nous attendions, même si normalement nous sommes censés recevoir une donnée qui commence par le bootloader, dans notre cas u-boot et sa version. Voyons maintenant en interagissant avec la **console série**.

#### Interragir avec la console série

Utilisez n’importe quel **USB à UART**, je vais utiliser personnellement un **CH340G**.
![CH340G](/images_extraire_fichiers_systeme_TL-WR841N-v14/IMG_20240526_004552_edit_796356241812337.jpg)

Réalisé le branchement en respectant le schéma suivant:
- TX => RX
- RX => TX
- GND => GND
- 3.3V => VCC ou alimenter directement avec la l'alimentation dédiée du routeur (ce que je fais personnellement)

Ensuite listé les **périphériques USB**
![lsusb](/images_extraire_fichiers_systeme_TL-WR841N-v14/Untitled-2024-05-26-0014.png)

Pour interragir avec la **console série UART**:
```bash
screen /dev/tty<USB> 115200
ou
putty
ou
minicom
```

![console](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_011324.png)

On s'aperçoit rapidement qu'il n'est pas possible de rentrer des commandes, nous sommes en fait dans un shell en lecture seule. `CTRL+A` puis `:quit` pour sortir de la console avec `screen`.

#### Comment contrer cette protection d'écriture ?

Dans un premier temps, par logique, je me suis dit qu'il devait y avoir un élément qui bloque le trafic entre la pin **TX de mon CH340G** et la pin **RX du routeur**. En cherchant un peu, on se rend vite compte qu'il y a une **résistance de type SMD** plutôt suspect juste en face de la pin **RX du routeur** (R18). C'est une méthode assez courante comme **sécurité pour l'UART** pour bloquer l'écriture sur la console série, elle permet de couper la transmission émise de la pin **TX de l'USB**, la pin **RX de la PCB** reçoit bien les données mais lorsqu'elle les transmet au **CPU** pour l'interprétation, la **résistance les bloque**.

Avec un multimètre, on obtient une valeur de **1kΩ** sur cette résistance, ça pourrait donc être notre responsable.

Avant de faire des bêtises, effectuons une analyse logique selon le branchement suivant:
- GND => GND
- TX => RX
- CHAN 1 => TX 

Après avoir réalisé cet essais afin de confirmer que le problème survenu lors de la réception des données de la pin **RX du routeur** et non de la pin **TX de mon CH340G**, il n'y a pas de perte de donnée au niveau de la transmission, les bits sont donc bien transmis, le problème vient effectivement du routeur et donc très probablement de la **résistance SMD de 1kΩ**.

Et bien, pour vérifier une dernière fois avant d'enlever la résistance, j'ai décidé d'alimenter en **5V** un court instant depuis mon **CH340G** histoire de tester et de voir si je pouvais entrer des commandes.

Ça marche! Nous avons donc effectivement trouvé notre coupable. J’ai pensé que **déssouder la résistance** (R18) serait trop dangereux pour l'intégrité des autres étants donnés leurs espacements très faibles (2 autres résistances SMD très proches), mais avec une **pince de précision coudée**, ça a fait l'affaire. On aurait également pu faire un pont avec un fil à étain entre la pin RX du routeur et la sortie de la résistance sur le circuit qui bloque la transmission des bits envoyés par la pin **TX du CH340G**.

![root](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_041428.png)
On peut voir que nous avons un **shell root** par défaut **sans authentification au préalable**!

#### Retrouver des informations sur le firmware

On peut récupérer quelques infos sur le firmware.
![kernel version](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_042522.png)
![cpuinfo](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_042559.png)

On observe d'ailleurs que la **version du noyau linux** est très ancienne.

#### Extraction des fichiers systèmes

Pour commencer, voyons quel fichier système permet la lecture mais surtout l'écriture
![mount](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_133318.png)

*On peut voir que nous avons les permissions de lecture et d'écriture pour:*
- rootfs
- proc
- ramfs
- sys

En général, **rootfs**, **proc** et **sys** sont restreints au niveau de l'écriture pour des raisons de sécurité, malgré les permissions, il nous reste donc **ramfs** (/var). À savoir que comme son nom l'indique, le stockage est destiné à la **RAM**, cette mémoire étant volatille, lors du redémarrage du routeur, nos actions seront effacées.

Voyons les fonctions de **busybox** disponible sur notre routeur
![busybox](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_134114.png)
On remarque que busybox est limité au niveau de ses fonctionnalitées, mais nous avons **tftp** !

Avant de continuer assurez-vous que vous avez un accès au réseau du routeur soit par un pont ethernet entre votre routeur et votre ordinateur ou en sans fil. Installé **tftp** et **configuré son serveur** sur votre ordinateur, vous pouvez suivre ce guide [HERE](https://www.fosslinux.com/50694/install-tftp-server-debian.htm) (penser à configurer par rapport au réseau du routeur). Le repos suivant peut être utile, il rassemble des fichiers binaire utile compiler pour l'architecture MIPS [HERE](https://github.com/darkerego/mips-binaries/tree/master). Ici nous aurons besoin de `busybox-mipsel`  car le processeur prend en compte **MIPSel** donc en **little-endian** [HERE](https://busybox.net/downloads/binaries/1.21.1/).

Maintenant, rendez-vous dans **/var/tmp** puis télécharger **busybox-mipsel** à l'aide de **tftp**
![busybox-mipsel](/images_extraire_fichiers_systeme_TL-WR841N-v14/Screenshot_20240526_153637.png)

Donner les permissions au binaire et exécuter le, nous avons toutes les fonctionnalitées qui étaient normalement restreintes par le **busybox d'origine du routeur**.
```bash
chmod +x busybox-mipsel
./busybox-mipsel
```

Plus qu'à faire une **archive des fichier systèmes** et de les transférer sur notre vers notre serveur **tftp** !

```bash
./busybox-mipsel tar cvf ./fs.tar ../../
tftp -pr fs.tar -l fs.tar 192.168.0.100
```

---
# Conclusion

Nous avons atteint notre objectif initial, qui consistait à récupérer les **fichiers système du routeur via la console série** en utilisant le protocole de communication série **UART**. À l'avenir, nous prévoyons **d'extraire le firmware à partir de la mémoire flash**, qui communique avec le CPU grâce au protocole de communication série **SPI**, et de procéder à la **rétro ingénierie de ce firmware**.

Je souhaite également souligner que, malgré la possibilité d'effectuer beaucoup plus d'opérations via **l'UART**, notamment avec un **accès root non authentifié**, cela n'était pas dans notre objectif original.
