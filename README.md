# 🌐 My self-Hosting journey : reprendre le contrôle

Hello ! Depuis 8 ans j'auto-héberge mes propres services et j'ai décidé de lancer ce nouveau repo afin de partager mes configurations, mes retours d'expérience et des tutoriels pas à pas pour déployer et administrer vos propres services en auto-hébergement (*self-hosting*). 

> ⚠️ Disclaimer : 
>- Ce guide n'est probablement pas exempt d'erreurs, mon infra reste perfectible, et je l'améliore au fil du temps.
>- Le fond technique vient de moi, je me sers de l'IA pour la relecture et la clarté de certaines explications.

## 🎯 Pourquoi le self-hosting ?

Marre de voir mes données personnelles éparpillées chez les GAFAM et d'assister impuissant à l'augmentation constante des abonnements numériques (Netflix, Spotify etc...). J'ai décidé de fermer la porte aux intermédiaires pour trois raisons majeures :

*   **Souveraineté des données :** Je veux être le seul et unique maître de mes données privées (photos, historiques de déplacement, musiques, documents). Plus de tracking, plus de profilage publicitaire.
*   **Indépendance financière & technique :** Remplacer les abonnements cloud par une infrastructure maison rentabilisée sur le long terme, tout en apprenant comment fonctionnent réellement les outils que j'utilise au quotidien.
*   **Soutien à l'Open Source (FOSS) :** Je privilégie exclusivement des logiciels libres et open source. Cela me garantit la transparence du code, la pérennité des outils grâce à la communauté et le respect fondamental de ma vie privée.

Le but que je cherche à atteindre n'est pas forcément de disparaitre du radar complètement mais du moins d'entamer cette démarche d'indépendance et essayer de réduire mon exposition au GAFAM. 

---

## Ma Stack & tutoriels disponibles

Voici mon setup hardware : 

• Synology DS218+ (Raid1 2*8Tb) - Mon NAS de stockage.  
• Raspberry Pi 2B - OS: MotionEye - Caméra de surveillance.  
• Mini PC Beelink EQ12 N100 32Gb RAM - Mon serveur principal sous Proxmox VE 9.2.3.  
• Mini PC N150 16Gb RAM - Mon serveur de sauvegarde sous Proxmox Backup Server 3.4.0.  

Voici les principaux services que j'héberge et leurs guides de déploiement (basés soit sur Docker soit LXC) dans ce repo :

| Service | Catégorie | Alternative à... | Tutoriel | Licence | Repo officiel
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Jellyfin** | Streaming Vidéo | Netflix, Prime Video | 📝[Voir le guide](./jellyfin) | GPL-2.0 | [Repo officiel](https://github.com/jellyfin/jellyfin)  
| **Navidrome** | Streaming Audio | Spotify, Deezer, Apple Music | 📝 A VENIR | AGPL-3.0 | [Repo officiel](https://github.com/navidrome/navidrome) 
| **Dawarich** | Tracking de localisation | Google Maps Timeline (Chronologie) | 📝 A VENIR | AGPL-3.0  | [Repo officiel](https://github.com/Freika/dawarich) 
| **Immich** | Gestionnaire de photos/vidéos | Google Photos | 📝 A VENIR | AGPL-3.0 | [Repo officiel](https://github.com/immich-app/immich) 
| **Paperless-ngx** | Gestionnaire électronique de documents | Google Drive | 📝 A VENIR  | GPL-3.0 | [Repo officiel](https://github.com/paperless-ngx/paperless-ngx) 
| **Vaultwarden** | Gestionnaire de mots de passe | 1Password, Lastpass | 📝 A VENIR  | AGPL-3.0 | [Repo officiel](https://github.com/dani-garcia/vaultwarden) 

> 💡 *D'autres services viendront enrichir cette liste.*

Contact : github@afcm.info
