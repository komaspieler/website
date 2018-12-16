---
title: "Linux Containers (LXC) mit LXD"
subtitle: Warum Linux Container und LXD ein viel zu unterschätztes Tool sind.
date: 2018-08-14
---

Virtualisierung bietet viele Vorteile - Anwendungen lassen sich damit schnell auf verschiedenen Plattformen/Betriebssystemen testen, Server können in ihren eigenen Sandkästen ausgeführt werden und wenn etwas nicht klappt, kann man mit wenigen Klicks/Befehlen einen einen funktionalen Zustand wiederherstellen. Linux Container bieten eine Möglichkeit der Virtualisierung unter Linux - und vor Kurzem ist mir erst bewusst geworden, wie viel Stress und Nerven ich mir in den letzten Jahren hätte sparen können, hätte ich sie mir früher angesehen.
<!--more-->

[linuxcontainers.org](https://linuxcontainers.org/) ist ein von Canonical (bekannt von Ubuntu) gesponsertes Projekt, das sich mit der Virtualisierung von Unix-basierten Systemen beschäftigt. Dabei wird ein expliziter Fokus darauf gelegt, ein möglichst stromlinienartiges Produkt zu liefern und ohne den "unnötigen" Ballast von Komplettvirtualisierungen (Hyper-V, VirtualBox, VMWare, ..) auszukommen.

Das bringt zwar die Einschränkung mit, dass sich mit LXC/LXD nur Unix-basierte Systeme virtualisieren lassen - möchte man also z.B. Windows unter Linux virtualisieren, muss man weiterhin auf VirtualBox o.ä. zurückgreifen. Aber: alles andere lässt sich innerhalb von Sekunden aufsetzen. Mal eben eine Ubuntu-Anwendung unter Arch Linux ausprobieren? Kein Problem! Eben schauen, ob selbstprogrammierte Anwendungen unter amd64 genau so laufen wie auf ARM? Geht genauso schnell!

Der Clou ist, dass nur ein absolutes Minimalsetup im Image ausgeliefert wird. Dieses ist meistens nur um die 100 MB groß und damit auch relativ schnell heruntergeladen. Ein einfaches ``lxc launch ubuntu:18.04`` reicht aus um eine virtuelle Maschine mit Ubuntu 18.04 ins Leben zu rufen. Ist das Image heruntergeladen und die Maschine gestartet, wechselt man einfach mit ``lxc exec <vmname> bash`` in die Maschine und hat eine root-Shell vor sich - einfacher geht es kaum.

Man spart sich also, das komplette Ubuntu-Setup ausführen zu müssen (Partitionierung, Benutzeraccounts anlegen, Updates herunterladen, Dateien kopieren, ...) und hat binnen Sekunden ein lauffähiges System. Braucht man ein komplexeres Basissetup kann man dieses ein mal hinterlegen und sich den Container dann als Vorlage ablegen.

Auch grafische Ausgaben sind möglich - z.B. über den X Server des Hostsystems (via X-Forwarding oder direkt per Konfiguration) oder auch als remote X client mit ``x2go``.

Ich bin begeistert! :)