---
author: jeff
comments: true
date: 2013-10-22 09:16:18+00:00
layout: post
title: "Connexion tmux via ssh"
categories: '*nix'
tags: shell SSH TMux
---


La commande suivante permet de lancer une connexion ssh et d'ouvrir directement une session tmux s'il y en a une d'ouverte sur l'hôte distant, d'en créer une nouvelle ou alors d'ouvrir une console zsh (si tmux n'est pas présent) ou bash (si aucun des précédents ne marche) :
  
    ssh $* -t 'tmux a || tmux || /bin/zsh || /bin/bash'

