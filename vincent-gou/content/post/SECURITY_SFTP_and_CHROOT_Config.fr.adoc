---
title: "Paramétrage SFTP sécurisé complet"
authors:
  - Vincent Gourrierec
date: 2019-06-11
tags: ["sftp","linux","securite"]
categories: ["posts"]
draft: false
keywords:
- vincent-gou.fr
- vincent gourrierec
- saint nazaire
- linux
- oracle
- progress
- openedge
- nextcloud
- docker
---

== Objectifs

Fournir un accès sécurisé à des ressources sur un serveur Linux via SFTP.

== Contrôles

=== Contrôle activation SFTP dans SSHD

[source,bash]
----
[root@server /]# cat /etc/ssh/sshd_config


	# override default of no subsystems
	#Subsystem      sftp    /usr/libexec/openssh/sftp-server
	Subsystem sftp internal-sftp
...
	Match Group sftp_users
	ChrootDirectory /data/sftp/%u
	ForceCommand internal-sftp
----

<<<

=== Création de l'arborescence

[source,bash]
----
[root@server /]# mkdir -p /data/sftp
[root@server /]# chmod 700 /data
----

<<<

=== Création du groupe et des users SFTP

[source,bash]
----
groupadd sftp_users // <1>
useradd -g sftp_users -d / -s /sbin/nologin LOGIOUEST // <2>
useradd -g sftp_users -d / -s /sbin/nologin LOGIREP // <3>
useradd -g sftp_users -d / -s /sbin/nologin TMH // <4>
useradd -g sftp_users -d / -s /sbin/nologin SCALIS // <5>
----
<1> Nom du groupe auquel appartiendront tous les utilisateurs SFTP crées.
<2> -g : Groupe d'appartenance du compte LOGIOUEST / -d Chemin du profil / -s shell de connxion => aucun
<3> -g : Groupe d'appartenance du compte LOGIREP / -d Chemin du profil / -s shell de connxion => aucun
<4> -g : Groupe d'appartenance du compte TMH / -d Chemin du profil / -s shell de connxion => aucun
<5> -g : Groupe d'appartenance du compte SCALIS / -d Chemin du profil / -s shell de connxion => aucun


Controle du fichier contenant les utilisateurs SFTP:

[source,bash]
----
cat /etc/passwd
....
LOGIOUEST:x:1002:1002::/:/sbin/nologin
LOGIREP:x:1003:1002::/:/sbin/nologin
TMH:x:1004:1002::/:/sbin/nologin
SCALIS:x:1005:1002::/:/sbin/nologin
----

<<<

==== Informations sur les compte SFTP crées

.Tableau des informations utilisateurs SFTP
[cols="1,3"]
|===
| Compte | Password

| LOGIOUEST
| IARDNdpvzwBo

| LOGIREP
| KDH9MEoIO5DF

| TMH
| msn6eg0QHxd3

| SCALIS
| 4ELVcihQrybK

|===

<<<

=== Création de l'arborescence de stockage des Flux

[source,bash]
----
mkdir -p /data/sftp/LOGIREP/upload // <1>
chown -R root:root /data/sftp/LOGIREP // <2>
chown -R LOGIREP:sftp_users /data/sftp/LOGIREP/upload // <3>
chmod 770 /data/sftp/LOGIREP/upload // <4>

mkdir -p /data/sftp/LOGIOUEST/upload // <1>
chown -R root:root /data/sftp/LOGIOUEST // <2>
chown -R LOGIOUEST:sftp_users /data/sftp/LOGIOUEST/upload // <3>
chmod 770 /data/sftp/LOGIOUEST/upload // <4>

mkdir -p /data/sftp/TMH/upload // <1>
chown -R root:root /data/sftp/TMH // <2>
chown -R TMH:sftp_users /data/sftp/TMH/upload // <3>
chmod 770 /data/sftp/TMH/upload // <4>

mkdir -p /data/sftp/SCALIS/upload // <1>
chown -R root:root /data/sftp/SCALIS // <2>
chown -R SCALIS:sftp_users /data/sftp/SCALIS/upload // <3>
chmod 770 /data/sftp/SCALIS/upload // <4>

----
<1> Création du répertoire personnel de l'utilisateur associé à la société, accessible en R/W pour l'utilisateur.
<2> Affectation des droits à *_root:root_* au répertoire dans lequel sera chrooté de l'utilisateur associé à la société.
<3> Affectation des droits à *_$COMPTE$:sftp_users_* au répertoire dans lequel l'utilisateur associé à la société pourra lire et écrire.
<4> Affectation des droits au répertoire à l'utilisateur et au groupe d'appartenance dans lequel l'utilisateur associé à la société pourra lire et écrire.

