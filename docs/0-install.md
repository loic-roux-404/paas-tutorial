# Introduction et installation

---

L'objectif de ce tutoriel est de vous permettre de créer sur une petite machine ou sur un serveur personnel un PaaS (Platform as a service). Un PaaS permet de déployer des applications en microservices. Celui-ci sera basé sur [kubernetes](https://kubernetes.io/fr/) pour la conteneurisation et [Kubeapps](https://kubeapps.dev/) pour l'interface de déploiement.

L'optique de cet outillage suivra :
- le principe **d'immutable infrastructure** avec l'idée de recréer plutôt que de mettre à jour. Ainsi nous aurons recour à des iso linux déjà prêt pour déployer la plateforme **kubernetes** / **kubeapps** directement sur un serveur.

- Le principe **d'infrastructure as code** (IaC) en gardant toutes la spécification de notre infrastructure dans des configurations et scripts. On utilisera également des tests basiques de nos configurations.

Pour cela nous ferons appel à un socle technique composé de :

- l'outil [`k3s`](https://k3s.io/) qui simplifie l'installation de kubernetes sur des machines ARM tout en restant compatible avec les architectures classiques X64. Il fourni par défaut des pods (containers en execution) pour inclure des fonctionnalités souvent recherchés sur ce type de configuration edge computing. (reverse proxy, configuration DNS...)
- [Packer](https://www.packer.io/) pour créer des images iso de machine linux
- [Ansible](https://www.ansible.com/) pour provisioner cette image
- [Azure](https://azure.microsoft.com/fr-fr/) pour nous founir des serveurs accessible en ssh sur lequels nous pourrons mettre en ligne
- [Terraform](https://www.terraform.io/) pour contrôler azure de manière IaC et de déclencher toute la mise en place du PaaS dessus.

## Installation de Docker

Pour rappel l'architecture de base de docker :

![docker architecture](https://docs.docker.com/engine/images/architecture.svg)

> Source documentation docker

et les couches des systèmes de conteneurisation docker et kubernetes :

![docker k8s architecture](images/kube-archi.png)

Pour utilisateurs de **windows** il faut un [**WSL**](https://devblogs.microsoft.com/commandline/a-preview-of-wsl-in-the-microsoft-store-is-now-available/#how-to-install-and-use-wsl-in-the-microsoft-store). 

**Pour WSL** :

Vous utilisez une version de Windows 11 ou supérieure (numéro de version de Windows 22000 ou supérieur).
Vous avez activé le composant optionnel Virtual Machine Platform
Vous pouvez le faire en exécutant : `dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all` dans une invite PowerShell élevée.

Cliquez sur ce [lien] (https://aka.ms/wslstorepage) pour accéder à la page du magasin WSL et cliquez sur Installer pour installer WSL.

Traduit avec www.DeepL.com/Translator (version gratuite)

- Télécharger après avoir suivi cette documentation la distribution linux ``Ubuntu 20.04.5 LTS`` depuis le windows store. 
- **+ Windows terminal bien que pas obligatoire il est très pratique pour accéder au shell**

Ensuite dans vscode installer l'extension wsl `ms-vscode-remote.remote-wsl`.

Laissez-le ensuite finir de s'initialiser.

## Mise à jour de Linux

Ensuite bonne habitude, on met à jour linux :

```bash
apt update && apt upgrade -y
```

Puis activer `systemd` en modifiant `/etc/wsl.conf` dans votre distribution linux :

```sh
echo -e '[boot]\nsystemd=true' >> /etc/wsl.conf
```

Puis redémarrer l'app Ubuntu. Si des problèmes apparaissent encore lancer la commande `wsl --shutdown` depuis un powershell en administrateur avant de lancer le shell WSL.

Ensuite pour finaliser l'installation de docker pour éviter les problèmes de droit avec rancher desktop :

```bash
sudo addgroup --system docker
sudo adduser $USER docker
newgrp docker
# And something needs to be done so $USER always runs in group `docker` on the `Ubuntu` WSL
sudo chown root:docker /var/run/docker.sock
sudo chmod g+w /var/run/docker.sock

```

## Rancher comme alternative à docker desktop

[**Rancher**](https://rancherdesktop.io/) l'alternative mieux configurée et sans soucis de license à docker desktop. Il est portable sur windows et mac et nous permet d'avoir une expérience docker complète et fonctionnelle sur notre machine.

Dans les choix proposés dans la mise en place :
- **Décocher kubernetes**
- Choisissez **dockerd** comme moteur de conteneurisation

Puis dans wsl : 

```sh
sudo systemctl restart docker
```

## Installation de l'environnement python

**Maintenant tout ce que nous allons faire se trouve dans la ligne de commande sur un shell `bash` ou `zsh`**

**Conda** : [docs.conda.io](https://docs.conda.io/en/latest/miniconda.html). Installer simplement avec le setup `.pkg` pour mac.

> Pour Linux et Windows avec WSL utilisez la ligne de commande ci-dessous pour l'installer
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -P /tmp
chmod +x /tmp/Miniconda3-latest-Linux-x86_64.sh
/tmp/Miniconda3-latest-Linux-x86_64.sh -p $HOME/miniconda
```

> Pour arm :
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-aarch64.sh -P /tmp
chmod +x /tmp/Miniconda3-py39_4.12.0-Linux-aarch64.sh
/tmp/Miniconda3-py39_4.12.0-Linux-aarch64.sh -p $HOME/miniconda
```

Veillez à bien accepter toutes les propositions (licence terms, initialize Miniconda3).

Puis lancer `conda init zsh` (ou `bash` si vous préférez)

**Relancer votre shell pour appliquer** (commande `exec $SHELL`)

