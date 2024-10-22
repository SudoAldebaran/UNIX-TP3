# UNIX-TP2

Deuxieme TP en cours d'UNIX

# 1 - Secure Shell : SSH

## 1.1 Exercice : connexion ssh root

Dans `sshd_config`, l'élément à configurer est PermitRootLogin

Les différentes options sont : 

> no

Les connexions SSH pour l'utilisateur root sont complètement interdites

Avantages :

- améliorer la sécurité en empechant les attaques par force brute sur le compte root
- permettre d'utiliser des comptes non privilégiés, ce qui est une bonne pratique de sécurité

Inconvénients :

- nécessite une connexion via un autre utilisateur, ce qui peut ajouter des étapes supplémentaires pour les admin qui doivent exécuter des commandes root.

Utilisation : Utilisé dans les environnements de production pour réduire le risque d'accès non autorisé

> yes

Les connexions ssh root sont autorisées avec un mot de passe

Avantages :

Pratique pour l'administration à distance, car les admin peuvent se connecter directement en tant que root

Inconvénients :

- augmente le risque de piratage, un attaquant peut essayer de deviner le mot de passe root
- expose le système à des attaques par force brute

Utilisation : Utilisé dans des environnements de test ou de développement où l'accès rapide à root est nécessaire, mais pas recommandé en production

> prohibit-password

Les connexions ssh root sont interdites via un mot de passe, mais autorisées avec des clefs ssh

Avantages :

- permet un accès sécurisé via des clefs, réduit donc le risque d'infiltration tout en étant flexible via l'authentification avec clefs

Inconvénients :

- nécessite la gestion des clefs ssh, ce qui peut être compliqué pour certains utilisateurs

Utilisation : Recommandé pour les environnements de production où la sécurité est primordiale, mais où un accès root est nécessaire

## 1.2 Exercice : authentification par clef / génération de clefs


### Génération des clefs publique et privée

>> ssh-keygen 

Les clefs ont bien été créee :

Emplacement de la clef privée :

Your identification has been saved in /Users/rayan/.ssh/id_ed25519
Your public key has been saved in /Users/rayan/.ssh/id_ed25519.pub

## 1.3 Exercice : authentification par clef / génération de clefs

### Copie de la clef publique sur le serveur distant

>> ssh-copy-id root@172.16.233.130

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/rayan/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@172.16.233.130's password: 

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'root@172.16.233.130'"
and check to make sure that only the key(s) you wanted were added.

Après la saisie du mot de passe, la clef a bien été transferée dans /root/.ssh/authorized_keys sur le serveur distant

Desormais, je suis le seul (ordinateur) à pouvoir me connecter au serveur, étant le seul à détenir la clef privée

Lors de la tentative de connexion, ssh recherche et essaie toutes les clefs présentes sur la machine locale :


debug1: Offering public key: /Users/rayan/.ssh/id_ed25519 ED25519 SHA256:Zk1jY8AoCwjdlBeKw30lkRpFSH0EDeC7M27ZqgjrR1I
debug1: Server accepts key: /Users/rayan/.ssh/id_ed25519 ED25519 SHA256:Zk1jY8AoCwjdlBeKw30lkRpFSH0EDeC7M27ZqgjrR1I
Authenticated to 172.16.233.130 ([172.16.233.130]:22) using "publickey".

Ci-dessus, le serveur a accepté la clef correspondante lorsque celle-ci a été envoyée.


## 1.4 Exercice : Authentification par clef : depuis la machine hote

>> rayan@Host-002 .ssh % ssh -i id_ed25519 root@172.16.233.130

Resultat : connexion reussie 

Preuve dans le changement du nom de Host :

>> root@serveur1:~# 

## 1.5 Exercice : Sécurisez

Afin de sécuriser l'accès à la machine root par clef uniquement, il faut changer les autorisations dans sshd_config et donc modifier les lignes suivantes :

>> PermitRootLogin prohibit-password
>> PasswordAuthentication no

Les attaques brute force ssh consistent à essayer de deviner le mot de passe d'un utilisateur en testant des milliers de combinaisons de mots de passe de manière automatique (souvent en utilisant des dictionnaires et des programmes tels que JohnTheRipper)

