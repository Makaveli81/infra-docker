# 🚀 Infrastructure Docker avec Authentification Unifiée

![Schéma de Principe](schema.png)

📌 **Projet auto-hébergé** permettant de déployer une infrastructure complète avec **Nextcloud**, **LDAP**, **Nginx Proxy Manager**, **Keycloak** et **Poste.IO** via **Docker Compose**.
🔐 **Authentification centralisée** grâce à **Light LDAP** et **Keycloak** pour une gestion unifiée des utilisateurs.
⚡ **Facile à installer et à gérer sur un serveur Docker ou Portainer.**

---

## 📖 Table des matières
1. [Introduction](#introduction)
2. [Services Déployés](#services-déployés)
3. [Installation](#installation)
   - [Prérequis](#prérequis)
   - [Déploiement](#déploiement)
4. [Configuration de l'Authentification Unifiée](#-configuration-de-lauthentification-unifiée)
   - [Création du Realm dans Keycloak](#création-du-realm-dans-keycloak)
   - [Intégration de light LDAP dans Keycloak](#intégration-de-light-ldap-dans-keycloak)
   - [Intégration de Light LDAP dans Nextcloud](#intégration-de-light-ldap-dans-nextcloud)
   - [Intégration de Nextcloud avec Keycloak](#intégration-de-nextcloud-avec-keycloak)
5. [Configuration de Poste.IO](#configuration-de-posteio)
   - [Configuration DNS](#configuration-dns)
   - [Paramétrage de Poste.IO](#configuration-de-posteio)
6. [Sources](#-sources)
7. [Licence](#licence)


---

## Introduction
### Pourquoi ce projet ?
💡 **Problème** : Gérer plusieurs services auto-hébergés avec **une authentification unique** peut être complexe.
🚀 **Solution** : Ce projet **simplifie** la gestion avec **Keycloak** et **LDAP**, tout en fournissant une **infrastructure clé en main**.

---

## Services Déployés
| Service                 | Rôle | Accès par défaut |
|-------------------------|------|-----------------|
| **Nextcloud**           | Cloud personnel | `http://localhost:8080` |
| **Light LDAP (LLDAP)**  | Gestion des utilisateurs | `ldap://lightldap:17170` |
| **Nginx Proxy Manager** | Reverse Proxy avec SSL | `http://localhost:81` |
| **Keycloak**            | SSO et gestion des identités | `http://localhost:8082` |
| **Poste.IO**            | Serveur de messagerie | `https://localhost:8081` |

---

## Installation
### Prérequis
✅ **Serveur avec Docker & Docker Compose**
✅ **Portainer (optionnel)**
✅ **Nom de domaine configuré (pour HTTPS et les emails)**

### Déploiement
1. **Cloner ce dépôt**
   ```bash
   git clone https://github.com/votre-repo.git
   cd votre-repo
   cp .env .env.backup
   nano .env

   ```

---

## 🔑 Configuration de l'Authentification Unifiée

### Création du Realm dans Keycloak
1. Connectez-vous à l'interface d'administration de Keycloak.
2. Cliquez sur **Create Realm**.
3. Donnez un nom au Realm, par exemple `nextcloud`.
4. Enregistrez les modifications.

### Intégration de Light LDAP dans Keycloak
1. Dans Keycloak, accédez à **User Federation** et ajoutez un nouveau fournisseur LDAP.
2. Remplissez les champs suivants :
   - **UI Display Name** : `LDAP`
   - **Vendor** : `Other`
3. **Paramètres de connexion et d'authentification** :
   - **Connection URL** : `ldap://lightldap:3890`
   - **Enable StartTLS** : `Off`
   - **Use Truststore SPI** : `Always`
   - **Connection pooling** : `On`
   - **Connection timeout** : (laisser vide ou définir selon les besoins)
   - **Bind Type** : `Simple`
   - **Bind DN** : `uid=admin,ou=people,dc=ldap,dc=example,dc=com`
   - **Bind Credentials** : (mot de passe LDAP admin)
4. **Recherche et mise à jour des utilisateurs** :
   - **Edit Mode** : `READ_ONLY`
   - **Users DN** : `dc=ldap,dc=example,dc=com`
   - **Username LDAP Attribute** : `uid`
   - **RDN LDAP Attribute** : `uid`
   - **UUID LDAP Attribute** : `uid`
   - **User Object Classes** : `person`
   - **User LDAP Filter** : `(memberOf=cn=nextcloud_users,ou=groups,dc=ldap,dc=example,dc=com)`
   - **Search Scope** : `One Level`
   - **Read timeout** : (laisser vide ou définir selon les besoins)
   - **Pagination** : `Off`
   - **Referral** : (laisser vide ou définir selon les besoins)
5. **Paramètres de synchronisation** :
   - **Import Users** : `On`
   - **Sync Registrations** : `On`
   - **Batch Size** : (laisser vide ou définir selon les besoins)
   - **Periodic Full Sync** : `Off`
   - **Periodic Changed Users Sync** : `Off`
6. **Intégration Kerberos** : Désactiver toutes les options.
7. **Cache settings** : `DEFAULT`
8. **Paramètres avancés** :
   - **Enable the LDAPv3 password modify extended operation** : `Off`
   - **Validate password policy** : `Off`
   - **Trust Email** : `Off`
   - **Connection trace** : `Off`
9. Sauvegardez la configuration et synchronisez les utilisateurs.
10. Vérifiez que les utilisateurs LDAP sont bien importés dans Keycloak.

### Intégration de Light LDAP dans Nextcloud
1. Installez l'application **LDAP user and group backend** dans Nextcloud.
2. Configurez Nextcloud pour utiliser Light LDAP :
   - **Serveur LDAP** : `ldap://lightldap`
   - **Port** : `3890`
   - **Utilisateur DN** : `uid=admin,ou=people,dc=ldap,dc=example,dc=com`
   - **Mot de passe** : (mot de passe LDAP admin)
   - **DN de base** : `dc=ldap,dc=example,dc=com`
   - **Attribut d’utilisateur** : `uid`
   - **Groupe à synchroniser** : `nextcloud`
3. Dans l'onglet **Serveur**, entrez les informations de connexion LDAP.
4. Dans l'onglet **Utilisateur**, configurez les filtres et les mappages d'attributs.
5. Dans l'onglet **Groupe**, configurez les filtres et les mappages de groupes.
6. Testez la connexion et assurez-vous que les utilisateurs LDAP sont synchronisés avec Nextcloud.

### Intégration de Nextcloud avec Keycloak
1. Dans Keycloak, créez un client pour Nextcloud :
   - **Client ID** : `nextcloud`
   - **Name** : `nextcloud`
   - **Client Protocol** : `openid-connect`
   - **Access Type** : `confidential`
   - **Root URL** : `https://nextcloud.example.com/`
   - **Valid Redirect URIs** : `https://nextcloud.example.com/*`
   - **Base URL** : `https://nextcloud.example.com/`
   - **Admin URL** : `https://nextcloud.example.com/`
   - **Web Origins** : `https://nextcloud.example.com/`
2. Récupérez le **Client Secret** généré.
3. Installez l'application **Social Login** dans Nextcloud et configurez la connexion OpenID Connect :
   - **Authorize URL** : `https://keycloak.example.com/auth/realms/nextcloud/protocol/openid-connect/auth`
   - **Token URL** : `https://keycloak.example.com/auth/realms/nextcloud/protocol/openid-connect/token`
   - **User Info URL** : `https://keycloak.example.com/auth/realms/nextcloud/protocol/openid-connect/userinfo`
   - **Client ID** : `nextcloud`
   - **Client Secret** : (clé récupérée de Keycloak)
   - **Scope** : `openid`
4. Testez la connexion pour valider l’intégration.

---

## 📬 Configuration de Poste.IO

### Configuration DNS
Pour que votre serveur de messagerie fonctionne correctement, vous devez configurer les enregistrements DNS suivants :

- **A**
  - Nom : `@`
  - Valeur : `192.0.2.1` (adresse IP de votre serveur)
  - _Indique l'adresse IP de votre serveur de messagerie. Cet enregistrement permet aux autres serveurs de trouver votre serveur de messagerie en utilisant votre nom de domaine._

- **CNAME**
  - Nom : `mail`
  - Valeur : `@`
  - _Alias pour le serveur de messagerie. Cet enregistrement permet de simplifier la gestion des noms de domaine en créant un alias pour le serveur de messagerie._

- **CNAME**
  - Nom : `mt57.mail`
  - Valeur : `smtp.mailtrap.example`
  - _Alias pour le serveur SMTP de Mailtrap. Cet enregistrement est utilisé pour rediriger le trafic SMTP vers le serveur de Mailtrap._

- **CNAME**
  - Nom : `mt-link.mail`
  - Valeur : `t.mailtrap.example`
  - _Alias pour le serveur de lien de suivi de Mailtrap. Cet enregistrement est utilisé pour rediriger les liens de suivi des emails vers le serveur de Mailtrap._

- **CNAME**
  - Nom : `rwmt1._domainkey.mail`
  - Valeur : `rwmt1.dkim.smtp.mailtrap.example`
  - _Enregistrement DKIM pour la validation des emails. Cet enregistrement permet de vérifier que les emails envoyés depuis votre domaine n'ont pas été altérés._

- **MX**
  - Nom : `@`
  - Valeur : `mail.votre-domaine.com`
  - _Indique le serveur de messagerie pour recevoir les emails. Cet enregistrement spécifie quel serveur de messagerie doit recevoir les emails envoyés à votre domaine._

- **TXT**
  - Nom : `@`
  - Valeur : `"v=spf1 mx include:_spf.smtp.example ~all"`
  - _Enregistrement SPF pour la validation des emails sortants. Cet enregistrement permet de spécifier quels serveurs sont autorisés à envoyer des emails pour votre domaine, réduisant ainsi le risque de spam._

- **TXT**
  - Nom : `_dmarc`
  - Valeur : `"v=DMARC1; p=none; rua=mailto:dmarc-reports@votre-domaine.com"`
  - _Enregistrement DMARC pour la gestion des rapports de validation des emails. Cet enregistrement permet de définir une politique de gestion des emails non conformes et de recevoir des rapports sur les tentatives de fraude._

- **TXT**
  - Nom : `s20250206443._domainkey`
  - Valeur : `"k=rsa; p=MIIBIjANBgkqh..."`
  - _Enregistrement DKIM pour la validation des emails. Cet enregistrement permet de vérifier que les emails envoyés depuis votre domaine n'ont pas été altérés._

### Configuration de poste.io
1. Accédez à l'interface d'administration de poste.io.
2. Configurez les paramètres de domaine et d'utilisateur selon vos besoins.
3. Assurez-vous que les enregistrements DNS sont correctement configurés et propagés.


---

## 📚 Sources

- [Nextcloud All-in-One](https://github.com/nextcloud/all-in-one)
- [Keycloak](https://www.keycloak.org/)
- [Poste.io](https://poste.io/)
- [Nginx Proxy Manager](https://nginxproxymanager.com/)
- [Light LDAP (LLDAP)](https://github.com/lldap/lldap)
- [Microsoft Exchange Server 2019 - Comprendre et configurer les enregistrements DNS](https://www.it-connect.fr/microsoft-exchange-server-2019-comprendre-et-configurer-les-enregistrements-dns/)

---

## Conclusion
Cette infrastructure permet une gestion centralisée des utilisateurs et un accès sécurisé aux services. Toute contribution ou amélioration est la bienvenue !

---

## 📜 Licence
Ce projet est sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.