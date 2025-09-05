# TP1 : Azure first steps 

## Prérequis

### 1. Installation Azure CLI sur Windows

Installation d'azure CLI en ligne de commande sur powershell : 
``` winget install --exact --id Microsoft.AzureCLI ```

Authentification au compte Azure EFREI : 
``` az login ```

Visualisation de la subscription actuelle :
``` az account show ```

### 2. Installation Terraform sur Windows

- Installation de chocolatey sur Powershell :
```  Set-ExecutionPolicy Bypass -Scope Process -Force; `                                             >> [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; ` >> iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1')) ```

- Installation de Terraform avec chocolatey : 
``` choco install terraform ``` 

- Vérification de l'installation de Terraform avec la commande suivante : 
``` terraform -help ```

## Création de la pair de clé SSH

### 1. Choix de l'algorithme de chiffrement

- source fiable qui explique pourquoi on évite RSA désormais (pour les connexions SSH notamment) : ``` http://shaarli.guiguishow.info/?njmgog ```


- source fiable qui recommande un autre algorithme de chiffrement : 
``` https://powerdmarc.com/fr/dkim-ed25519-configuration-guide/ ```

### 2. Génération de la paire de clés

- Voici la ligne de commande qui permet de créer la clé SSH :
``` ssh-keygen -t ed25519 -C "florian.andries@efrei.net" -f $env:USERPROFILE\.ssh\cloud_tp1 ```

- Elle respecte : 
  - le nom "cloud_tp1"
  - Il est situé dans le dossier standard
  - Elle utilise l'algorithme ed25519 choisi à l'étape précédente
  - Elle est protégée par un mot de passe (étape d'après cette ligne de commande)

- Résultat : 
    ``` 
    PS C:\Users\Florian Andries> ssh-keygen -t ed25519 -C "florian.andries@efrei.net" -f $env:USERPROFILE\.ssh\cloud_tp1
    Generating public/private ed25519 key pair.
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in C:\Users\Florian Andries\.ssh\cloud_tp1
    Your public key has been saved in C:\Users\Florian Andries\.ssh\cloud_tp1.pub
    The key fingerprint is:
    SHA256:IdzcwMgAkrQoCMbigdoNHlq9mbuhJW8mWSOpCbWVa4I florian.andries@efrei.net
    The key's randomart image is:
    +--[ED25519 256]--+
    |*+.o.o o.        |
    |O+= ..oo.o       |
    |B*.+ =o + .      |
    |+.+ B  . .       |
    | o + o  S        |
    |E * O            |
    |.o @ +           |
    |o + =            |
    |   +             |
    +----[SHA256]-----+ 
    ```
### 3. Configuration de l'agent SSH de OpenSSH (Powershell) :

- Installation : 
``` Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0 ```

- Démmarage du service : 
```Start-Service ssh-agent ```

- Rendre automatique au démarrage de Windows
```Set-Service -Name ssh-agent -StartupType Automatic```


- Ajout de la clé :
``` ssh-add $env:USERPROFILE\.ssh\cloud_tp1 ```

- Résultat : 
    ```
    PS C:\Users\Florian Andries\.ssh> ssh-add $env:USERPROFILE\.ssh\cloud_tp1
    Enter passphrase for C:\Users\Florian Andries\.ssh\cloud_tp1:
    Identity added: C:\Users\Florian Andries\.ssh\cloud_tp1 (florian.andries@efrei.net)
    ```

- Vérification : 
``` ssh-add -l ```

- Résultat : 
    ```
    PS C:\Users\Florian Andries\.ssh> ssh-add -l
    256 SHA256:IdzcwMgAkrQoCMbigdoNHlq9mbuhJW8mWSOpCbWVa4I florian.andries@efrei.net (ED25519)
    ```

## Spawn des VMs

### 1. Connexion en SSH à la VM

```
PS C:\Users\Florian Andries\.ssh> ssh fandries@20.39.235.112
The authenticity of host '20.39.235.112 (20.39.235.112)' can't be established.
ED25519 key fingerprint is SHA256:1sXD7vpj32SaSOb6qY7Bb/sT4SKG28xG8cFhGK5n3dE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.39.235.112' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.11.0-1018-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Sep  5 10:30:42 UTC 2025

  System load:  0.04              Processes:             156
  Usage of /:   5.6% of 28.02GB   Users logged in:       0
  Memory usage: 1%                IPv4 address for eth0: 172.16.0.4
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

fandries@VM-TP1-FAndries:~$
```

### 2. Création d'une VM avec azure CLI

- Ligne de commande qui permet la création d'une VM en CLI :

    ```
    az vm create -g ResourceGroupFlo -n VM-CREATION-CLI --image Ubuntu2404 --admin-username fandries --size Standard_B1s --ssh-key-value ~/.ssh/cloud_tp1.pub --location francecentral
    ```

- Résultat : 
    ```
    PS C:\Users\Florian Andries> az vm create -g ResourceGroupFlo -n VM-CREATION-CLI --image Ubuntu2404 --admin-username fandries --size Standard_B1s --ssh-key-value ~/.ssh/cloud_tp1.pub --location francecentral
    The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
    Selecting "northeurope" may reduce your costs. The region you've selected may cost more for the same services. You can disable this message in the future with the command "az config set core.display_region_identified=false". Learn more at https://go.microsoft.com/fwlink/?linkid=222571

    {
    "fqdns": "",
    "id": "/subscriptions/f54a3cba-e415-4e49-8dee-dd037e9b4475/resourceGroups/ResourceGroupFlo/providers/Microsoft.Compute/virtualMachines/VM-CREATION-CLI",
    "location": "francecentral",
    "macAddress": "60-45-BD-1A-C1-E4",
    "powerState": "VM running",
    "privateIpAddress": "172.16.0.5",
    "publicIpAddress": "4.211.179.63",
    "resourceGroup": "ResourceGroupFlo"
    }
    ```

### 3. Connexion à la VM précédemment créée

- Preuve de la bonne création de la VM, et de la connexion sans mot de passe : 
    ```
    PS C:\Users\Florian Andries> ssh fandries@4.211.179.63
    The authenticity of host '4.211.179.63 (4.211.179.63)' can't be established.
    ED25519 key fingerprint is SHA256:kQwEE+7s3IXmylxpHpyM1K4CZiJzmAh+xcZZGzvCE5s.
    This key is not known by any other names.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '4.211.179.63' (ED25519) to the list of known hosts.
    Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.11.0-1018-azure x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/pro

    System information as of Fri Sep  5 11:52:40 UTC 2025

    System load:  0.08              Processes:             112
    Usage of /:   5.6% of 28.02GB   Users logged in:       0
    Memory usage: 29%               IPv4 address for eth0: 172.16.0.5
    Swap usage:   0%

    Expanded Security Maintenance for Applications is not enabled.

    0 updates can be applied immediately.

    Enable ESM Apps to receive additional future security updates.
    See https://ubuntu.com/esm or run: sudo pro status


    The list of available updates is more than a week old.
    To check for new updates run: sudo apt update


    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.

    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.

    fandries@VM-CREATION-CLI:~$
    ```

**Preuve de la présence de ...**

- **du service ```walinuxagent.service```** : 
    ```
    fandries@VM-CREATION-CLI:~$ systemctl list-units --type=service | grep walinuxagent.service
    walinuxagent.service                                  loaded active running Azure Linux Agent
    ```

- **du service ```cloud-init.service```** : 
    ```
    fandries@VM-CREATION-CLI:~$ systemctl list-units --type=service | grep cloud-init.service
    cloud-init.service                                    loaded active exited  Cloud-init: Network Stage
    ```