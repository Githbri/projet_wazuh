# 📌 Projet Wazuh - SIEM avec Vagrant, MailDev et Suricata

Ce projet met en place une **infrastructure de monitoring de sécurité** basée sur **Wazuh** avec une architecture en **deux machines Vagrant** :
- **ServerBri** : Le serveur Wazuh qui collecte et analyse les logs.
- **AgentBri** : La machine surveillée qui envoie ses logs au serveur Wazuh.

J'ai également intégré **MailDev** pour recevoir des alertes par email.

## 🏗️ Architecture  

### 🔹 **Machines Vagrant**  
| Nom de la VM   | Rôle | Adresse IP |
|---------------|------|------------|
| ServerBri     | Serveur Wazuh (SIEM) |
| AgentBri      | Machine surveillée (Wazuh Agent) |

Les deux machines sont sur un **réseau privé** configuré via **Vagrant**.

---

## 🚀 Installation du serveur Wazuh (ServerBri)  

### 📌 **Installation de Wazuh**  

1️⃣ **Mettre à jour la machine**  

sudo apt update && sudo apt upgrade -y


2️⃣ **Installer les dépendances**  

sudo apt install curl apt-transport-https unzip -y


3️⃣ **Ajouter le dépôt Wazuh**  

curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo apt-key add GPG-KEY-WAZUH
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update


4️⃣ **Installer Wazuh Manager**  

sudo apt install wazuh-manager -y


5️⃣ **Démarrer et activer Wazuh**  

sudo systemctl enable --now wazuh-manager

## 🔍 Installation de l'Agent Wazuh (AgentBri)  

1️⃣ **Mettre à jour la machine**  

sudo apt update && sudo apt upgrade -y

2️⃣ **Ajouter le dépôt Wazuh**  

curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo apt-key add GPG-KEY-WAZUH
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update


3️⃣ **Installer l'agent Wazuh**  

sudo apt install wazuh-agent -y


4️⃣ **Configurer l'agent pour pointer vers le serveur**  

sudo sed -i 's|MANAGER_IP|INSERER VOTRE IP|' /var/ossec/etc/ossec.conf


5️⃣ **Démarrer et activer l'agent**  

sudo systemctl enable --now wazuh-agent


6️⃣ **Enregistrer l'agent sur le serveur**  
Sur **ServerBri**, exécute cette commande :  

sudo /var/ossec/bin/agent-auth -m VOTRE IP


7️⃣ **Redémarrer l'agent pour appliquer la config**  

sudo systemctl restart wazuh-agent


## 📩 Installation de MailDev (Sur ServerBri)  

MailDev permet de recevoir les alertes Wazuh sous forme d'emails.

1️⃣ **Installer Node.js et NPM**  

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs


2️⃣ **Installer MailDev**  

sudo npm install -g maildev


3️⃣ **Lancer MailDev**  

maildev --web 1080 --smtp 1025 &

(Le `&` permet de lancer en arrière-plan.)

4️⃣ **Rendre MailDev persistant au reboot**  
Créer un service systemd :  

sudo nano /etc/systemd/system/maildev.service

Et y mettre :

[Unit]
Description=MailDev
After=network.target

[Service]
ExecStart=/usr/bin/maildev --web 1080 --smtp 1025
Restart=always
User=vagrant

[Install]
WantedBy=multi-user.target

Puis activer le service :

sudo systemctl daemon-reload
sudo systemctl enable --now maildev

## 🛡️ Installation de Suricata (AgentBri)  

**Suricata** est un moteur de détection d'intrusion (IDS) qui analyse le trafic réseau.

### 📌 **Installation de Suricata**
1️⃣ **Ajouter le dépôt Suricata**  

sudo add-apt-repository ppa:oisf/suricata-stable -y
sudo apt update


2️⃣ **Installer Suricata**  

sudo apt install suricata -y


3️⃣ **Vérifier l’installation**  

suricata --build-info


### 📌 **Configuration de Suricata**
1️⃣ **Éditer le fichier de configuration**  

sudo nano /etc/suricata/suricata.yaml

Dans la section `outputs`, activer les logs JSON :
yaml
outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: /var/log/suricata/eve.json

Puis sauvegarder (`CTRL + X`, `Y`, `Entrée`).

2️⃣ **Démarrer et activer Suricata**  

sudo systemctl enable --now suricata


