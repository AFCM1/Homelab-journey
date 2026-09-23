# Tutoriel : Déployer et configurer Jellyfin installé sur LXC

Jellyfin est une alternative 100% open-source et gratuite à Netflix, Plex et Prime Video. Contrairement à d'autres solutions, Jellyfin ne possède aucune fonctionnalité cachée derrière un paywall (pas de "Pass" payant) et respecte totalement la vie privée.
Vous pourrez ainsi visionner les films et séries que vous possédez.

Jellyfin est un service que j'utilise quasiment tous les jours, que ça soit chez moi en local ou lorsque je suis à l'extérieur (J'ai rendu mon Jellyfin accessible depuis l'extérieur via [Pangolin](https://docs.pangolin.net/) installé sur un VPS (Fournisseur IONOS 2,7€/mois).
Cela me permet donc également d'ouvrir l'accès à mes proches et ami(e)s.

---

## Sommaire

- [Pourquoi Jellyfin ?](#pourquoi-jellyfin-)
- [1 - Méthodes d'installation principales](#1---méthodes-dinstallation-principales)
- [2 - Mapping des dossiers Synology/Media sur le host Proxmox et LXC Jellyfin](#2---mapping-des-dossiers-synologymedia-sur-le-host-proxmox-et-lxc-jellyfin)
  - [2.1 - Montage CIFS Synology sur le host Proxmox](#21---montage-cifs-synology-sur-le-host-proxmox-prérequis)
  - [2.2 - Bind-mount côté host Proxmox](#22---bind-mount-côté-host-proxmox)
  - [2.3 - Accès groupe côté LXC (mapping uid/gid)](#23---accès-groupe-côté-lxc-mapping-uidgid)
- [3 - Configuration](#3---configuration)

Ce dont je ne traiterai pas ici :
- Le transcoding
- L'accès à Jellyfin hors de mon réseau local
- Automatiser l'obtention des films, séries et sous-titres avec la suite *arr (Sonarr, Radarr..)
---

## Pourquoi Jellyfin ?

- **Zéro abonnement :** Fini de subir les hausses de prix des plateformes de streaming de films et séries.
- **Contrôle total :** Tes fichiers médias restent sur ton stockage local. Aucune donnée de visionnage n'est partagée avec des tiers.
- **Haute performance :** Supporte le transcodage matériel (GPU Intel/NVIDIA) pour lire sans problèmes tes vidéos en 4K sur n'importe quel écran.

---

## 1 - Méthodes d'installation principales

1. Via **Docker Compose**. [Lien site officiel Jellyfin](https://jellyfin.org/docs/general/installation/container/)
2. Mon jellyfin est installé sur un LXC Unprivileged (Linux Container). [Lien d'installation](https://community-scripts.org/scripts/jellyfin).

---

### Méthode 1 : docker-compose

Crée un dossier nommé `jellyfin` sur ton serveur, puis copies colles-y le [docker-compose.yml](https://github.com/AFCM1/homelab_journey/blob/jellyfin/jellyfin/docker-compose.yml).
Puis ensuite lances le avec `docker compose up -d`

### Méthode 2 : LXC

Sur ce site se trouve tout un tas de script d'installation pour mettre en place vos services [LXC Jellyfin](https://community-scripts.org/scripts/jellyfin).

## 2 - Mapping des dossiers Synology/Media sur le host Proxmox et LXC Jellyfin

Vue d'ensemble du chemin de données :

```
Synology (partage CIFS) → mount autofs sur le host Proxmox → bind-mount mp0 dans le LXC → groupe lxc_shares (GID 10000)
```

Mes fichiers multimédia sont stockés sur mon NAS Synology dans un dossier à sa racine
```
/synology/Media/Movies
/synology/Media/Anime
/synology/Media/Series
```
<img width="1042" height="680" alt="image" src="https://github.com/user-attachments/assets/6865fd9b-a21e-470f-a517-8a27e0902d46" />

Avant de pouvoir bind-mount un partage réseau sur un LXC, il faut que le host Proxmox monte lui-même les partages CIFS du Synology, via `autofs` + `/etc/fstab`.

### 2.1 - Montage CIFS Synology sur le host Proxmox (prérequis)

### Fichier de credentials

> ⚠️ **Important :** ne jamais mettre le mot de passe en clair dans `/etc/fstab` — ce fichier est lisible par tout utilisateur local. Utilise un fichier dédié :

```bash
cat > /etc/synology-credentials <<'EOF'
username=<synology_user>
password=<synology_password>
EOF
chmod 600 /etc/synology-credentials
```

### Entrées `/etc/fstab`

Une ligne par partage réseau, montage à la demande via `x-systemd.automount` (évite un boot bloquant si le NAS n'est pas encore up) :

```
//<ip_synology>/Media /mnt/synology/Media cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,forceuid,forcegid,dir_mode=0770,file_mode=0770,credentials=/etc/synology-credentials 0 0
```
<img width="1534" height="148" alt="image" src="https://github.com/user-attachments/assets/4d69c9cd-eb5a-4012-bc06-30da1e847399" />

### 2.2 - Bind-mount côté host Proxmox

Le conteneur LXC Jellyfin doit être arrêté pour modifier son fichier de conf en toute sécurité :

```bash
pct stop <ctid>
```

Vérifie qu'aucun index `mpN` n'est déjà utilisé :

```bash
grep mp /etc/pve/lxc/<ctid>.conf
```

Ajoute la ligne (adapte l'index si `mp0` existe déjà) :

```bash
echo "mp0: /mnt/synology/Media,mp=/mnt/synology/Media" >> /etc/pve/lxc/<ctid>.conf
```
`mp0` : mount point (mp0, mp1, mp2...)  
`/mnt/synology/Media` (avant la virgule) : chemin sur l'hôte Proxmox  
`mp=/mnt/synology/Media` (après la virgule) : chemin à l'intérieur du LXC où ce dossier apparaîtra  

Démarre le container LXC :

```bash
pct start <ctid>
```

En te connectant en ssh sur ton LXC Jellyfin tu devrais désormais voir le partagé monté :

```bash
ls -la /mnt/synology/Media
```
<img width="736" height="384" alt="image" src="https://github.com/user-attachments/assets/bd8caf29-691a-4c5c-97f7-601ff331df4b" />

### 2.3 - Accès groupe côté LXC (mapping uid/gid)

#### 📝 Comprendre le mapping

Un LXC **unprivileged** ne voit jamais les vrais UID/GID du host : le kernel lui
réserve une plage dédiée (`subuid`/`subgid`, généralement `100000–165535`) et
fait correspondre l'UID/GID interne `0` (root du CT) au `100000` du host, `1`
au `100001`, etc. — un simple décalage fixe de `100000`.

### En résumé

| Contexte | UID/GID observé |
|---|---|
| Host Proxmox | `110000` |
| Conteneur LXC (Jellyfin) | `10000` |

Dès que Jellyfin dans le LXC tente de lire un fichier, c'est le
**kernel** via le mécanisme des *user namespaces* qui traduit
automatiquement `10000` en `110000` pour vérifier les droits côté host.

Le mount CIFS côté host utilise justement `uid=100000,gid=110000`, ce qui donne une fois remappé :

```
host: 100000 → conteneur: 0     (root)
host: 110000 → conteneur: 10000
```

Concrètement, côté host :

```bash
cd /mnt/synology/Media
ll
# drwxrwx--- 2 100000 110000 ... Movies
```

Et depuis l'intérieur du CT, le même dossier :

```bash
pct enter <ctid>
cd /mnt/synology/Media
ll
# drwxrwx--- 2 root lxc_shares ... Movies
```

C'est le même fichier, juste vu à travers le décalage du conteneur.

#### Permission user jellyfin

L'utilisateur `jellyfin` n'étant pas root dans le CT, il n'a accès en lecture/écriture qu'en rejoignant le groupe GID `10000` :

```bash
groupadd -g 10000 lxc_shares # on crée un groupe lxc_shares avec une gid fixe
usermod -aG lxc_shares jellyfin # on ajoute jellyfin au groupe lxc_shares afin que jellyfin puisse lire/écrire le dossier Media et donc permettre à Jellyfin de scanner la bibliothèque
systemctl restart jellyfin   # nécessaire pour que le nouveau groupe soit pris en compte
```

Vérifie l'appartenance au groupe :

```bash
id jellyfin
# uid=110(jellyfin) gid=118(jellyfin) groups=118(jellyfin),44(video),104(render),10000(lxc_shares)
```

#### Test d'écriture

Valide que `jellyfin` a bien les droits en écriture (pas juste en lecture) :

```bash
sudo -u jellyfin touch /mnt/synology/Media/Movies/.test && echo OK && rm /mnt/synology/Media/Movies/.test
```

`OK` en sortie = accès confirmé bout en bout (host CIFS → LXC → utilisateur jellyfin).
Tu peux passer à la config des bibliothèques dans Jellyfin (section 3).

## 3 - Configuration

1 - Rendez-vous sur l'ip de votre serveur (ip de votre serveur docker ou ip de votre LXC) : `http://<ip>:8096`.  
2 - Crée votre compte administrateur.  
3 - `Set up your media libraries`  
Ici vous allez mapper le type de contenu au dossier sur lequel se trouve vos fichiers multimédia (le chemin à l'intérieur du LXC).
Pour mon cas j'ai :

| Content type | Display Name | Folder |
|---|---|---|
| Shows | Anime | `/mnt/synology/Media/Anime` |
| Shows | Series | `/mnt/synology/Media/Series` |
| Movies | Movies | `/mnt/synology/Media/Movies` |

4 - `Preferred Metadata Language`
Choisissez la langue qui vous convient pour les metadata.  
5 - `Networking Settings`
Laissez les options par défaut.  
6 - L'assistant de configuration est terminé.  
7 - Après un scan des librairies vous devriez les voir sur la page d'accueil de Jellyfin  
8 - Dès que vous déposerez un film, série dans le dossier correspondant sur Synology vous le verrez apparaitre sur Jellyfin (faire un reload de la librairie pour un refresh immédiat)

<img width="960" height="361" alt="image" src="https://github.com/user-attachments/assets/6898398d-3594-472a-84f1-7c038dd59437" />

Bon visionnage ! 

Notes : 
- Mon tutoriel et explications peuvent toujours comporter des erreurs ou incohérences car mon setup n'est pas parfait. Mais je cherche toujours à l'améliorer.
- Il est prévu que je passe mes partages réseaux en NFS car ils sont actuellement en CIFS. Le NFS permet d'obtenir notamment de meilleures performances réseaux.
