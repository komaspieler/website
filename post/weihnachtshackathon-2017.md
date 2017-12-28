---
title: Weihnachtshackathon 2017
subtitle: WS2812B (NeoPixel) Implementierung auf einem RaspberryPi 3B
date: 2017-12-28
draft: true
---

Dieses Jahr haben meine Freundin und ich Weihnachten mit einem Hackathon verbracht - das erste Weihnachten ohne Familienbesuch.
Einerseits war das sehr ungewohnt, andererseits hat es uns erlaubt, an einem Projekt zu basteln, das uns beide interessiert und
an dem wir Spass haben.

So haben wir uns vorgenommen, das WS2812B Protokoll zu implementieren um eine RGB-LED-Leiste ansteuern zu koennen. Das Geraet der Wahl,
um schnelle Ergebnisse zu erzielen: ein RaspberryPi 3B.

> Note: Alle Ergebnisse unseres Hackathons sind in einem [github-Repository](https://github.com/komaspieler/ws2812-hackathon) dokumentiert!

Von Freunden aus dem Verein [Breadboarder e.V.](https://www.breadboarder.de) war ich auf die Leisten aufmerksam geworden und hatte schon einige Tips bekommen.
So erscheinen die Timing-Vorgaben aus dem [WS2812B Datenblatt](https://github.com/komaspieler/ws2812-hackathon/blob/master/datasheets/WS2812B%20Datasheet.pdf)
zunaechst sehr streng, fallen aber [bei genauerer Betrachtung](https://wp.josh.com/2014/05/13/ws2812-neopixels-are-not-so-finicky-once-you-get-to-know-them/) eher
als Minima ins Auge, die man es einzuhalten gilt.

Wenige hundert Nanosekunden sind auf einer 8-16 MHz Plattform schon ohne Betriebssystem eine Herausforderung. Mit dem RaspberryPi hatten wir
nun aber ein vollwertiges Raspbian und damit Linux am laufen. Gluecklicherweise ist im Model 3B auch ein ARMv8 mit 600-1200 MHz Taktrate verbaut.

Das erlaubte es uns, per direktem Bitbanging eine (im Mittelwert) recht zuverlaessige Implementation des Protokolls zu schaffen ([siehe demo-Order im 
repository](https://github.com/komaspieler/ws2812-hackathon/tree/master/demo-pigpio)). Hier hatten wir aber das Problem, dass bei fixen 600 MHz (Powersave-Modus)
immer noch vereinzelt 0'en als 1'en erkannt wurden und wir somit abundzu staerkere Artefakte hatten. Gerade bei niedrigen Helligkeiten sind diese
stark aufgefallen.
