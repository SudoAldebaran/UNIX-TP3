# UNIX-TP3

Troisième TP en cours d'UNIX

## Paramètres

### Script :

```
#!/bin/bash

echo "Bonjour, vous rentré $# paramètres"

echo "Le nom du script est $0"

if [ $# -ge 3 ]; then
  echo "Le 3eme paramètre est $3"
else
  echo "Le 3eme paramètre n'a pas été fourni"
fi

echo "Voici la liste des paramètres : $@"
```

### Resultat :

```
root@serveur1:~# ./analyse.sh youpi haha yo
Bonjour, vous rentré 3 paramètres
Le nom du script est ./analyse.sh
Le 3eme paramètre est yo
Voici la liste des paramètres : youpi haha yo
```

## Vérification du nombre de paramètres

### Script :

```
#!/bin/bash

if [ $# -ne 2 ]; then
  echo "Erreur : vous devez rentrer exactement 2 paramèt>
  exit 1
fi

CONCAT="$1$2"

echo "Le résultat de la concatenation est : $CONCAT"
```

### Resultat :

```
root@serveur1:~# ./concat.sh
Erreur : vous devez rentrer exactement 2 paramètres
root@serveur1:~# ./concat.sh hello hello
Le résultat de la concatenation est : hellohello
```

Le message d'erreur fonctionne correctement

## Argument type et droits

### Script : 

```
#!/bin/bash

if [ $# -ne 1 ]; then
  echo "Erreur : Vous devez fournir un fichier ou un répertoire"
  exit 1
fi

FILE=$1

if [ ! -e "$FILE" ]; then
  echo "Le fichier \"$FILE\" n'existe pas"
  exit 1
fi

if [ -d "$FILE" ]; then
  echo "Le fichier \"$FILE\" est un répertoire"
elif [ -f "$FILE" ]; then
  if [ -s "$FILE" ]; then
    echo "Le fichier \"$FILE\" est un fichier ordinaire qui n’est pas vide"
  else
    echo "Le fichier \"$FILE\" est un fichier ordinaire vide"
  fi
else
  echo "Le fichier \"$FILE\" est d’un autre type"
fi

USER=$(whoami)

echo -n "\"$FILE\" est accessible par $USER "

if [ -r "$FILE" ]; then
  echo -n "en lecture"
fi

if [ -w "$FILE" ]; then
  echo -n "en écriture "
fi

if [ -x "$FILE" ]; then
  echo -n "en execution"
fi

echo
```

### Resultat :

```
root@serveur1:~# ./test-fichier.sh date.sh
Le fichier "date.sh" est un fichier ordinaire qui n’est pas vide
"date.sh" est accessible par root en lectureen écriture en execution
root@serveur1:~# ./test-fichier.sh hello.sh
Le fichier "hello.sh" n'existe pas
root@serveur1:~# ./test-fichier.sh lelo
Le fichier "lelo" n'existe pas
```

## Afficher le contenu d'un repertoire

### Script :

```
#!/bin/bash

if [ $# -ne 1 ]; then
  echo "Erreur : Vous devez fournir un repertoire en paramètre"
  exit 1
fi

DIR=$1

if [ ! -d "$DIR" ]; then
  echo "Le repertoire \"$DIR\" n'existe pas ou n'est pas un répertoire"
  exit 1
fi

echo "####### fichiers dans $DIR/"
for file in "$DIR"/*; do
  if [ -f "$file" ]; then
    echo "$file"
  fi
done

echo "####### repertoires dans $DIR/"
for dir in "$DIR"/*; do
  if [ -d "$dir" ]; then
    echo "$dir"
  fi
done
```

### Resultat :

```
root@serveur1:~# ./listedir.sh /boot
####### fichiers dans /boot/
/boot/config-6.1.0-25-arm64
/boot/initrd.img
/boot/initrd.img-6.1.0-25-arm64
/boot/initrd.img.old
/boot/System.map-6.1.0-25-arm64
/boot/vmlinuz
/boot/vmlinuz-6.1.0-25-arm64
/boot/vmlinuz.old
####### répertoires dans /boot/
/boot/efi
/boot/grub
```

## Lister les utilisateurs

### Script :

```
#!/bin/bash

for user in $(cut -d':' -f1,3 /etc/passwd); do
  login=$(echo $user | cut -d':' -f1)
  uid=$(echo $user | cut -d':' -f2)
  if [ "$uid" -gt 100 ]; then
    echo "$login"
  fi
done
```

### Resultat :

```
root@serveur1:~# ./listeuser.sh
nobody
systemd-network
systemd-timesync
sshd
```

### Script awk :

```
#!/bin/bash

awk -F':' '$3 > 100 {print $1}' /etc/passwd
```

cut extrait les champs du fichier /etc/passwd (le champ 1 pour les logins et le champ 3 pour les UIDs)
awk fait la même chose plus simplement, en vérifiant directement si le champ 3 (UID) est supérieur à 100 et en affichant le champ 1 (login)

## Mon utilisateur existe t'il

### Script :

```
#!/bin/bash

if [ $# -ne 2 ]; then
  echo "Erreur : Vous devez fournir deux paramètres : 'login' ou 'uid' et la valeur correspondante."
  exit 1
fi

OPTION=$1
VALUE=$2

if [ "$OPTION" == "login" ]; then
  UID=$(awk -F':' -v login="$VALUE" '$1 == login {print $3}' /etc/passwd)
  if [ ! -z "$UID" ]; then
    echo "L'utilisateur $VALUE a pour UID : $UID"
  fi
elif [ "$OPTION" == "uid" ]; then
  LOGIN=$(awk -F':' -v uid="$VALUE" '$3 == uid {print $1}' /etc/passwd)
  if [ ! -z "$LOGIN" ]; then
    echo "L'utilisateur avec l'UID $VALUE est : $LOGIN"
  fi
else
  echo "Erreur : Le premier paramètre doit être 'login' ou 'uid'."
  exit 1
fi
```

