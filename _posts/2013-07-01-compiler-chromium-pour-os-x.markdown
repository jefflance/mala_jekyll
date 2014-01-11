---
author: jeff
comments: true
date: 2013-07-01 04:35:24+00:00
layout: post
title: "Compiler Chromium pour OS X"
categories: Mac
tags: chromium google OSX
---

**Première étape : Récupérer les outils nécessaires.**
    
    # git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    # export PATH="$PATH":`pwd`/depot_tools


**En deux : générer les clé de l'API Google.**  
  
Voir pour cela la documentation [ici](https://sites.google.com/a/chromium.org/dev/developers/how-tos/api-keys).


**Trois : choisir les sources.**  
  
Elles sont situées là : [http://src.chromium.org/svn/releases/](http://src.chromium.org/svn/releases/).  
Un listing des versions se trouve là : [http://omahaproxy.appspot.com/](http://omahaproxy.appspot.com/)


**Étape quatre : configurer le client svn et synchroniser les sources.**

    # gclient config http://src.chromium.org/svn/releases/27.0.1453.116
  
> _On peut éditer le fichier .gclient pour éviter de récupérer des choses inutiles :_  

>    # cat .gclient   
>    "custom_deps": {
    "src/third_party/WebKit/LayoutTests": None,  
    "src/chrome/tools/test/reference_build/chrome": None,  
    "src/chrome_frame/tools/test/reference_build/chrome": None,  
    "src/chrome/tools/test/reference_build/chrome_linux": None,  
    "src/chrome/tools/test/reference_build/chrome_mac": None,  
    "src/third_party/hunspell_dictionaries": None,  
    }
  

**Et enfin cinq : on compile.**

    # cd chromium/src/chrome
    # xcodebuild -project chrome.xcodeproj -configuration Release -target chrome

Et voiloù !