---
title: "🛵 Basique Race Condition 🛵"
date: 2024-08-02
draft: false
category: "Exploitation"
tags: ["Exploitation", "Pwn", "Basic"]
language: fr
---

![INIT](/images_basic_race_conditions/fotor-ai-2024080214383.jpg)

# Introduction

Les systèmes informatiques modernes reposent sur des architectures complexes où la concurrence et le parallélisme sont omniprésents. Dans ce contexte, les vulnérabilités de type **Race Condition** émergent comme un défi significatif pour la sécurité des systèmes et des logiciels.

_Cet article vise à examiner la nature de ces vulnérabilités, leurs implications pour la sécurité, et les stratégies de mitigation actuelles._

# Définition et Mécanisme

> Une **Race Condition**, ou condition de concurrence, se manifeste lorsque le comportement d'un système dépend de la séquence ou du timing d'événements incontrôlables. Dans le contexte de la sécurité informatique, ces vulnérabilités surviennent généralement dans des environnements **multi-threads ou multi-processus** où l'accès aux ressources partagées n'est pas correctement synchronisé.

# Implications pour un SI

_Les conséquences des **Race Conditions** peuvent être graves:_
1. Élévation de privilèges: Un attaquant peut exploiter le délai entre la vérification et l'utilisation pour modifier les conditions du système.
2. Corruption de données: Des accès concurrents non synchronisés peuvent conduire à des états de données incohérents.
3. Déni de service: L'exploitation de ce type de vulnérabilité peut provoquer des blocages o udes crashs système.

# Méthodologie d'Analyse

Pour étudier ce type de vulnérabilité, une approche combinant analyse statique de code et tests dynamiques est employé. L'analyse statique permet d'identifier les points potieltiels de **Race Condition**, tandis que les tests dynamiques permettent de confirmé leur exploitabilité.

# Cas d'Etude

Certaines études, révélent que les **Race Conditions** sont particulèrement prévalentes dans les **systèmes d'exploitation**, les **bases de données** et les **applications web** à haute concurrence. Les secteurs les plus touchés seraient les **services financiers**, les **infrastructures critiques** et les **plateformes de commerce électronique**.

Un cas d'étude notable concerne une vulnérabilité découverte dans un système de paiement en ligne. L'exploitation d'une **Race Condition** permettait à un attaquant d'initier plusieurs transactions simultanées, conduisant à une double dépense et à des pertes financières signficatives.

# Mitigation

_Plusieurs approches ont été identifiées pour atténuer les risques liés à ces vulnérabilités:_
- Utilisation de primitives de synchronisation: Mutex, sémaphores et verrous pour contrôler l'accès aux ressources partagées.
- Programmation sans été: Réduction de la dépendance aux états partagés.
- Transactions atomiques: Garantir que les opérations complexes sont exécutées de manière indivisible.

# Exemple

#### Proof:

Considérons l'exemple suivant:
```c
if (access("file", W_OK) != 0) {
    exit(1);
}

fd = open("file", O_WRONLY);
write(fd, buffer, sizeof(buffer));
```

Ce code vérifie d'abord les permissions d'écriture sur un fichier avant de l'ouvrir et d'y écrire. Cependant, un intervalle de temps existe entre la vérification et l'ouverture du fichier, créant une fenêtre pour une exploitation.

#### Mitigation:
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    int fd;
    char buffer[] = "Hello, World!";

    fd = open("file", O_WRONLY);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    if (write(fd, buffer, sizeof(buffer)) == -1) {
        perror("write");
        close(fd);
        exit(1)
    }

    close(fd);
    return 0;
}
```

_Bonnes pratiques:_
- Vérification des erreurs: Toujours vérifier les erreurs lors de l'ouverture et de l'écriture des fichiers.
- Éviter les Race Conditions: Ne pas séparer les vérifications d'accès et les opérations sur les fichiers.

# Bibliographie

1. Collin, J., Borrett, M., & Finkelstein, A. (2016). Formal analysis of concurrent Java systems. Software Engineering, IEEE Transactions on, 42(11), 1080-1094.
2. Wang, R., Wang, X., Zhang, K., & Li, Z. (2019). Exploiting race conditions in payment systems. In Proceedings of the 2019 ACM SIGSAC Conference on Computer and Communications Security (pp. 2617-2631).
3. Zheng, Y., Bowers, K. D., & Popa, R. A. (2015). TACHYON: Fast and secure payment processing. In Proceedings of the 22nd ACM SIGSAC Conference on Computer and Communications Security (pp. 1446-1457).
