---
author: jeff
comments: true
date: 2013-03-15 15:36:04+00:00
layout: post
title: Récupérer les paroles de titre dans songbird
categories: '*nix'
tags: GeekTool shell songbird sqlite3
---


Je me suis demandé comment récupérer, via un shell, les paroles d'un titre lu par Songbird pour les afficher notamment sur le bureau via GeekTool.
Je pars du principe que Songbird est configuré pour télécharger à la lecture du titre les paroles correspondantes et les stocke dans sa base de données.

Les données sur les médias gérés par songbird sont dans la base de donnée `main@library.songbirdnest.com.db` de Songbird (elle se trouve dans le dossier de profil).  
Les paroles et infos sur les médias sont dans la table : `resource_properties`.

En-tête de la table :

    media_item_id | property_id | obj | obj_searchable | obj_sortable | obj_secondary_sortable

> `media_item_id` : id du média dans songbird (numéro basé sur le classement alphabétique dans songbird).  
Un média est donc identifié par un _id_ unique.

Chaque entrée correspond aux données que songbird possède sur un média.
Pour un média (un _id_) donné, on a donc plusieurs entrées identifiables par le champ `property_id`.


Considérons donc un média dont l'_id_ est `media_item_id`. Les paroles de ce média sont stockées dans le champ `obj` de l'entrée avec le champ `property_id` valant _27_.

Pour afficher les paroles d'un titre dont on connaît l'_id_, il suffit de taper la requête suivante :

    select obj from resource_properties where media_item_id = 10 and property_id = 27;

On trouve donc chaque info dans le champ `obj` de chaque entrée. Il s'agit alors simplement de choisir la bonne entrée.

Le titre du média en cours est stocké dans le fichier `songbird_nowplaying.txt`.

Dans un shell, on récupère l'_id_ du fichier avec :
    
    sqlite3 main@library.songbirdnest.com.db "select media_item_id from media_items where content_url like '%$(cat /Volumes/Datas/tmp/songbird_nowplaying.txt | sed 's/ /%20/g')%'";

Enfin, avec une requête imbriquée on récupère les paroles du fichier en lecture :
    
    sqlite3 main@library.songbirdnest.com.db "select obj from resource_properties where media_item_id = (select media_item_id from media_items where content_url like '%$(cat /Volumes/Datas/tmp/songbird_nowplaying.txt | sed 's/ /%20/g')%') and property_id = 27";