### Autres techniques de protection

> Fail2ban : surveille les tentatives de connexion ssh dans les logs et bloque les adresses IP après plusieurs tentatives échouées

Avantage : facile à configurer et efficace pour bloquer temporairement les attaques répétées

Inconvénient : ne protège pas contre des attaques distribuées venant de différents ip

> Changer le port par défaut (22) : modifier le port ssh par defaut pour réduire la visibilité du serveur aux scans automatisés

Avantage : reduit les tentatives d'attaque automatiques

Inconvénient : ne protège pas contre des attaques ciblées

# 2 - Processus

## 2.1 Exercice : Etude des processus UNIX

### 1

>> ps -eo user,pid,%cpu,%mem,stat,lstart,time,command

TIME correspond au temps CPU cumulé utilisé par le processus. C'est le temps total pendant lequel le processeur a été occupé à exécuter ce processus

Processus ayant utilisé le plus le processeur : 640 _windowserver      9,5  0,6 Ss   Sam 21 sep 15:35:11 2024     812:03.92

Premier processus lancé après le démarrage du système : 948 Sam 21 sep 15:35:29 2024     /System/Library/CoreServices/SubmitDiagInfo server-init


Ma machine a démarrée le 21 septembre à 15:33 :

>> who -b
>> system boot  21 sep 15:33 

Autre commande pour trouver le temps depuis lequel le serveur tourne : 

>> uptime
>> 00:52:42 up 2 min,  2 users

Nombre approximatif de processus crées depuis le démarrage de la machine : 

>> ps aux | wc -l

Resultat : 715

### 2

Afficher tous les PPIDs :

>> ps -eo pid,ppid,comm   

Resultat pour ps :

23544 21597 ps

>> ps -o pid,ppid,comm -p 21597

Resultat :

  PID  PPID COMM
21597 21596 -zsh


Même chose avec pstree : 

>> pstree -p 21597
-+= 00001 root /sbin/launchd
 \-+= 19365 rayan /System/Applications/Utilities/Terminal.app/Contents/MacOS/Terminal
   \-+= 21596 root login -pf rayan
     \-+= 21597 rayan -zsh
       \-+= 23560 rayan pstree -p 21597
         \--- 23561 root ps -axwwo user,pid,ppid,pgid,command

### 4

Affichage des processus les plus gourmands classés par resident memory (RES) :

>> top -o RES

Resultat :

612 root      20   0  400708  24228  10620 S   0.0   1.2   0:00.29 fail2ban-server    


fail2ban-server est le processus le plus gourmand

Changement de couleur : touche Z pendant que top est lancé -> selection de la couleur


