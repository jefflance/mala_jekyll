---
layout: post
title:  "Commandes de post-installation"
date:   2012-02-23 23:53:29
categories: FreeBSD portsnap
---

Àpres une install toute fraîche de FreeBSD :

1. Mettre à jour l'arbre des ports

        $ portsnap fetch extract

1. Récupérer l'arbre des sources

    On commence par faire une copie du fichier sup correspondant à notre release en l'adaptant au passage.  

        $ mkdir -p /usr/local/etc/cvsup
        $ awk '\!/^#/' /usr/share/examples/cvsup/standard-supfile | \
        sed 's/CHANGE_THIS/cvsup/' > /usr/local/etc/cvsup/srctree.supfile

    On récupère l'arbre des sources à l'aide de [cvsup][cvsup-doc] (j'utilise ici csup qui est une alternative codée en C de cvsup).

        $ csup -L 2 /usr/local/etc/cvsup/srctree.supfile

Note : on peut egalement récupérer l'arbre des ports avec cvsup. Pour cela il suffit d'utiliser le fichier /usr/share/examples/cvsup/ports-supfile

[cvsup-doc]: http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/cvsup.html
