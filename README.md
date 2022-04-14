# GravityZone-amavisd-new
Dieses Tool erlaubt die Integartion von "Bitdefender GravityZone Business Security" in eine (bestehende) amavisd-new Installation

Vorbemerkunen:
==============

Irgendwie habe ich aktuell keinen Vernünftigen Virenscanner gefunden, der mit amavis (http://amavis.org) funktionier, mir ClamAV bin ich nicht wirklich Glücklich geworden. Daher ist der entschluss gekommen, selbst einen kommerziellen Virenscanner zu implementieren. Ich habe mich im ersten Schritt für mein wohlgemerkt eher kleines Mail-Setup für Bitdefender GravityZone Business Security entschieden.

Voraussetzungen:
================
- Eine lauffähige amavis-new Installation
- Eine freie Server-Lizenz von Bitdefender GravityZone Business Security
- root Rechte auf dem System mit der laufenden amavis-d Installation

Empfehlungen:
=============
- legt euch ein Sicherheitsprofil an, welches dafür sorgt, dass euch der GravityZone im Bereich des Mailverkehrs auch nicht in die Quere komme,
  bei mir sind mittels policy alle Aktivitäten soweit eingeschränkt, das der Scanner nur per Command-Line aktiv wird. Ich will ja nicht das er
  bevor er aufgerufen wird, schon dinge tut weil jemand mir ne Mail mit Virus schickt

Vorbereitungen:
===============

a) wir müssen leider (ich hab noch keinen anderen Weg gefunden, das System etwas asutricksen um eine vernünftige "root-Umgebung" für den Scan zu haben. Ich empfehle dafür auch entsprechende Härtungsmaßnahmen damit das nicht ausgenutzt werden kann! Per Sudo scheints nicht sinnvoll zu gehen, daher habe ich mich für ein ssh root@localhost entschieden :D
 - passwortloser SSH-Private-Key in /var/spool/amavis/.ssh/
 - passender Public-Key in /root/.ssh/
 - /var/spool/amavisd/.ssh/known_hosts (mit dem ensprechenden Eintrag für localhost)

Beispiel:
```bash
[root@hostname ~]# sudo -u amavis bash
bash-4.2$ cd
bash-4.2$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/var/spool/amavisd/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/spool/amavisd/.ssh/id_ed25519.
Your public key has been saved in /var/spool/amavisd/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx amavis@hostname
The key's randomart image is:
+--[ED25519 256]--+
[...]
+----[SHA256]-----+
bash-4.2$ ssh -l root localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
ECDSA key fingerprint is MD5:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
root@localhost's password: ^C

bash-4.2$ exit
exit
[root@hostname ~]# cat /var/spool/amavisd/.ssh/id_ed25519.pub >> /root/.ssh/authorized_keys
[root@hostname ~]#
```

b) Ich behandle jetzt nicht wie man amavis in die entsprechenden MTA´s einbinden

und dann brauchen wir eigentlich nur noch ein Script und eine Anpassung an der amavisd.conf

Installation:
=============

- copy bitdefender-wrapper -> /usr/local/bin/bitdefender-wrapper 
- change @av_scanners = ( [...] ); mit bitdefender.amavisd-config
- restart amavisd

PS: Meine Scripte sind manchmal etwas umständlich, ich weiß und gerade bei if;then;else bin ich nicht immer ganz konsistent, wers anpassen mag / anders besser findet. No Porblem, bitte einfach entsprechend der Lizenz anpassen.
