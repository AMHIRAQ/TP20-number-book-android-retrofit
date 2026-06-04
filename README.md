

https://github.com/user-attachments/assets/9cc320e1-4d73-4e89-bab7-1550af8c18d8





# 📒 TP 20 - Application Number Book avec Android, Contacts et API distante via Retrofit

> Application Android Java permettant de lire les contacts du téléphone,
> de les synchroniser vers un backend PHP/MySQL, et d'effectuer une recherche distante.

---

## 📋 Table des matières

1. [Présentation du projet](#présentation-du-projet)
2. [Architecture globale](#architecture-globale)
3. [Structure des fichiers](#structure-des-fichiers)
4. [Prérequis](#prérequis)
5. [Installation — Backend PHP](#installation--backend-php)
6. [Installation — Base de données](#installation--base-de-données)
7. [Installation — Application Android](#installation--application-android)
8. [Configuration de l'URL du serveur](#configuration-de-lurl-du-serveur)
9. [Fonctionnement détaillé](#fonctionnement-détaillé)
10. [Tests](#tests)
11. [Dépannage courant](#dépannage-courant)
12. [Questions de compréhension](#questions-de-compréhension)
13. [Extensions possibles](#extensions-possibles)

---

## Présentation du projet

**Number Book** est une application mobile Android qui :

- demande la permission d'accéder aux contacts du téléphone ;
- lit et affiche les contacts dans une liste moderne (RecyclerView avec avatars colorés) ;
- envoie les contacts vers un serveur distant via Retrofit (bibliothèque HTTP) ;
- stocke les données dans une base MySQL via un backend PHP ;
- permet de rechercher des contacts dans la base distante par nom ou numéro.

Ce projet illustre les concepts fondamentaux du développement mobile connecté :
ContentResolver, gestion des permissions Android, RecyclerView, Retrofit, Gson, et API REST PHP/MySQL.

---

## Architecture globale

```
┌─────────────────────────────────────────────────────────────┐
│                     APPLICATION ANDROID                      │
│                                                             │
│  ContentResolver → contacts téléphone                       │
│  RecyclerView    → affichage liste                          │
│  Retrofit        → appels HTTP vers le backend              │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP (JSON)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                     BACKEND PHP (Apache)                     │
│                                                             │
│  insertContact.php   ← POST /api/insertContact.php          │
│  getAllContacts.php  ← GET  /api/getAllContacts.php          │
│  searchContact.php  ← GET  /api/searchContact.php?keyword=  │
└────────────────────────┬────────────────────────────────────┘
                         │ PDO
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                     BASE DE DONNÉES MySQL                    │
│                                                             │
│  Base : numberbook                                          │
│  Table : contact (id, name, phone, source, created_at)      │
└─────────────────────────────────────────────────────────────┘
```

---

## Structure des fichiers

```
NumberBook/
│
├── app/src/main/
│   ├── AndroidManifest.xml               # Permissions READ_CONTACTS + INTERNET
│   │
│   ├── java/com/example/numberbook/
│   │   ├── Contact.java                  # Modèle Java (nom, téléphone, source…)
│   │   ├── ApiResponse.java              # Réponse JSON du serveur (success, message)
│   │   ├── ContactApi.java               # Interface Retrofit (déclaration des endpoints)
│   │   ├── RetrofitClient.java           # Singleton Retrofit avec BASE_URL
│   │   ├── ContactAdapter.java           # Adapter RecyclerView + avatars colorés
│   │   └── MainActivity.java             # Activité principale (logique complète)
│   │
│   └── res/
│       ├── layout/
│       │   ├── activity_main.xml         # Interface principale
│       │   └── item_contact.xml          # Ligne de contact (CardView + avatar)
│       ├── drawable/
│       │   └── avatar_bg.xml             # Fond circulaire pour l'avatar
│       └── values/
│           ├── strings.xml
│           └── themes.xml
│
├── app/build.gradle                      # Dépendances Retrofit, RecyclerView, Material
│
└── numberbook-api/                       # Backend PHP
    ├── config/
    │   └── Database.php                  # Connexion PDO à MySQL
    ├── model/
    │   └── Contact.php                   # Classe Contact PHP
    ├── service/
    │   └── ContactService.php            # Logique CRUD (insert, getAll, search)
    ├── api/
    │   ├── insertContact.php             # Endpoint POST — insertion
    │   ├── getAllContacts.php            # Endpoint GET  — lecture complète
    │   └── searchContact.php            # Endpoint GET  — recherche par mot-clé
    └── database/
        └── schema.sql                   # Script SQL de création de la base
```

---

## Prérequis

### Côté serveur

| Outil | Version recommandée |
|-------|---------------------|
| Apache ou Nginx | Dernière version stable |
| PHP | 7.4 ou supérieur |
| MySQL | 5.7 ou supérieur |
| XAMPP / WAMP / MAMP | (optionnel, tout-en-un) |

### Côté Android

| Outil | Version recommandée |
|-------|---------------------|
| Android Studio | Hedgehog ou supérieur |
| JDK | 11 ou supérieur |
| Android SDK | API 24 (Android 7) minimum |
| Émulateur ou appareil physique | Android 7+ |

---

## Installation — Backend PHP

### Étape 1 — Copier les fichiers

Copier le dossier `numberbook-api/` dans le répertoire racine du serveur web :

```
XAMPP  → C:\xampp\htdocs\numberbook-api\
WAMP   → C:\wamp\www\numberbook-api\
MAMP   → /Applications/MAMP/htdocs/numberbook-api/
Linux  → /var/www/html/numberbook-api/
```

### Étape 2 — Vérifier la configuration

Ouvrir `config/Database.php` et vérifier les paramètres :

```php
private $host     = "localhost";
private $dbName   = "numberbook";
private $username = "root";
private $password = "";  // ← modifier si votre MySQL a un mot de passe
```

### Étape 3 — Tester les endpoints

Ouvrir un navigateur et tester :

```
http://localhost/numberbook-api/api/getAllContacts.php
→ Doit retourner : []  (tableau vide avant insertion)

http://localhost/numberbook-api/api/searchContact.php?keyword=test
→ Doit retourner : []
```

---

## Installation — Base de données

### Option A — Via le script SQL fourni

```bash
mysql -u root -p < numberbook-api/database/schema.sql
```

### Option B — Via phpMyAdmin

1. Ouvrir `http://localhost/phpmyadmin`
2. Créer une nouvelle base nommée `numberbook` (collation : `utf8mb4_unicode_ci`)
3. Sélectionner la base → onglet **SQL**
4. Copier-coller le contenu de `database/schema.sql` et exécuter

### Option C — Manuellement

```sql
CREATE DATABASE IF NOT EXISTS numberbook
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE numberbook;

CREATE TABLE contact (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(150) NOT NULL,
    phone      VARCHAR(50)  NOT NULL,
    source     VARCHAR(50)  DEFAULT 'mobile',
    created_at DATETIME     DEFAULT CURRENT_TIMESTAMP
);
```

---

## Installation — Application Android

### Étape 1 — Ouvrir le projet

1. Lancer Android Studio
2. **File → Open** → sélectionner le dossier `NumberBook/`
3. Attendre la synchronisation Gradle

### Étape 2 — Ajouter les dépendances Gradle

Vérifier que `app/build.gradle` contient bien :

```groovy
implementation 'com.squareup.retrofit2:retrofit:2.11.0'
implementation 'com.squareup.retrofit2:converter-gson:2.11.0'
implementation 'androidx.recyclerview:recyclerview:1.3.2'
implementation 'com.google.android.material:material:1.11.0'
```

Puis faire : **File → Sync Project with Gradle Files**

### Étape 3 — Lancer l'application

- **Émulateur** : créer un AVD via *Device Manager*, puis cliquer sur ▶ Run
- **Appareil physique** : activer le mode développeur et le débogage USB, puis connecter via USB

---

## Configuration de l'URL du serveur

Ouvrir `RetrofitClient.java` et modifier `BASE_URL` selon votre configuration :

```java
// ✅ Émulateur Android Studio (10.0.2.2 = localhost de la machine hôte)
private static final String BASE_URL = "http://10.0.2.2/numberbook-api/api/";

// ✅ Appareil physique sur le même réseau Wi-Fi
private static final String BASE_URL = "http://192.168.1.10/numberbook-api/api/";
//                                              ^^^^^^^^^^^^^^
//                                              Remplacer par l'IP réelle de votre machine
```

> 💡 Pour trouver votre adresse IP locale :
> - **Windows** : `ipconfig` dans un terminal
> - **Linux/Mac** : `ifconfig` ou `ip addr`

> ⚠️ Assurez-vous que `AndroidManifest.xml` contient `android:usesCleartextTraffic="true"`
> si vous utilisez HTTP (non HTTPS).

---

## Fonctionnement détaillé

### Bouton « Charger les contacts »

1. Vérifie si la permission `READ_CONTACTS` est accordée
2. Si non → affiche la boîte de dialogue de permission Android
3. Si oui → interroge `ContentResolver` avec `ContactsContract.CommonDataKinds.Phone.CONTENT_URI`
4. Parcourt le `Cursor` pour extraire nom (`DISPLAY_NAME`) et numéro (`NUMBER`)
5. Affiche les contacts dans la RecyclerView, triés par ordre alphabétique

### Bouton « Synchroniser vers le serveur »

1. Vérifie que la liste n'est pas vide
2. Pour chaque contact, appelle `contactApi.insertContact(contact)` via Retrofit
3. Retrofit convertit l'objet Java en JSON grâce à Gson et envoie une requête POST
4. Le backend PHP insère le contact dans MySQL et retourne `{"success": true}`
5. À la fin, affiche un bilan (succès / échecs)

### Bouton « Rechercher »

1. Lit le mot-clé saisi dans le champ de texte
2. Appelle `contactApi.searchContacts(keyword)` → requête GET avec paramètre `?keyword=`
3. Le serveur retourne la liste des contacts correspondants (nom ou numéro contenant le mot-clé)
4. Gson convertit le JSON en `List<Contact>` et la RecyclerView se met à jour

---

## Tests

### Test 1 — Base de données

```sql
USE numberbook;
SHOW TABLES;
-- Résultat attendu : contact
```

### Test 2 — API via navigateur ou Postman

```
GET  http://localhost/numberbook-api/api/getAllContacts.php
→ []  (vide au départ)

GET  http://localhost/numberbook-api/api/searchContact.php?keyword=ali
→ contacts dont le nom ou numéro contient "ali"

POST http://localhost/numberbook-api/api/insertContact.php
Body (JSON) : { "name": "Alice Dupont", "phone": "+33612345678" }
→ { "success": true, "message": "Contact inséré avec succès." }
```

### Test 3 — Application Android

| Étape | Action | Résultat attendu |
|-------|--------|-----------------|
| 1 | Lancer l'application | Interface principale affichée |
| 2 | Cliquer sur "Charger les contacts" | Boîte de permission apparaît |
| 3 | Accepter la permission | Liste des contacts du téléphone affichée |
| 4 | Cliquer sur "Synchroniser" | Toast "X contacts en cours…" puis bilan |
| 5 | Vérifier dans MySQL | Les contacts sont présents dans la table |
| 6 | Saisir "ali" puis "Rechercher" | Seuls les contacts correspondants s'affichent |

---

## Dépannage courant

### ❌ `CLEARTEXT communication not permitted`

**Cause** : Android 9+ bloque les requêtes HTTP non sécurisées par défaut.

**Solution** : Vérifier que `AndroidManifest.xml` contient :
```xml
android:usesCleartextTraffic="true"
```

---

### ❌ `Connection refused` ou `Failed to connect`

**Causes possibles** :
- Le serveur Apache/XAMPP n'est pas démarré
- L'IP dans `BASE_URL` est incorrecte
- L'émulateur doit utiliser `10.0.2.2` et non `localhost` ni `127.0.0.1`
- Un pare-feu bloque la connexion

**Solution** : Vérifier l'IP, démarrer le serveur, tester l'URL dans un navigateur.

---

### ❌ La liste de contacts est vide

**Causes possibles** :
- L'émulateur n'a pas de contacts (en ajouter via l'application "Contacts" de l'émulateur)
- La permission a été refusée

**Solution** : Vérifier les permissions dans *Paramètres → Applications → Number Book → Autorisations*.

---

### ❌ `Erreur de connexion MySQL`

**Cause** : Mauvais identifiants dans `Database.php`.

**Solution** : Vérifier `$username` et `$password` dans `config/Database.php`.

---

### ❌ `404 Not Found` sur les endpoints PHP

**Cause** : Le dossier `numberbook-api` n'est pas au bon endroit dans `htdocs`.

**Solution** : Vérifier le chemin et l'URL. Tester avec `http://localhost/numberbook-api/api/getAllContacts.php`.

---

## Questions de compréhension

1. **Quel est le rôle de `ContentResolver` dans Android ?**
   > C'est l'interface permettant d'interroger les fournisseurs de contenu (contacts, calendrier, médias…) de manière uniforme et sécurisée.

2. **Pourquoi faut-il demander `READ_CONTACTS` à l'exécution ?**
   > Depuis Android 6, les permissions dites "dangereuses" (accès aux données personnelles) doivent être demandées explicitement à l'utilisateur pendant l'utilisation de l'application.

3. **Quelle est la différence entre `RecyclerView` et `ListView` ?**
   > `RecyclerView` est plus performant car il recycle les vues (ViewHolder obligatoire), supporte des mises en page variées (grille, horizontal…) et gère mieux les animations.

4. **Quel est le rôle de Retrofit ?**
   > Retrofit simplifie les appels HTTP en permettant de décrire l'API sous forme d'interface Java annotée, et convertit automatiquement JSON ↔ objets Java via Gson.

5. **Pourquoi utiliser `enqueue()` plutôt qu'un appel synchrone ?**
   > Android interdit les opérations réseau sur le thread principal (UI thread). `enqueue()` exécute l'appel sur un thread séparé et retourne le résultat via un callback.

6. **Pourquoi stocker le numéro de téléphone en `VARCHAR` et non en `INT` ?**
   > Un numéro peut contenir `+`, espaces, tirets ou parenthèses. Un `INT` perdrait ces caractères et causerait des problèmes d'internationalisation (ex : `+33`, `(0)1`…).

7. **Quel est l'avantage des requêtes préparées en PHP ?**
   > Elles séparent la structure SQL des données, ce qui empêche les injections SQL et améliore les performances lors d'appels répétés.

8. **Pourquoi le backend retourne-t-il du JSON ?**
   > JSON est un format léger, lisible, et universel. Gson (côté Android) le convertit automatiquement en objets Java, simplifiant considérablement le code.

---

## Extensions possibles

| Fonctionnalité | Description |
|----------------|-------------|
| 🔄 Sync bidirectionnelle | Télécharger les contacts distants dans le téléphone |
| 🗑️ Suppression distante | Ajouter un endpoint `deleteContact.php` |
| ✏️ Mise à jour | Modifier un contact existant dans la base |
| 🔍 Recherche locale en temps réel | Filtrer la liste affichée sans requête réseau |
| 💾 Cache local avec Room | Persister les données localement pour le mode hors-ligne |
| 🔐 Authentification | Sécuriser l'API avec un token JWT |
| 📊 Détection des doublons | Vérifier si le numéro existe avant insertion |
| ⏳ ProgressBar | Afficher une barre de chargement pendant la synchronisation |
| 🖼️ Photo de contact | Afficher la vraie photo du contact au lieu de l'initiale |
| 📄 Pagination | Charger les résultats par pages (LIMIT/OFFSET en SQL) |

---

## Auteur

Développé dans le cadre du cours **Programmation Mobile : Android avec Java** — TP 20.

---
