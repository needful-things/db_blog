---
layout: post
title:  Agent Sudo - Writeup
date:   2020-04-27 17:00:00 +0100
image:  posts/agentsudo-tryhackme/agentsudo.png
tags:   TryHackMe.com
---
Agent Sudo ist eine VM auf [TryHackMe.com](https://tryhackme.com). Ziel dieser Maschine war es an die einzelnen Flags zu gelangen.

Generell fange ich immer mit einem NMAP Scan an. Hier sehen wir das 3 Ports offen sind.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/nmap.png)

Mit dem FTP Service konnte ich mich leider nicht mit einem anon User verbinden, daher habe ich mich dem Webservice gewidmet.

Der Webservice zeigte folgenden Inhalt:

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/webservice.png)

Mit der Webconsole im Firefox habe ich mal versucht den User-Agent Header zu bearbeiten. Diesen habe ich auf „R“ gesetzt, weil die Nachricht von Agent R stammte. 

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/user-agent-r.png)

Nach dem Versand des editierten Requests, erhielt ich folgende Antwort

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/user-agent-r-error.png)

Die Nachricht legt nahe, dass die Codenamen der Agenten dem Alphabet entsprechen. Hier habe ich Burp eingeschaltet um automatisch das ganze Alphabet durch zu testen.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/intruder-position.png)

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/intruder-payload.png)

Der Buchstabe C unterschied sich von den anderen und gab eine 302 Meldung zurück.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/intruder-attack.png)

Auf folgende Seite wurde ich weitergeleitet.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/agentc.png)

Aus der Nachricht wissen wir, dass der Name des Agenten C, Chris ist. Ebenfalls wissen wir, dass sein Passwort schwach ist, was gerade nach dem Tool Hydra schreit. Hail Hydra !

Dies beim FTP ausprobiert und wir haben das Passwort von chris.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/chris-pw.png)

Auf dem ftp sehen wir 3 Dateien. 2 Bilder und ein txt file. Die Nachricht „To_agentJ.txt“ gab uns den Hinweis, dass das Passwort von Agent J in einen der beiden Bilder gespeichert sein muss.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/chris-ftp.png)

Mit dem Befehl Strings habe ich untersucht, ob sich vielleicht direkt das Passwort aus einem der beiden Bilder rauslesen lässt. Dies leider ohne Erfolg, aber ich konnte im Bild cutie.png feststellen, dass eine Datei im Bild versteckt sein muss.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/strings.png)

Als nächstes habe ich die süsse.png mit Binwalk überprüft. Ein Zip file war im Bild enthalten. Dieses Zipfile habe ich mittels DD kopiert. (der Parameter binwalk –e wäre auch möglich)

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/binwalk.png)

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/dd.png)

Das Zipfile war mit einem Passwort versehen. Dieses habe ich mit John the ripper geknackt.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/zip2john.png)

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/john-zip.png)

Aus dem Zipfile habe ich folgenden Inhalt erhalten

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/to-agentr.png)

QXJlYTUx ist base64 encoded. Decodiert steht das für „Area51“.

Mit dem Tool steghide und der Passphrase „Area51“ habe ich das erste Bild „cute-alien.jpg überprüft. Und siehe da, hier gibt es ein verstecktes file.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/steghide.png)

Aus dem verstecktem File habe ich nun endlich den Benutzer und das passwort von Agent J herausgefunden.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/message.png)

Mit dem Benutzer "james" und dem Password "hackerrules!" Konnte ich mich per ssh auf der Maschine einloggen und das user flag ergattern.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/userflag.png)

Nachdem ich mich etwas auf der Maschine umgeschaut hatte, hab ich herausgefunden, dass ich sudo Rechte für bash habe.

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/sudo-l.png)

Mittels der folgenden Schwachstelle habe ich root Rechte erhalten: [https://resources.whitesourcesoftware.com/blog-whitesource/new-vulnerability-in-sudo-cve-2019-14287](https://resources.whitesourcesoftware.com/blog-whitesource/new-vulnerability-in-sudo-cve-2019-14287)

![]({{site.baseurl}}/img/posts/agentsudo-tryhackme/rootflag.png)

Insgesamt hat mir die Maschine Spass gemacht aber man muss schon etwas Geduld aufbringen. Der schwierigste Teil meines Erachtens war es, das Zip-file im Bild zu finden.

Danke fürs lesen und bis zum nächsten Artikel.
