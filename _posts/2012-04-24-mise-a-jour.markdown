---
author: jeff
date: 2012-04-24 18:11:32+00:00
layout: post
title: "Mise à jour"
categories: FreeBSD
tags: csup FreeBSD make
---


Voici les différentes étapes pour mettre à jour une FreeBSD, à l'ancienne.
Cette méthode n'est plus vraiment à jour dans la mesure où il existe l'outil `freebsd-update`.  
  
On commence par récupérer (ou mettre à jour) l'arbre des sources.


**Avec csup : dépréciée**.  
Pour cela on prépare un fichier de configuration pour csup.

    # cat /usr/local/etc/cvsup/srctree.supfile
    *default host=cvsup.fr.FreeBSD.org
    *default base=/var/db
    *default prefix=/usr
    *default release=cvs tag=RELENG_9
    *default delete use-rel-suffix
    *default compress
    
    src-all

> *default host=cvsup.fr.FreeBSD.org : serveur à partir duquel on récupère les mises à jour.  
> *default base=/var/db : répertoire où se trouve la base de données des mises à jour.  
> *default prefix=/usr : les sources vont être déposées dans /usr/src (le répertoire src étant implicitement indiqué).  
> *default release=cvs tag=RELENG_9 : indique la branche de la distribution que l'on veut obtenir.  
> *default delete use-rel-suffix : on autorise csup à supprimer des fichiers.  
> *default compress : enclenche la compression gzip durant la connexion.  
> src-all : la collection que l'on veut mettre à jour, ici tout.

Puis on lance csup

    # csup -L 2 /usr/local/etc/cvsup/srctree.supfile


**Avec subversion : conseillée**.  

    # svn checkout svn+ssh://svn.freebsd.org/base/head /usr/src

> On peut de la même manière récupérer l'arbre des ports : `svn checkout svn+ssh://svn.freebsd.org/ports/head /usr/ports`, voire l'arborescence de la documentation :   `svn checkout svn+ssh://svn.freebsd.org/doc/head /usr/doc`


**Construire le "monde" et le noyau**.

    # cd /usr/src
    # rm -rf /usr/obj
    # make buildworld kernel KODIR=/boot/kernel.testing

> KODIR=/boot/kernel.testing
Notre nouveau noyau va s'appeler kernel.testing.  
Ceci va nous permettre de faire un teste de celui avec notre matériel avant de l'utiliser définitivement. En cas de problème on pourra rebooter sur l'ancien noyau.

On y va. On demande à rebooter sur le nouveau noyau.
    
    # nextboot -k kernel.testing
    # shutdown -r now

S'il n'y a pas d'erreurs, on renomme les noyaux.
    
    # cd /boot
    # mv kernel kernel.old
    # mv kernel.testing kernel


**Mettons à jour le système**.

La [documentation officielle](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/makeworld.html) recommande de redémarrer en mode single pour la procédure de màj du système mais cela peut être fait en coupant un maximum de services, on peut à ce moment laisser sshd en cas de besoin.  
Pour ma part, j'ai choisi le redémarrage à chaud :  
    
    # init 1

Enfin, on termine le travail.  
    
    # cd /usr/src
    # mergemaster -p
    # make installworld
    # mergemaster -iU
    # yes | make delete-old
    # shutdown -r now
    
That's all.
