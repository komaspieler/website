---
title: "Mailman bei Uberspace"
subtitle: Ein Tutorial zur Installation von Mailman auf einem Uberspace
date: 2015-09-16
---

[Mailman](http://www.gnu.org/software/mailman/) dürfte als Listenmanager bekannt sein wie ein bunter Hund: das benutzerfreundliche Web-Frontend und die gelungenen Archivmöglichkeiten machen ihn zum Listenmanager der Wahl für den Otto-Normal-Nutzer, der die Kommandozeilenbasierte Verwaltung von Mailinglisten nicht gewohnt ist.

<!--more-->
Uberspace bietet von Haus aus [ezmlm-idx Mailinglisten](https://wiki.uberspace.de/mail:ezmlmidx) an. ezmlmidx hat aber z.B. kein Webinterface zum Abbonieren und muss per Kommandozeile administriert werden. Wer seinen Listenmitgliedern und Admins ein Webinterface bieten möchte oder aus anderen Gründen einen mailman haben will, für den ist bei Uberspace die Installation und Benutzung von Mailman wie so vieles anderes ebenfalls möglich - es müssen aber einige Besonderheiten bedacht werden, die man normalerweise nicht berücksichtigen würde.

Dieses Tutorial bezieht sich auf das aktuelle [uberspace mit ContOS 6](https://uberspace.de/tech), nicht auf die [uberspace 7 beta](https://blog.uberspace.de/wip-die-u7-public-beta/).


---

**Dieses Tutorial ergänzt das [offizielle Installation Manual](https://www.gnu.org/software/mailman/mailman-install/mailman-install.html) auf die Besonderheiten bei Uberspace in den Schritten 1 - 7 und 10.**

---

Nicht zuletzt deshalb war auch ich nur wegen der Hilfestellungen und Hinweise des Uberspace-Teams (nochmal vielen Dank an Jonas an dieser Stelle) dazu in der Lage. Doch wie installiert man Mailman nun bei Uberspace?

Im Folgenden gelten folgende Variablen:

- `$uberspacehosthost` - Name des Uberspace-Hosts
- `$mailhost` - die Domain der Mailadressen der Mailmanlisten 
- `$urlhost` - die Domain unter der das Mailman Webfrontend laufen soll

Außerdem gehe ich davon aus, dass du die angegebenen Befehle auf dem Zielaccount bei Uberspace ausführst. Sollte dies nicht der Fall sein, musst du ggf. in allen relevanten Befehlen die Zeichenkette `` `whoami` `` durch den gewünschten Benutzernamen ersetzen.

### Schritt 1 - das Programmarchiv besorgen

> **Update vom 25.08.2014:** Diese Anleitung kann auch problemlos mit Version 2.1.18-1 durchgeführt werden! Die Abhängigkeit zu dnspython kann durch den einfachen Befehl `easy_install dnspython` erfüllt werden! (Danke, Fabian!)

~~Zum Zeitpunkt dieses Tutorials ist die Version 2.1.18-1 die aktuellste von Mailman. Diese besitzt jedoch eine Abhängigkeit zum bei Uberspace nicht vorinstallierten Python-Moduls "dnspython". Da ich mich mit der Installation dieses Moduls noch nicht beschäftigt habe und ohne die Neuerungen in der .18-1 leben kann, bin ich vorerst auf die Vorgängerversion 2.1.17 umgestiegen. Ein Tutorial mit Ergänzung um die Installation von dnspython folgt eventuell noch.~~

Zum Herunterladen des Pakets für Mailman 2.1.17 loggen wir uns zunächst bei Uberspace ein und laden dann das Paket, entpacken es in `~/tmp/` und wechseln in das Verzeichnis:

    wget http://ftp.gnu.org/gnu/mailman/mailman-2.1.17.tgz
    tar xfz mailman-2.1.17.tgz -C ~/tmp/
    cd ~/tmp/mailman-2.1.17/

Bevor wir weiter machen, legen wir nun auch schon das Installationsverzeichnis an:

    mkdir /var/www/virtual/`whoami`/mailman
    chmod a+rx,g+ws /var/www/virtual/`whoami`/mailman


### Schritt 2 - das Configure-Skript laufen lassen

Nun müssen wir Mailmans configure laufen lassen mit folgendem Aufbau (Optionen nachzulesen in der [offiziellen Installationsanleitung](https://www.gnu.org/software/mailman/mailman-install/mailman-install.html)).

    ./configure --with-username=`whoami` --with-groupname=`whoami` \
    --prefix=/var/www/virtual/`whoami`/mailman/ \
    --with-mail-gid=`whoami` --with-cgi-gid=`whoami` \
    --with-mailhost=$mailhost \
    --with-urlhost=$urlhost


### Schritt 3 - make & make install

Wenn alles gut gegangen ist bei configure, können wir nun make ausführen:

    make

Erscheinen hier keine Fehler, kann auch	`make install`	ausgeführt werden.


### Schritt 4 - Installation prüfen

Wir wechseln mit `cd /var/www/virtual/`whoami`/mailman` in das Verzeichnis und führen dort das Skript `bin/check_perms` aus. Dieses Überprüft die (Mindest-!)Berechtigungen der Verzeichnisse und gibt uns bei Abweichungen von der Norm ein paar Hinweise zum Beheben. Werden Fehler gefunden, können diese durch erneuten Aufruf des Skripts mit `bin/check_perms -f` behoben werden. Idealerweise behebt ihr alle Fehler, die das Skript nicht beheben kann, noch manuell (z.B. die Verzeichnisrechte für das private archive folder).


### Schritt 5 - den Webserver vorbereiten

Bei diesem Schritt hatte ich die meisten Probleme. Da wir bei Uberspace keinen Zugriff auf die VirtualHost-Konfiguration des Webservers haben, kann dieser Schritt nicht entsprechend des offiziellen Handbuchs befolgt werden. Stattdessen müssen wir einen SymLink und eine htaccess Datei anlegen:

    ln -s /var/www/virtual/`whoami`/mailman/cgi-bin /var/www/virtual/`whoami`/html/mailman
    ln -s /var/www/virtual/`whoami`/mailman/archives/public /var/www/virtual/`whoami`/html/pipermail
    printf "Options +ExecCGI\nSetHandler cgi-script" > /var/www/virtual/`whoami`/mailman/cgi-bin/.htaccess

Anschließend korrigieren wir noch die Datei- und Verzeichnisrechte für die CGI-Skripte:

    chmod -R 0755 /var/www/virtual/`whoami`/mailman/cgi-bin


### Schritt 6 - auf die Verwendung von qmail vorbereiten

In Mailman können Administratoren neue Listen per Webinterface anlegen. Dieses Tutorial nutzt (aktuell) noch ein bashscript zum Erstellen neuer Listen. Die Liste wird hierbei im Webinterface erzeugt und die nötigenWeiterleitungen mit dem unten stehenden Script erzeugt.

Im [Handbuch zur Installation](https://www.gnu.org/software/mailman/mailman-install/qmail-issues.html)gibt es ein einfaches Skript, welches wir benutzen können, um die nötigen .qmail-Dateien anzulegen, die Mailman zum Funktionieren braucht. Da dieses noch leicht angepasst werden muss, hier ein Vollzitat mit angepassten Pfaden:

    #!/bin/sh
    if [ $# = 1 ]; then
    i=$1
    echo Making links to $i in the current directory...
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman post $i" > .qmail-$i
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman admin $i" > .qmail-$i-admin
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman bounces $i" > .qmail-$i-bounces
    # The following line is for VERP
    # echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman bounces $i" > .qmail-$i-bounces-default
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman confirm $i" > .qmail-$i-confirm
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman join $i" > .qmail-$i-join
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman leave $i" > .qmail-$i-leave
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman owner $i" > .qmail-$i-owner
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman request $i" > .qmail-$i-request
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman subscribe $i" > .qmail-$i-subscribe
    echo "|preline /var/www/virtual/`whoami`/mailman/mail/mailman unsubscribe $i" > .qmail-$i-unsubscribe
    fi

Dieses Skript wird dann im Home-Verzeichnis z.B. als addlist.sh als ausführbare Datei abgelegt und muss ausgeführt werden, wenn eine neue Liste angelegt wird (als `~/addlist.sh listenname`).


### Schritt 7 - die restliche Installation

> **Update 25.03.2017:** In neueren Versionen ist diese Anpassung nicht mehr nötig, da die SMTP Auth Option offiziell übernommen und eingepflegt wurde!

Der Rest der Installation läuft mit einer Ausnahme ab wie im [offiziellen Handbuch](https://www.gnu.org/software/mailman/mailman-install/customizing.html) geschrieben. Die Ausnahme bezieht sich auf die Anpassung der mm_cfg.py in /var/www/virtual/`whoami`/mailman/Mailman:

Zuerst legt ein [E-Mail Postfach](https://wiki.uberspace.de/mail:vmailmgr) für den Listenverkehr im Webinterface von Uberspace oder auf der [Konsole](https://wiki.uberspace.de/mail:vmailmgr#e-mail-account_anlegen) an (z.B. 'listenversender@domain.tld' mit Passwort 'HollaDieWaldF33!' - ihr benutzt natürlich andere Daten!!).

Dann müssen an der dafür vorgesehenen Stelle (siehe Originaldatei) folgende Zeilen eingefügt werden:

    SMTP_AUTH = Yes
    SMTPHOST = '$uberspacehost.uberspace.de'
    SMTPPORT = '587'
    SMTP_USER = 'listenversender@domain.tld'
    SMTP_PASSWD = 'HollaDieWaldF33!'
    DELIVERY_MODULE = 'ASMTPDirect'


Das Modul "ASMTPDirect" existiert allerdings noch nicht, dafür müssen wir noch wenige Schritte durchführen. Zunächst wechseln wir in das Verzeichnis <i>/mailman/Mailman/Handlers</i> und kopieren die SMTPDirect.py:

    cd /var/www/virtual/`whoami`/mailman/Mailman/Handlers
    cp SMTPDirect.py ASMTPDirect.py

Nun muss die ASMTPDirect.py noch bearbeitet werden (z.B. mit nano), wobei folgende zwei Zeilen zwischen Zeile 64 und 65 der Originaldatei eingefügt werden müssen:

    if mm_cfg.SMTP_AUTH:
                    self.__conn.login(mm_cfg.SMTP_USER, mm_cfg.SMTP_PASSWD)

Anschließend die Änderungen speichern. Der Rest kann komplett konform mit der [offiziellen Anleitung](https://www.gnu.org/software/mailman/mailman-install/customizing.html) befolgt werden. Hier kann mit Schritt 7 weiter gemacht werden (Customizing).


### Schritt 9

Im Schritt [Set up cron](https://www.gnu.org/software/mailman/mailman-install/node41.html) soll man den cron als user `mailman` (`crontab -u mailman crontab.in`) anlegen. 

Da mailman unter deinem uberspace-user läuft muss und kann man den [cron](https://wiki.uberspace.de/system:cron) als uberspace-user anlegen. 

		cd /var/www/virtual/`whoami`/mailman/cron/
		crontab crontab.in

### Schritt 10 


Bei [Schritt 10](https://www.gnu.org/software/mailman/mailman-install/node42.html) sind nochmal ein paar Anpassungen notwendig. Wir starten hier den "qrunner" der dafür sorgt, dass Mailman ankommende Mails auch an die Mitglieder der Mailinglisten verteilen kann. Außerdem stellen wir sicher, dass der qrunner nach einem Reboot automatisch gestartet wird. Dazu greifen wir auf die daemontools, die von uberspace bereit gestellt werden, zurück.

[Lest euch dazu unbedingt den Artikel im uberspace-wiki durch, da es ein paar Stolperfallen gibt, die man kennen sollte.](https://wiki.uberspace.de/system:daemontools)

Zunächst legen wir uns einen eigenen ~/service Ordner an:

    test -d ~/service || uberspace-setup-svscan

Dann erstellen wir den Ordner ~/etc/mailman-supervise:

    mkdir ~/etc/mailman-supervise

Anschließend brauchen wir in diesem Ordner ein Skript mit dem Namen run, das wir z.B. mit nano erstellen:

    nano ~/etc/mailman-supervise/run

Der Inhalt des Skripts ist folgender:

    #!/bin/sh
    exec ./qrunner

Als nächstes erstellen wir das Skript ~/etc/mailman-supervise/qrunner mit folgendem Inhalt:

    #!/bin/sh
    while `/bin/true`; do
    /var/www/virtual/`whoami`/mailman/bin/qrunner --runner=All --once
    sleep 60
    done

Jetzt erstellen wir den Ordner ~/etc/mailman-supervise/log:
    mkdir ~/etc/mailman-supervise/log

Und das Skript ~/etc/mailman-supervise/log/run mit dem Inhalt:

    #!/bin/sh
    exec multilog t ./main

Nun ändern wir noch die Rechte:

    chmod 755 ~/etc/mailman-supervise/*
    chmod 755 ~/etc/mailman-supervise/log/run

Als letztes verlinken wir den Ordner ~/etc/mailman-supervise nach ~/service:
    ln -s ~/etc/mailman-supervise ~/service

Nun läuft der qrunner. Mit dem Befehl `svstat ~/service/mailman-supervise` kann man das noch überprüfen. Der Befehl gibt zurück wie lange der Daemon bereits läuft.

Ergänzung zu Schritt 10 von Jonatan von [Uberspace](https://uberspace.de).

## Debug

Du hast alles so gemacht wie oben beschreiben und es läuft trotzdem nicht?
Unter `/var/www/virtual/`whoami`/mailman/logs` werden zahlreiche logs geschrieben.

Oder im [offiziellen Troubleshooting](https://www.gnu.org/software/mailman/mailman-install/troubleshooting.html) nachschlagen.

## https

[Wir alle empfehlen dir ausdrücklich, von HTTPS Gebrauch zu machen](https://wiki.uberspace.de/webserver:https) und [Mailman kann auch https](https://wiki.list.org/DOC/4.27%20Securing%20Mailman%27s%20web%20GUI%20by%20using%20Secure%20HTTP-SSL%20%28HTTPS%29).


Dazu in in der Datei `/mailman/Mailman/mm_cfg.py` folgendes Pattern eingeben: 

		DEFAULT_URL_PATTERN = 'https://%s/mailman/'

Und dann die URLs neu laden lassen.

		/var/www/virtual/`whoami`/mailman/bin/withlist -l -a -r fix_url
