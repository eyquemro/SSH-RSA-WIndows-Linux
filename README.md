# Connexion SSH RSA de Linux vers Windows

## Introduction
Ce guide vous montre comment configurer une connexion SSH sécurisée de votre machine Linux vers un système Windows en utilisant l'authentification par clé RSA, ainsi que la gestion des droits d'accès sur les fichiers et dossiers.

## Prérequis
- **Linux** : Accès à un terminal.
- **Windows** : OpenSSH Server installé et configuré.
- Accès administrateur sur le système Windows.

## Étapes à suivre

### 1. Installer OpenSSH sur Windows
- Ouvrez les **Paramètres** de Windows.
- Allez dans **Applications** > **Fonctionnalités facultatives**.
- Cliquez sur **Ajouter une fonctionnalité**.
- Recherchez **OpenSSH Server** et installez-le.

### 2. Démarrer le serveur OpenSSH
- Ouvrez le **Panneau de configuration**.
- Accédez à **Outils d'administration** > **Services**.
- Recherchez **OpenSSH SSH Server**, faites un clic droit dessus, puis sélectionnez **Démarrer**.

### 3. Générer une clé RSA sur Linux
- Ouvrez un terminal sur votre machine Linux.
- Exécutez la commande suivante pour générer une paire de clés RSA :

  ```bash
  ssh-keygen -t rsa -b 4096
  ```

- Appuyez sur **Entrée** pour accepter le chemin par défaut (généralement `~/.ssh/id_rsa`).
- Vous pouvez choisir d'entrer une phrase de passe pour plus de sécurité.

### 4. Copier la clé publique sur Windows
Pour copier la clé publique, vous pouvez utiliser la commande `ssh-copy-id` (si disponible) :

```bash
ssh-copy-id user@windows_host
```

Remplacez `user` par votre nom d'utilisateur Windows et `windows_host` par l'adresse IP ou le nom d'hôte de votre machine Windows.

Si `ssh-copy-id` n'est pas disponible, copiez la clé manuellement :

1. Affichez votre clé publique avec la commande :

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

2. Connectez-vous à votre machine Windows, puis ouvrez le fichier `authorized_keys` :

   ```powershell
   notepad C:\ProgramData\.ssh\authorized_keys 
   ```

   (Si le fichier n'existe pas, créez-le.)

3. Collez la clé publique dans ce fichier et enregistrez-le.

### 5. Gérer les droits d'accès sur le fichier `authorized_keys`
Il est essentiel de définir les bonnes permissions sur le fichier `authorized_keys` pour garantir la sécurité. Utilisez les commandes PowerShell suivantes :

```bash
icacls C:\Users\Romain\.ssh\authorized_keys /inheritance:r /grant "NT AUTHORITY\SYSTEM:F" /grant "Romain:F"
```

#### Explication des paramètres :
- **C:\Users\Romain\.ssh\authorized_keys** : chemin vers ton fichier `authorized_keys`.
- **/inheritance:r** : supprime l'héritage des permissions du dossier parent.
- **/grant "NT AUTHORITY\SYSTEM:F"** : accorde un contrôle total au compte système.
- **/grant "Romain:F"** : accorde un contrôle total à l'utilisateur `Romain`.

Après avoir appliqué cette commande, tu devrais avoir les permissions correctes sur le fichier `authorized_keys`, ce qui te permettra de te connecter via SSH en utilisant ta clé RSA.

### 6. Fichier de Configuration `sshd_config`

Le fichier de configuration se trouve généralement à l'emplacement suivant :

```
C:\ProgramData\ssh\sshd_config
```

#### Étapes de Configuration

#### 1. Ouvrir le fichier de configuration
Ouvrez le fichier `sshd_config` avec un éditeur de texte en tant qu'administrateur. Par exemple, vous pouvez utiliser Notepad :

```powershell
notepad C:\ProgramData\ssh\sshd_config
```

#### 2. Configuration des Ports
- **Port** : Le serveur écoute par défaut sur le port 22. Si vous souhaitez changer le port, décommentez et modifiez la ligne suivante :

```plaintext
Port 22
```

- **ListenAddress** : Configurez l'adresse d'écoute. Pour écouter sur toutes les interfaces, laissez comme suit :

```plaintext
ListenAddress 0.0.0.0
```

#### 3. Clés d'Hôte
Assurez-vous que les chemins des fichiers de clés d'hôte sont corrects. Par défaut, ils sont définis comme suit :

```plaintext
HostKey __PROGRAMDATA__/ssh/ssh_host_rsa_key
HostKey __PROGRAMDATA__/ssh/ssh_host_dsa_key
HostKey __PROGRAMDATA__/ssh/ssh_host_ecdsa_key
HostKey __PROGRAMDATA__/ssh/ssh_host_ed25519_key
```

#### 4. Paramètres d'Authentification
- **PubkeyAuthentication** : Assurez-vous que l'authentification par clé publique est activée :

```plaintext
PubkeyAuthentication yes
```

- **PasswordAuthentication** : Si vous souhaitez désactiver l'authentification par mot de passe, laissez la ligne suivante :

```plaintext
PasswordAuthentication no
```

- **PermitRootLogin** : Pour interdire la connexion en tant que root, utilisez :

```plaintext
PermitRootLogin prohibit-password
```

#### 5. Gestion des Sessions
- **MaxAuthTries** : Limitez le nombre de tentatives d'authentification :

```plaintext
MaxAuthTries 6
```

- **MaxSessions** : Limitez le nombre de sessions simultanées :

```plaintext
MaxSessions 10
```

#### 6. Fichier des Clés Autorisées
Vérifiez que le chemin du fichier des clés autorisées est correct :

```plaintext
AuthorizedKeysFile .ssh/authorized_keys
```

#### 7. Configuration du Sous-système
Assurez-vous que le sous-système SFTP est configuré pour utiliser le serveur SFTP :

```plaintext
Subsystem sftp sftp-server.exe
```

#### 8. Sauvegarder et Fermer
Après avoir effectué les modifications, sauvegardez le fichier et fermez l'éditeur.

#### 9. Redémarrer le Service SSH
Pour appliquer les modifications, redémarrez le service OpenSSH SSH Server. Ouvrez PowerShell en tant qu'administrateur et exécutez :

```powershell
Restart-Service sshd
```

### 7. Établir la connexion SSH
Pour vous connecter à votre machine Windows depuis Linux, exécutez la commande suivante :

```bash
ssh user@windows_host
```

### Conclusion
Vous avez maintenant configuré une connexion SSH sécurisée de votre machine Linux vers Windows en utilisant l'authentification par clé RSA. Vous avez également défini les droits d'accès appropriés sur le fichier `authorized_keys` pour garantir la sécurité. Si vous rencontrez des problèmes, assurez-vous que le service OpenSSH est en cours d'exécution sur Windows et que le pare-feu autorise le port 22.
