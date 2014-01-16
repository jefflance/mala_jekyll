---
author: jeff
comments: true
date: 2013-03-15 13:03:04+00:00
layout: post
title: "Affichage de la météo sur le bureau"
categories: Mac
tags: GeekTool OSX Python
---

Voici un script Python pour récupérer les données météo et pour les afficher sur le bureau via GeekTool.

    #! /usr/bin/python
    # -*- coding: utf-8 -*-
    """
    Author: C. Nichols <mohawke@gmail.com>
    Site: http://www.darkartistry.com
    
    Weather script to be used with Geektool,
    or whatever you like...
    Ref: http://developer.yahoo.com/weather/
    """
    import subprocess,StringIO
    import urllib
    from xml.dom import minidom
    import re
    import shutil
    
    def get_attribute(html, attr):
    res = re.search(r'%s="(.*?)"' % attr, html)
    return res.group(1)
    
    def curl(url):
    c = subprocess.Popen(["curl", url], stdout=subprocess.PIPE)
    (out,err) = c.communicate()
    return out
    
    def get_radar(image_out):
    image_url = "http://images.myforecast.com/images/cw/radar/northeast/northeast_anim.gif"
    result = curl(image_url)
    if result:
    open(image_out,'wb').write(result)
    else:
    print 'unable to retrieve image file.'
    
    def weather_for_zip(zip_code):
    WEATHER_URL = 'http://xml.weather.yahoo.com/forecastrss?p=%s'
    WEATHER_NS = 'http://xml.weather.yahoo.com/ns/rss/1.0'
    wurl = WEATHER_URL % zip_code
    
    results = curl(wurl)
    weather_icon = get_attribute(results, 'src')
    icon = weather_icon.split('/')[-1]
    
    dom = minidom.parseString(results)
    forecasts = []
    for node in dom.getElementsByTagNameNS(WEATHER_NS, 'forecast'):
    forecasts.append({
    'date': node.getAttribute('date'),
    'low': node.getAttribute('low'),
    'high': node.getAttribute('high'),
    'condition': node.getAttribute('text')
    })
    ycondition = dom.getElementsByTagNameNS(WEATHER_NS, 'condition')[0]
    return {
    'current_condition': ycondition.getAttribute('text'),
    'current_temp': ycondition.getAttribute('temp'),
    'forecasts': forecasts,
    'title': dom.getElementsByTagName('title')[0].firstChild.data,
    'icon': icon
    }
    '''
    You can now simply build your output from a simple key/val dictionary
    including the weather icon buy pointing to your icon set or change
    to download from Yahoo. Also, grab the radar if you want one.
    
    To run in Geektool, open shell and add: python /path/to/your/weather.py
    
    Don't forget the change the paths to the image files below.
    '''
    try:
    myWeather = weather_for_zip("43230")
    forcasts = []
    for i in myWeather['forecasts']:
    forcasts.append(i['date']+': '+i['condition']+'\n')
    
    message = u"%s°F %s\n%s" %(myWeather['current_temp'],
    myWeather['current_condition'],
    forcasts[-1])
    print message.encode('utf-8')
    
    # Copy yahoo weather icon to weather...
    localIcon = "/Users/mohawke/Geektool/Symbols/%s.png" %myWeather['icon'].split('.')[0]
    replaceIcon = "/Users/mohawke/Geektool/weather.png"
    
    shutil.copyfile(localIcon,replaceIcon)
    
    except Exception, error:
    print "Unable to get weather data"
    #print error


Source : [http://www.darkartistry.com](http://www.darkartistry.com)
