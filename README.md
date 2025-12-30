# ArcadeBox

ArcadeBox est un concept permettant de transformer un PC en **borne arcade CRT 15 kHz** sous Windows 7 pro.  
Le projet repose sur une configuration minimale, optimisée pour l’émulation et l’expérience arcade authentique.

---

## Objectif

- Utiliser un PC équipé d’une carte graphique **ATI** et des drivers [**CRT Emudriver & CRT Tools 2.0**](http://geedorah.com/eiusdemmodi/forum/viewtopic.php?id=295) pour générer un signal vidéo 15 kHz compatible avec un téléviseur ou moniteur CRT.  
- Alléger Windows 7 en désactivant les services et programmes non essentiels.  
- Sécuriser le système avec **Malwarebytes Windows Firewall Control (WFC)** pour contrôler toutes les connexions entrantes et sortantes.  
- Utiliser uniquement des programmes portables pour limiter l’impact sur le système.  
- Permettre une gestion distante via **TeamViewer** et un échange de fichiers par **client FTP**.  

---

## Logiciels utilisés

- **Frontend** : [Pegasus](https://pegasus-frontend.org/)  
- **Émulation** : [RetroArch](https://www.retroarch.com/)  
- **Menu système** : développé avec [AutoHotKey v2](https://www.autohotkey.com/docs/v2/), remplaçant l’explorer de Windows par une interface minimaliste et adaptée à l’usage arcade.
- **Pare-feu** : [Malwarebytes Windows Firewall Control (WFC)](https://www.binisoft.org/wfc)
- **Gestion à distance** : **Remote Desktop Protocol (RDP)** natif de Windows 7 Pro
- **Serveur FTP** : [Xlight FTP Server](https://www.xlightftpd.com/)

---

## Gestion des comptes Windows

ArcadeBox repose sur deux comptes distincts :

- **Admin** : compte administrateur protégé par mot de passe.  
- **ArcadeBox** : compte invité configuré en **autologin** au démarrage de la borne arcade.  

Cette séparation garantit la sécurité et la simplicité d’utilisation pour les joueurs.

---

## Fonctionnalités principales

- Signal vidéo 15 kHz pour CRT via CRT Emudriver.  
- Système Windows 7 allégé et sécurisé.  
- Interface minimaliste pensée pour une borne arcade.  
- Gestion distante et transfert de fichiers simplifiés.  
- Expérience utilisateur fluide et adaptée aux enfants comme aux passionnés.  

---

## Installation

1. Installer Windows 7 sur un PC équipé d’une carte ATI compatible.  
2. Installer **CRT Emudriver & CRT Tools 2.0**.  
3. Configurer les comptes Windows (Admin + ArcadeBox).  
4. Installer Pegasus et RetroArch en version portable.  
5. Déployer le menu système AutoHotKey compilé en `.exe`.  
6. Configurer Malwarebytes WFC, TeamViewer et FTP selon vos besoins.  

---

## Licence
  
- Le **code source** d’ArcadeBox est distribué sous licence [GNU GPL v3](LICENSE).  
  Vous êtes libres de l’utiliser, le modifier et le redistribuer, à condition que toute version dérivée reste sous la même licence et que le code source soit rendu disponible.

- La **documentation et les guides** sont publiés sous licence [Creative Commons BY-NC-SA 4.0](LICENSE-DOCS).  
  Vous pouvez les partager et les adapter, mais **pas pour un usage commercial**, et vous devez conserver la même licence pour vos dérivés.