Changer la colonne de trie : Shift + (colonne que l'on souhaite mettre en avant)

>> htop

Avantages : c'est beaucoup plus jolie ! Il est plus intuitif que top. Commandes pour kill des processus

Inconvenients : plus gourmand que top

# 3 - Arret d'un processus


Création des deux scripts dans leurs fichiers respectifs :

>> nano date.sh 

!/bin/sh
while true; do 
    sleep 1
    echo -n 'date '
    date +%T
done

>> nano date-toto.sh

!/bin/sh
while true; do 
    sleep 1
    echo -n 'toto '
    date --date '5 hour ago' +%T
done

Problème rencontré, les scripts ne se lancent pas :

>> ./date.sh
>> -bash: ./date.sh: Permission denied

Verification des permitions de date.sh :

>> ls -l date.sh
>> -rw-r--r-- 1 root root 76 Oct 14 02:06 date.sh

-rw-r--r-- indique que le fichier n'est pas exécutable

Donner les permitions d'execution avec la commande : 

>> chmod +x date.sh

Reverification :

>> ls -l date.sh
>> -rwxr-xr-x 1 root root 76 Oct 14 02:06 date.sh

-rwxr-xr-x indique que le fichier peut bien s'executer, le programme fonctionne bien

Lancement des scripts : 

>> ./date.sh
>> ./date-toto.sh

Arret des processus à l'aide de kill :

Trouver le PID du script avec :

>> ps aux

root        1374  0.0  0.0   2324   816 pts/0    S+   02:30   0:00 /bin/sh ./date.sh

>> kill 1374

>> #!/bin/sh : Indique que le script doit être exécuté avec l’interpréteur de commandes sh
>> while true; do ... done : crée une boucle infinie, exécutant le bloc de code entre do et done sans fin
>> sleep 1 : met le script en pause pendant 1 seconde à chaque itération pour éviter une surcharge d'affichage
>> echo -n 'date ' : affiche le texte 'date' sans saut de ligne
>> date --date '5 hour ago' +%T : affiche l'heure qu'il était 5 heures auparavant au format heure:minutes

# 4 - Les tubes

## Différence entre tee et cat

### cat :

Utilisé principalement pour afficher le contenu d'un fichier dans la sortie standard (écran) ou pour concaténer plusieurs fichiers.
Il ne modifie pas le contenu des données, il les affiche ou les combine.

tee :

Utilisé pour lire de l'entrée standard, écrire dans un ou plusieurs fichiers, et aussi afficher le contenu à la sortie standard.
Cela permet de « diviser » la sortie : elle est à la fois affichée et enregistrée dans un fichier.


### Explication des commandes


>> ls | cat

Affiche simplement la liste des fichiers sur l'écran, mais l'utilisation de cat ici fais en sorte que les fichiers soient affichés superposés contrairement à ls uniquement

>> ls -l | cat > liste

Crée le fichier liste avec la liste détaillée des fichiers

>> ls -l | tee liste

Afficher la liste détaillée des fichiers à l'écran et l'enregistre dans le fichier liste

>> ls -l | tee liste | wc -l

Affiche la liste détaillée des fichiers à l'ecran, l'enregistre dans liste, et affiche le nombre de lignes

# 5 - Journal système rsyslog

### rsyslog

>> ps aux | grep syslog

Resultat :

root         575  0.0  0.1 222128  3832 ?        Ssl  22:04   0:00 /usr/sbin/rsyslogd -n -iNONE

rsyslog est bien lancé. Son PID est 575

>> /var/log/syslog 

C'est le fichier où rsyslog écrit la plupart des messages systeme standards, y compris ceux provenant des services

>> /var/log/auth.log 

Pour les messages liés à l'authentification (connexions ssh, sudo, etc...)

/var/log/kern.log : contient les messages relatifs au noyau (kernel)
/var/log/dmesg : contient les messages produits pendant le démarrage du système
/var/log/mail.log : utilisé pour enregistrer les messages relatifs aux services de mails
/var/log/daemon.log : enregistre les messages provenant des daemons en arrière-plan

### cron

Le service cron est utilisé pour exécuter des tâches planifiées automatiquement à des intervalles réguliers. Les utilisateurs peuvent définir des commandes ou des scripts à exécuter à des heures spécifiques ou à des périodes récurrentes, comme tous les jours ou chaque semaine.

tail -f affiche en temps réel les dernières lignes d'un fichier. Souvent utilisée pour surveiller des fichiers de log, car elle continue d'afficher les nouvelles lignes ajoutées au fichier au fur et à mesure qu'elles sont écrites

En se connectant à la VM depuis un autre terminal, affichage de la connexion dans le journal : 

Oct 15 22:39:21 serveur1 sshd[729]: Accepted publickey for root from 172.16.233.1 port 61849 ssh2: ED25519 SHA256:Zk1jY8AoCwjdlBeKw30lkRpFSH0EDeC7M27ZqgjrR1I
Oct 15 22:39:21 serveur1 sshd[729]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
Oct 15 22:39:21 serveur1 systemd-logind[541]: New session 5 of user root.
Oct 15 22:39:21 serveur1 systemd[1]: Started session-5.scope - Session 5 of User root.
Oct 15 22:39:21 serveur1 sshd[729]: pam_env(sshd:session): deprecated reading of user environment enabled

 /etc/logrotate.conf est un fichier de configuration utilisé par le service logrotate, qui gère la rotation, la compression, la suppression et l'envoi par email des fichiers journaux sur linux

## dmesg

La commande dmesg affiche que le cpu de ma machine est de type ARM (Macbook M2)

Le modèle de carte réseaux détecté est le suivant : Intel(R) PRO/1000 Network Driver
