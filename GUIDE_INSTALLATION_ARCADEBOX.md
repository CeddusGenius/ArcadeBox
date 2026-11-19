# GUIDE MAÎTRE D'INSTALLATION, D'OPTIMISATION ET DE SÉCURISATION — ARCADEBOX 15 KHZ

* **Système :** Windows 10 IoT Enterprise LTSC 2021 x64 (Build 21H2 — Support jusqu'en 2032)
* **Matériel cible :** PC dédiés (8 Go ou 16 Go de RAM), carte graphique ATI (CRT Emudriver 15 kHz), émulation jusqu'à la Wii et jeux PC standalone
* **Structure disque :** `C:` (Système optimisé à **30 Go pile**) / `D:` (`ArcadeBox`, `Scripts` et `user` portables)
* **Comptes Windows :** **`Admin`** (Administrateur protégé par mot de passe) + **`ArcadeBox`** (Compte joueur standard en Autologin)

---

## PARTIE 1 : PARTITIONNEMENT ET INSTALLATION DU SYSTÈME

### 1.1 Création des partitions (Chiffres ronds NTFS)
Lors de l'installation de Windows, utilisez la formule `(Taille en Go × 1024) + 8 Mo` pour obtenir des valeurs exactes dans l'explorateur :
* **Partition Système (`C:`) :** Saisir **30728 Mo** (pour obtenir **30 Go pile**).
* **Partition Data (`D:`) :** Allouer tout le reste de l'espace du SSD.

### 1.2 Installation hors-ligne et création des comptes
1. Installer **Windows 10 Enterprise LTSC 2021** sur `C:` **SANS CONNEXION INTERNET** (débrancher le câble Ethernet / ignorer le Wi-Fi).
2. Créer le premier compte nommé **`Admin`** avec un mot de passe fort.
3. Une fois sur le bureau, ouvrir **Paramètres > Comptes > Famille et autres utilisateurs** (ou `netplwiz`) et créer un second compte local **Standard** (non administrateur) nommé **`ArcadeBox`** (sans mot de passe, pour l'autologin joueur).

### 1.3 Transformation en Windows 10 IoT Enterprise et Activation
1. Sur la session **`Admin`**, ouvrir l'**Invite de commandes (`cmd`) en tant qu'administrateur**.
2. Injecter la clé IoT pour basculer instantanément de l'édition Enterprise vers **IoT Enterprise** :
   ```cmd
   slmgr /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
   ```
3. Connecter la borne à Internet, vérifier l'activation dans **Paramètres > Mise à jour et sécurité > Activation**, puis lancer **Windows Update** jusqu'à ce que toutes les mises à jour cumulatives soient installées.

### 1.4 Pilotes et Runtimes indispensables
* Installer **CRT Emudriver & CRT Tools 2.0** (pour le signal vidéo 15 kHz).
* Installer le pack complet **Visual C++ Redistributables (All-in-One)**, **DirectX End-User Runtimes (June 2010)** et **OpenAL**.
* Activer **.NET Framework 3.5** via *Panneau de configuration > Programmes > Activer ou désactiver des fonctionnalités Windows*.

---

## PARTIE 2 : ARCHITECTURE PORTABLE ET REDIRECTION SUR `D:`

### 2.1 Arborescence sur le lecteur `D:`
Créer la structure suivante à la racine de `D:` :
* `D:\ArcadeBox\` *(Pegasus-FE, RetroArch, GroovyMAME, ROMs, médias, jeux PC standalone, et dossier `Tools\` pour UltraVNC et Xlight)*
* `D:\Scripts\` *(Vos fichiers `.bat` modulaires détaillés en Partie 3)*
* `D:\user\` *(Contenant les sous-dossiers vides : `Documents`, `Images`, `Videos`, `Musique`, `Telechargements`, `Bureau` et `saved games`)*

### 2.2 Déplacement des dossiers utilisateur vers `D:\user\`
Sur la session **`ArcadeBox`** (et **`Admin`** si souhaité) :
1. Faire un clic droit sur chaque bibliothèque (`Documents`, `Images`, `Vidéos`, `Musique`, `Téléchargements`, `Bureau`) > **Propriétés** > onglet **Emplacement** > **Déplacer...** vers le sous-dossier correspondant dans `D:\user\...`. *(Ne jamais pointer deux bibliothèques vers le même dossier).*
2. Exécuter le **Script 1** (ci-dessous) pour lier `Saved Games` via une jonction NTFS.
---

## PARTIE 3 : CATALOGUE DES SCRIPTS MODULAIRES (`D:\Scripts`)

Créez un fichier `.bat` séparé pour chaque fonction dans `D:\Scripts`. Tous les scripts doivent être lancés par clic droit > **Exécuter en tant qu'administrateur**.

### A. Scripts d'optimisation disque et système (À lancer après Windows Update)

* **`1_Jonction_SavedGames.bat`** *(Redirige les sauvegardes de jeux PC vers `D:\user\saved games`)*
  ```bat
  @echo off
  if not exist "D:\user\saved games" mkdir "D:\user\saved games"
  xcopy "%USERPROFILE%\Saved Games\*" "D:\user\saved games\" /E /H /K /Y
  rmdir /s /q "%USERPROFILE%\Saved Games"
  mklink /J "%USERPROFILE%\Saved Games" "D:\user\saved games"
  pause
  ```

* **`2_Desactiver_Hibernation.bat`** *(Supprime `hiberfil.sys` et libère ~3,2 Go sur 8 Go RAM ou ~6,4 Go sur 16 Go RAM)*
  ```bat
  @echo off
  powercfg -h off
  pause
  ```

* **`3_Regler_Pagefile_2Go.bat`** *(Fixe la mémoire virtuelle à 2048 Mo sur 8 Go et 16 Go RAM)*
  ```bat
  @echo off
  wmic computersystem where name="%computername%" set AutomaticManagedPagefile=False
  wmic pagefileset where name="C:\\pagefile.sys" set InitialSize=2048,MaximumSize=2048
  echo Fichier d'echange fixe a 2048 Mo. Redemarrage requis.
  pause
  ```

* **`4_Nettoyage_WinSxS.bat`** *(Compresse et purge les anciens composants après les mises à jour)*
  ```bat
  @echo off
  Dism.exe /online /Cleanup-Image /StartComponentCleanup /ResetBase
  pause
  ```

* **`5_Vider_Temp.bat`** *(Vide les dossiers Temp utilisateur, Windows et cache Windows Update)*
  ```bat
  @echo off
  net stop wuauserv
  del /q /f /s "%TEMP%\*"
  del /q /f /s "C:\Windows\Temp\*"
  del /q /f /s "C:\Windows\SoftwareDistribution\Download\*"
  net start wuauserv
  pause
  ```
  *(Compléter ensuite par un passage de `cleanmgr` > **Nettoyer les fichiers système** en cochant toutes les cases).*

### B. Scripts d'immersion Console / Arcade (À lancer sur le profil `ArcadeBox`)

* **`6_Desactiver_Ecran_Verrouillage.bat`** *(Supprime l'écran de verrouillage avant l'ouverture de session)*
  ```bat
  @echo off
  reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Personalization" /v NoLockScreen /t REG_DWORD /d 1 /f
  pause
  ```

* **`7_Desactiver_Notifications.bat`** *(Coupe les pop-ups et le Centre de maintenance)*
  ```bat
  @echo off
  reg add "HKCU\SOFTWARE\Policies\Microsoft\Windows\Explorer" /v DisableNotificationCenter /t REG_DWORD /d 1 /f
  reg add "HKCU\Microsoft\Windows\CurrentVersion\PushNotifications" /v ToastEnabled /t REG_DWORD /d 0 /f
  pause
  ```

* **`8_Masquer_Bureau_BarreTaches.bat`** *(Cache toutes les icônes du bureau et masque automatiquement la barre des tâches)*
  ```bat
  @echo off
  reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v HideIcons /t REG_DWORD /d 1 /f
  powershell -command "&{\(p='HKCU:SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\StuckRects3';\)v=(Get-ItemProperty -Path p).Settings;v[8]=3;&Set-ItemProperty -Path p -Name Settings -Value v}"
  taskkill /f /im explorer.exe
  start explorer.exe
  pause
  ```

* **`8b_Afficher_Corbeille_Explorateur.bat`** *(Ajoute la Corbeille dans le volet latéral gauche de l'explorateur)*
  ```bat
  @echo off
  reg add "HKCU\Software\Classes\CLSID\{645FF040-5081-101B-9F08-00AA002F954E}" /v System.IsPinnedToNameSpaceTree /t REG_DWORD /d 1 /f
  taskkill /f /im explorer.exe
  start explorer.exe
  pause
  ```

* **`9_Desactiver_Touches_Remanentes.bat`** *(Évite les fenêtres Windows lors d'appuis répétés sur Maj/Ctrl en jeu)*
  ```bat
  @echo off
  reg add "HKCU\Control Panel\Accessibility\StickyKeys" /v Flags /t REG_SZ /d 506 /f
  reg add "HKCU\Control Panel\Accessibility\ToggleKeys" /v Flags /t REG_SZ /d 58 /f
  reg add "HKCU\Control Panel\Accessibility\Keyboard Response" /v Flags /t REG_SZ /d 122 /f
  pause
  ```

* **`10_Desactiver_Veille_Ecran.bat`** *(Force le mode Performances élevées et empêche l'extinction de l'écran)*
  ```bat
  @echo off
  powercfg /setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
  powercfg /change monitor-timeout-ac 0
  powercfg /change standby-timeout-ac 0
  powercfg /change disk-timeout-ac 0
  pause
  ```

### C. Scripts de restauration (Pour réutilisation sur un PC classique)

* **`R1_Restaurer_Bureau.bat`**
  ```bat
  @echo off
  reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v HideIcons /t REG_DWORD /d 0 /f
  taskkill /f /im explorer.exe
  start explorer.exe
  pause
  ```

* **`R2_Reactiver_Notifications.bat`**
  ```bat
  @echo off
  reg add "HKCU\SOFTWARE\Policies\Microsoft\Windows\Explorer" /v DisableNotificationCenter /t REG_DWORD /d 0 /f
  reg add "HKCU\Microsoft\Windows\CurrentVersion\PushNotifications" /v ToastEnabled /t REG_DWORD /d 1 /f
  pause
  ```
---

## PARTIE 4 : NETTOYAGE ET SÉCURISATION DU SYSTÈME (Sur la borne)

*(À exécuter depuis la session **`Admin`** dans **PowerShell Administrateur**).*

### 4.1 Suppression de Windows Media Player et blocage de réinstallation
```powershell
dism /online /remove-capability /capabilityname:Media.WindowsMediaPlayer~~~~0.0.12.0
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Servicing" -Name "RepairContentServerSource" -PropertyType DWord -Value 2 -Force
```
*(Redémarrer le PC après l'exécution).*

### 4.2 Suppression totale de la télémétrie et des services de tracking
```powershell
$RegistryPaths = @(
    "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection",
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection"
)
foreach ($path in $RegistryPaths) {
    if (-not (Test-Path $path)) { New-Item $path -Force | Out-Null }
    New-ItemProperty -Path $path -Name "AllowTelemetry" -PropertyType DWord -Value 0 -Force | Out-Null
    New-ItemProperty -Path $path -Name "MaxTelemetryAllowed" -PropertyType DWord -Value 0 -Force | Out-Null
}

New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppCompat" -Name "AITEnable" -PropertyType DWord -Value 0 -Force | Out-Null
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppCompat" -Name "DisableInventory" -PropertyType DWord -Value 1 -Force | Out-Null

$Services = @("DiagTrack", "dmwappushservice", "WerSvc", "PcaSvc")
foreach ($service in $Services) {
    if (Get-Service -Name $service -ErrorAction SilentlyContinue) {
        Stop-Service -Name $service -Force -ErrorAction SilentlyContinue
        Set-Service -Name $service -StartupType Disabled
    }
}

$Tasks = @(
    "\Microsoft\Windows\Application Experience\Microsoft Compatibility Appraiser",
    "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
    "\Microsoft\Windows\Application Experience\StartupAppTask",
    "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator"
)
foreach ($task in $Tasks) {
    Disable-ScheduledTask -TaskName $task -ErrorAction SilentlyContinue | Out-Null
}
```

---

## PARTIE 5 : ACCÈS DISTANT SÉCURISÉ (SSH + ULTRAVNC 15 KHZ + XLIGHT FTPS)

### 5.1 Serveur OpenSSH natif (Tunnel chiffré pour UltraVNC)
1. **Installer le service (PowerShell Admin) :**
   ```powershell
   Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
   Set-Service -Name sshd -StartupType Automatic
   Start-Service sshd
   ```
2. **Configurer `C:\ProgramData\ssh\sshd_config` :**
   ```text
   Port 54322
   PasswordAuthentication no
   PubkeyAuthentication yes
   ForceCommand internal-sftp
   AllowTcpForwarding yes
   X11Forwarding no
   ```
3. **Déployer la clé publique ED25519 :**
   * Sur votre **PC d'administration**, générer la clé via `ssh-keygen -t ed25519 -C "admin-arcadebox"`.
   * Copier le contenu de `C:\Users\[VotreNom]\.ssh\id_ed25519.pub` dans `C:\ProgramData\ssh\administrators_authorized_keys` sur la borne.
   * Verrouiller les droits NTFS sur la borne (PowerShell Admin) :
     ```powershell
     $KeyPath = "C:\ProgramData\ssh\administrators_authorized_keys"
     icacls $KeyPath /inheritance:r /grant "NT AUTHORITY\SYSTEM:(F)" /grant "BUILTIN\Administrators:(F)"
     Restart-Service sshd
     ```

### 5.2 UltraVNC Portable (Contrôle à distance sans perte de synchro 15 kHz)
1. Placer **UltraVNC Server** (`winvnc.exe`) dans `D:\ArcadeBox\Tools\UltraVNC\`.
2. Configurer `ultravnc.ini` :
   * **Port :** `5900` | **Mot de passe VNC** défini.
   * **LoopbackOnly :** Cocher **Allow Loopback Connections** (`LoopbackOnly=1`) pour interdire tout accès direct hors tunnel SSH.
   * Désactiver toute option modifiant la résolution ou le fond d'écran afin de préserver le signal **CRT Emudriver 240p/288p**.

### 5.3 Xlight FTP Server Portable (Transfert rapide des ROMs et Assets en FTPS)
1. Placer **Xlight FTP Server** (`xlight.exe`) dans `D:\ArcadeBox\Tools\Xlight\`.
2. Configurer le serveur virtuel :
   * **Port d'écoute :** `2121`.
   * **Chiffrement SSL/TLS :** Générer un certificat auto-signé dans Xlight et activer **FTPS Explicite (Auth TLS)** obligatoire.
   * **Plage de ports passifs :** Fixer de **50000 à 50020**.
   * **Utilisateur FTP :** Créer un compte `arcadeftp` avec mot de passe fort et verrouiller son répertoire racine (`Home Directory`) sur `D:\ArcadeBox`.

### 5.4 Règles du pare-feu Malwarebytes WFC (Sur la borne)
Passer WFC en mode **Medium Filtering** et créer uniquement ces règles :
* **ENTRÉE :** Autoriser `C:\Windows\System32\OpenSSH\sshd.exe` sur **TCP 54322** (restreint à votre IP / réseau local).
* **ENTRÉE :** Autoriser `D:\ArcadeBox\Tools\Xlight\xlight.exe` sur **TCP 2121** et **TCP 50000-50020** (restreint à votre réseau local).
* **ENTRÉE :** Bloquer `winvnc.exe` sur le réseau externe (il écoute uniquement sur `127.0.0.1:5900`).
* **SORTIE :** Autoriser `svchost.exe` (`wuauserv`) sur **TCP 80, 443** uniquement lors des mises à jour manuelles.

---

## PARTIE 6 : DÉMARRAGE AUTOMATIQUE ET SCRIPTS PC CLIENT

### 6.1 Démarrage automatique de la borne (Session `ArcadeBox`)
1. Configurer l'**Autologin** sur le compte standard **`ArcadeBox`** (via `netplwiz`, en décochant *"Les utilisateurs doivent entrer un nom d'utilisateur et un mot de passe..."*).
2. Sur la session **`ArcadeBox`**, appuyer sur **Windows + R**, taper `shell:startup` et placer les raccourcis vers :
   * Votre front-end **Pegasus-FE** (`D:\ArcadeBox\Pegasus\pegasus-fe.exe`) et votre menu système **AutoHotKey v2**.
   * `winvnc.exe` et `xlight.exe` (si vous souhaitez qu'ils soient actifs dès l'allumage de la borne).

### 6.2 Scripts prêts à l'emploi sur votre PC d'administration

1. **Configuration PuTTY (Tunnel UltraVNC) :**
   * `Host Name` = `[IP_DE_LA_BORNE]` | `Port` = `54322`
   * `SSH > Auth > Credentials` : charger `id_ed25519.ppk`.
   * `SSH > Tunnels` : Source port `55900` -> Destination `127.0.0.1:5900` (**Add**).
   * `SSH` : cocher **Don't start a shell or command at all**. Enregistrer sous `Tunnel_VNC_ArcadeBox`.

2. **Lanceur Prise en main 15 kHz (`VNC_ArcadeBox.bat`) :**
   ```bat
   @echo off
   title Prise en main ArcadeBox 15kHz (UltraVNC + SSH)
   start "" "C:\Program Files\PuTTY\putty.exe" -load "Tunnel_VNC_ArcadeBox"
   timeout /t 2 /nobreak >nul
   start "" "C:\Program Files\uvnc bvba\UltraVNC\vncviewer.exe" 127.0.0.1::55900
   exit
   ```

3. **Synchronisation rapide des ROMs/Assets via Xlight FTPS (`transfert_xlight.txt` + `WinSCP_Xlight_Sync.bat`) :**
   * **`transfert_xlight.txt` :**
     ```text
     option batch abort
     option confirm off
     open ftpes://arcadeftp:[MotDePasse]@[IP_DE_LA_BORNE]:2121/ -certificate="*"
     synchronize remote "C:\Mon_Projet_ArcadeBox\ROMS" "/ROMS"
     close
     exit
     ```
   * **`WinSCP_Xlight_Sync.bat` :**
     ```bat
     @echo off
     title Synchronisation FTPS Xlight - ArcadeBox
     "C:\Program Files (x86)\WinSCP\WinSCP.exe" /log="C:\Users\%USERNAME%\Desktop\winscp_xlight.log" /script="C:\Users\%USERNAME%\Desktop\transfert_xlight.txt"
     if %errorlevel% neq 0 (
         echo [ERREUR] Echec du transfert FTPS. Verifiez Xlight et winscp_xlight.log.
         pause
         exit
     )
     echo Synchronisation terminee avec succes !
     timeout /t 3
     exit
     ```

---

## LICENCE
* **Documentation & Guides :** Publiés sous licence **Creative Commons BY-NC-SA 4.0** (ArcadeBox 15kHz Project by CeddusGenius).
* **Scripts `.bat` & `.ps1` :** Distribués sous licence **GNU GPL v3**.