### Resultat :

```
root@serveur1:~# ./checkuser.sh login root
L'utilisateur root a pour UID : 0
```

## Creation utilisateur

### Script :

```
#!/bin/bash

if [ "$USER" != "root" ]; then
  echo "erreur : ce script doit être exécuté en tant que root"
  exit 1
fi

read -p "Login : " login
read -p "Nom : " nom
read -p "Prénom : " prenom
read -p "UID : " uid
read -p "GID : " gid
read -p "Commentaires : " commentaires

# vérification si l'utilisateur existe déja
if [ $(./check_user.sh login "$login") ]; then
  echo "erreur : l'utilisateur $login existe déja"
  exit 1
fi

if [ $(./check_user.sh uid "$uid") ]; then
  echo "erreur : utilisateur avec l'UID $uid existe déja"
  exit 1
fi

# vvérification si un répertoire avec le meme nom dans /home existe
if [ -d "/home/$login" ]; then
  echo "erreur: le répertoire /home/$login existe déja"
  exit 1
fi

# creation de l'utilisateur
useradd -u "$uid" -g "$gid" -c "$nom $prenom, $commentaires" -m -d "/home/$login" "$login"

if [ $? -eq 0 ]; then
  echo "l'utilisateur $login a été créé"
else
  echo "erreur : impossible de créer l'utilisateur $login"
fi
```

### Resultat :

```
root@serveur1:~# ./createuser.sh
Login : ray
Nom : Rayan
Prénom : Super
UID : 99
GID : 00
Commentaires : Moi
./createuser.sh: line 16: ./check_user.sh: No such file or directory
./createuser.sh: line 21: ./check_user.sh: No such file or directory
useradd warning: ray's uid 99 outside of the UID_MIN 1000 and UID_MAX 60000 range.
l'utilisateur ray a été créé
```

L'UID doit se situer entre 1000 et 60000, cependant l'utilisateur a bien été crée

### Verification : 

```
root@serveur1:~# ./checkuser.sh login ray
L'utilisateur ray a pour UID : 99
root@serveur1:~# ./checkuser.sh login root
L'utilisateur root a pour UID : 0
```

## Lecture au clavier 

```
Comment quitter more ? -> touche q

Comment avancer d’une ligne ? -> touche Entrée

Comment avancer d’une page ? ->  barre d’espace

Comment remonter d’une page ? -> less -> b pour remonter d'une page

Comment chercher une chaîne de caractères ? -> taper / suivi de la chaine à chercher, et Entrée

Comment passer à l’occurrence suivante ? -> appuyer sur n après avoir fait une recherche avec /
```

### Script :

```
#!/bin/bash

if [ $# -ne 1 ]; then
  echo "erreur: vous devez specifier un répertoire en argument"
  exit 1
fi

DIR=$1

if [ ! -d "$DIR" ]; then
  echo "erreur : le répertoire spécifié n'existe pas"
  exit 1
fi

for file in "$DIR"/*; do
  if [ -f "$file" ]; then
    file_type=$(file "$file")

    if [[ $file_type == *"text"* ]]; then
      echo -n "Voulez-vous visualiser le fichier $file ? (oui/non) : "
      read response

      if [ "$response" == "oui" ]; then
        more "$file"
      fi
    fi
  fi
done
```

### Resultat :

```
root@serveur1:~# ./voirtextefichier.sh ~
Voulez-vous visualiser le fichier /root/analyse.sh ? (oui/non) : oui
#!/bin/bash

echo "Bonjour, vous rentré $# paramètres"

echo "Le nom du script est $0"

if [ $# -ge 3 ]; then
  echo "Le 3eme paramètre est $3"
else
  echo "Le 3eme paramètre n'a pas été fourni"
fi

echo "Voici la liste des paramètres : $@"
```

## Appreciation

### Script :

```
#!/bin/bash

while true; do
  echo -n "entrez une note (ou appuyez sur 'q' pour quitter) : "
  read input

  if [ "$input" == "q" ]; then
    echo "Programme terminé"
    break
  fi

  # Vérification si l'entrée est un nombre valide
  if ! [[ "$input" =~ ^[0-9]+$ ]]; then
    echo "Veuillez entrer un nombre valide"
    continue
  fi

  note=$input

  if [ "$note" -ge 16 ] && [ "$note" -le 20 ]; then
    echo "Très bien"
  elif [ "$note" -ge 14 ] && [ "$note" -lt 16 ]; then
    echo "Bien"
  elif [ "$note" -ge 12 ] && [ "$note" -lt 14 ]; then
    echo "Assez bien"
  elif [ "$note" -ge 10 ] && [ "$note" -lt 12 ]; then
    echo "Moyen"
  elif [ "$note" -lt 10 ]; then
    echo "Insuffisant"
  else
    echo "Note hors des limites (0-20)"
  fi
done
```

### Resultat :

```
root@serveur1:~# ./appreciation.sh
entrez une note (ou appuyez sur 'q' pour quitter) : q
Programme terminé
root@serveur1:~# ./appreciation.sh
entrez une note (ou appuyez sur 'q' pour quitter) : 12
Assez bien
entrez une note (ou appuyez sur 'q' pour quitter) : 17
Très bien
```