<<<

=== Paramétrage compte SFTP ADM

[source,bash]
----
[root@server sftp]# useradd -g sftp_users -d / -s /sbin/nologin SFTP_ADM // <1>
[root@server sftp]# cat /etc/passwd
...
SFTP_ADM:x:1006:1002::/:/sbin/nologin
----

<1> -g : Groupe d'appartenance du compte SFTP_ADM / -d Chemin du profil / -s shell de connxion => aucun

.Tableau d'information de l'utilisateur SFTP_ADM
[cols="1,3"]
|===
| Compte | Password

| SFTP_ADM
| zh5wACyrpMzM

|===

<<<

=== Paramétrage SSHD

[source,bash]
----
[root@server sftp]# cp -p /etc/ssh/sshd_config /etc/ssh/sshd_config.sav
[root@server sftp]# vi /etc/ssh/sshd_config
# override default of no subsystems
#Subsystem      sftp    /usr/libexec/openssh/sftp-server // <1>
## Add umask to U+G none to Others
Subsystem sftp internal-sftp -u 0007 // <2>
IgnoreRhosts yes
IgnoreUserKnownHosts no
PrintMotd yes
StrictModes yes
PubkeyAuthentication yes
#RSAAuthentication yes
PermitRootLogin no
PermitEmptyPasswords no

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server

# A placer avant le test sur le groupe si le user appartient au groupe sftp_users
# SSHD lit la config dans l'ordre d apparition....
Match User SFTP_ADM // <3>
ChrootDirectory /data/sftp // <4>
## Add umask to U+G none to Others
ForceCommand internal-sftp -u 0007 // <5>

Match Group sftp_users // <6>
ChrootDirectory /data/sftp/%u // <7>
## Add umask to U+G none to Others
ForceCommand internal-sftp -u 0007 -d /upload // <8>
----
<1> Désactivation server SFTP par défaut
<2> Activation serveur SFTP intégré à sshd
<3> Bloc concernant le compte SFTP_ADM
<4> Chroot du compte *_SFTP_ADM_* vers /data/sftp
<5> Obligation du SFTP uniquement et changement du UMASK vers 0007 (correspond à 770)
<6> Bloc concernant les membres du groupe *_sftp_users_*
<7> Chroot des membres du groupe *_sftp_users_* vers /data/sftp/%u (%u est une variable pour le nom d'utilisateur)
<8> Obligation du SFTP uniquement et changement du UMASK vers 0007 (correspond à 770) et déplacement automatique dans le répertoire upload

=== Redémarrage SSHD et controles
[source,bash]
----
systemctl restart sshd
systemctl status sshd
----

<<<

==== Paramétrage SSHD complémentaire

Désactivation  RSAAuthentication

[source,bash]
----
[root@server sftp]# vi /etc/ssh/sshd_config
	#RSAAuthentication yes
	RSAAuthentication no
----

Avant:
[source,bash]
----
Jun 04 14:47:43 server systemd[1]: Stopped OpenSSH server daemon.
Jun 04 14:47:43 server systemd[1]: Starting OpenSSH server daemon...
Jun 04 14:47:43 server sshd[48410]: /etc/ssh/sshd_config line 152: Deprecated option RSAAuthentication
Jun 04 14:47:43 server sshd[48410]: Server listening on 0.0.0.0 port 22.
Jun 04 14:47:43 server sshd[48410]: Server listening on :: port 22.
Jun 04 14:47:43 server systemd[1]: Started OpenSSH server daemon.
----

Après:

[source,bash]
----
Jun 04 14:48:54 server systemd[1]: Starting OpenSSH server daemon...
Jun 04 14:48:54 server sshd[48475]: /etc/ssh/sshd_config line 55: Deprecated option RSAAuthentication
Jun 04 14:48:54 server sshd[48475]: /etc/ssh/sshd_config line 153: Deprecated option RSAAuthentication
Jun 04 14:48:54 server sshd[48475]: Server listening on 0.0.0.0 port 22.
Jun 04 14:48:54 server sshd[48475]: Server listening on :: port 22.
Jun 04 14:48:54 server systemd[1]: Started OpenSSH server daemon.
----

<<<

=== Tests

Test connexion Filezilla => OK

Test création d'un fichier sous chaque profil => OK

<<<
