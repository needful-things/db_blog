---
layout: post
title:  Mr Robot CTF - Writeup
date:   2020-04-13 18:44:00 +0100
image:  posts/mrrobot-tryhackme/mrrobot-tryhackme.jpeg
tags:   TryHackMe.com
---
Mr Robot CTF ist eine VM auf [TryHackMe.com](https://tryhackme.com), welche sich an Beginner/Intermediate Benutzer richtet. Ziel ist es 3 versteckte keys auf der Maschine zu finden.

Ein NMAP Scan hat mir gezeigt, dass die Ports 80 und 443 offen sind auf der Maschine.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/nmap-scan.png)

Ich habe daher den Browser geöffnet und nachgeschaut was auf dem Port 80 läuft.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/startseite.png)

Es wurde eine kleine Console simuliert bei der man ein paar Commands eingeben konnte. Ich habe jeden Command durchprobiert, konnte aber leider keine nützlichen Informationen finden.
Im nächsten Schritt habe ich das Tool Gobuster verwendet um mir zugängliche Verzeichnisse anzuzeigen.

Hier Interessant war der login und robots Eintrag.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/gobuster.png)

Es gab zwei Einträge in der Robots Datei. 

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/robots.png)

Der erste Eintrag führte mich direkt zum ersten Key.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/key1.png)

Mit dem zweiten Eintrag konnte ich eine grössere Text Datei namens „fsocity.dic“ herunterladen. Die fsocity Datei sah so aus, als würden mit grösster Wahrscheinlichkeit Benutzernamen oder Passwörter sich darin befinden.

Die http://"ip"/login URL leitete mich auf eine Wordpress Login Seite.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/wplogin.png)

Sich mit default credentials einzuloggen hat leider nicht geklappt. Der Versuch ist es aber immer Wert. Mit dem Wissen, dass ich es hier mit einem Wordpress zu tun habe, habe ich mal das Tool „wpscan“ verwendet. Mit diesem Tool lässt sich viel über ein laufendes Wordpress herausfinden.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/wpscan.png)

Die Informationen die ich aus dem Scan erhalten habe wie z.B die Wordpress Version, haben mich nicht weiter gebracht. Ich habe ein bisschen mit den Parametern herumgespielt und versucht Benutzernamen herauszufinden, leider auch erfolglos.
Nach etwas Recherche stiess ich auf folgenden Artikel: [https://hackertarget.com/wordpress-user-enumeration/](https://hackertarget.com/wordpress-user-enumeration/)

Anscheinend besteht die Chance, dass sich die Fehlermeldung unterscheidet bei einem erfolglosen einloggen mit einem vorhandenem User zu einem nicht vorhandenem User.

Mit dem Wissen, habe ich nun versucht mittels dem Tool hydra und der fsocity.dic Datei einen vorhandenen Login Benutzernamen herauszufinden. Und siehe da, ein valider Benutzername ist Elliot.

{% highlight js %}
hydra -L fsocity.dic -p asdf 10.10.116.134 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.102.32%2Fwp-admin%2F&testcookie=1:Invalid username'
{% endhighlight %}

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/loginname.png)

Mit dem Benutzernamen Elliot habe ich nun versucht das Passwort zu Bruteforcen mit derselben Datei „fsocity.dic“. Leider anfangs erfolglos. Es dauert einfach viel zu lange. Irgendeinmal kam es dann auch mir in den Sinn die fsocity Datei mal auf doppelte Einträge zu überprüfen. Folgenden Befehl habe ich verwendet um doppelte Einträge aus der Datei zu entfernen. 

{% highlight js %}
  cat file | sort | uniq > file-uniq
{% endhighlight %}

Die Datei konnte ich so stark verkleinern. Mit der neuen „fsocity-uniq“ Datei nochmals den Bruteforce gestartet und ich konnte das Passwort finden.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/elliot-pw.png)

Erfolgreich eingeloggt im Wordpress, konnte ich auf der 404 Error Page meine Reverse Shell platzieren um mir so Zugang zur Maschine zu verschaffen.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/errorpage.png)

Netcat session geöffnet und eine Seite besucht, die es nicht gibt. Voila ich hab eine Shell.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/shell.png)

Ich habe mich etwas auf der Maschine umgeschaut und gesehen, dass das Homeverzeichnis vom User robot lesbar war. In diesem Verzeichnis befand sich der zweite Key und auch das Password von diesem User als md5 gehashed.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/verzeichnis-robot.png)

Das Passwort aus dem MD5 Hash konnte ich mit dem Tool „john the ripper“ herausfinden.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/john-md5.png)

Mit dem herausgefundenem Passwort konnte ich mich als User robot einloggen und den zweiten Key ausgeben.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/key2.png)

Root Userrechte habe ich erlangt, indem ich mir alle Files aufgelistet habe, welche das SUID Bit gesetzt hatten.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/suid.png)

Hier stach NMAP heraus. NMAP lässt sich im interactive Mode starten. Im interactive Mode ist es möglich Shellcommands abzusetzen.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/nmap-interactive.png)

So habe ich es geschafft den letzten Key auszugeben, welcher sich im Homeverzeichnis vom Root Benutzer befand.

![]({{site.baseurl}}/img/posts/mrrobot-tryhackme/key3.png)

Alles in allem war es eine lustige Maschine. Am meisten Zeit habe ich damit verbraten das Passwort des Wordpress Benutzers zu Bruteforcen. Leider kam mir nicht von Anfang an in den Sinn, die fsocity.dic Datei auf doppelte Einträge zu überprüfen…