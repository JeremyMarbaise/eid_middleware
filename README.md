Voici la documentation mise à jour pour votre API locale d'interaction avec la carte d'identité électronique belge :

---

# API pour la carte eID belge - Projet ASP.NET

Ce projet fournit une API pour interagir avec la carte eID belge en utilisant un wrapper C# (`EidSamples`). Il permet de récupérer des données personnelles depuis la carte eID, d'authentifier des utilisateurs via des signatures cryptographiques, et de signer des données.

## Table des matières
1. [Fonctionnalités](#fonctionnalités)
2. [Dépendances](#dépendances)
3. [Installation](#installation)
4. [Endpoints de l'API](#endpoints-de-lapi)
5. [Exemples d'utilisation](#exemples-dutilisation)
6. [Sécurité](#sécurité)

---

## Fonctionnalités
- Récupération des données personnelles :
  - Photo
  - Nom complet
  - Numéro national
  - Date de naissance
  - Genre
  - Adresse
- Authentification via signature cryptographique
- Signature de données arbitraires
- Récupération du certificat d'authentification
- Récupération de la clé publique

---

## Dépendances
1. **EidSamples** : Wrapper C# pour interagir avec la carte eID belge
2. **ASP.NET Core** : Framework pour construire l'API
3. **Middleware eID belge** : Doit être installé sur le système

---

## Installation
1. **Cloner le dépôt** :
```bash
   git clone https://github.com/JeremyMarbaise/eid-mw.git
```
2. **Utiliser Visual Studio et ouvrir le fichier sln**

  Le fichier sln se trouve dans [doc/sdk/examples/CS/EidSamples.sln](examples/CS/EidSamples.sln)

3. **Lancer le projet**
   Avec Visual Studio choisir le projet de source TestApi et lancer
---


L'API sera disponible sur `http://localhost:7043` (port par défaut).

---

## Endpoints de l'API

### 1. Récupération des données d'enregistrement
- **Endpoint**: `GET /auth/register`
- **Description**: Récupère toutes les données personnelles nécessaires à l'enregistrement
- **Réponse**:
```json
{
  "name": "Doe John",
  "nationalNumber": "12345678901",
  "DateOfBirth": "1980-01-01",
  "gender": "M",
  "address": "Rue de la Loi 16, 1000 Bruxelles"
}
```

### 2. Récupération de la photo
- **Endpoint**: `GET /photo`
- **Description**: Récupère la photo au format JPEG
- **Réponse**: Binaire JPEG (Content-Type: image/jpeg)

### 3. Authentification
- **Endpoint**: `POST /auth/authentication`
- **Description**: Signe une concaténation de challenge, timestamp et salt
- **Requête**:
```json
{
  "challenge": "base64_encoded_challenge",
  "salt": "base64_encoded_salt"
}
```
- **Réponse**: Signature au format binaire (retourné comme tableau d'octets)

### 4. Signature de données
- **Endpoint**: `POST /auth/sign`
- **Description**: Signe des données arbitraires
- **Requête**:
```json
{
  "data": "base64_encoded_data"
}
```
- **Réponse**: Signature au format binaire

### 5. Récupération du certificat
- **Endpoint**: `GET /auth/certificate`
- **Description**: Récupère le certificat d'authentification
- **Réponse**:
```json
{
  "certificate": "base64_encoded_certificate"
}
```

### 6. Récupération de la clé publique
- **Endpoint**: `GET /auth/publickey`
- **Description**: Récupère la clé publique
- **Réponse**:
```json
{
  "pubkey": "base64_encoded_public_key"
}
```

---

## Exemples d'utilisation

### 1. Enregistrement d'un nouvel utilisateur
```javascript
// Récupération des données
const response = await fetch('http://localhost:5000/auth/register');
const userData = await response.json();

console.log(userData);
```

### 2. Processus d'authentification
```javascript
// Génération des éléments
const challenge = crypto.getRandomValues(new Uint8Array(32));
const salt = crypto.getRandomValues(new Uint8Array(16));

// Envoi à l'API
const authResponse = await fetch('http://localhost:5000/auth/authentication', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    challenge: btoa(String.fromCharCode(...challenge)),
    salt: btoa(String.fromCharCode(...salt))
  })
});

const signature = await authResponse.json();
// Vérification de la signature...
```

### 3. Signature de document
```javascript
const documentHash = await crypto.subtle.digest('SHA-256', documentContent);

const signResponse = await fetch('http://localhost:5000/auth/sign', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    data: btoa(String.fromCharCode(...new Uint8Array(documentHash)))
  })
});

const documentSignature = await signResponse.json();
```

---

## Sécurité
1. **Fenêtre temporelle** : L'authentification utilise des fenêtres temporelles de 5 minutes pour prévenir les attaques par rejeu
2. **Concaténation sécurisée** : Les données à signer sont une concaténation de :
   - Le challenge fourni
   - Le timestamp de la fenêtre courante
   - Un salt aléatoire
3. **CORS** : L'API est configurée avec une politique CORS permissive (`AllowAllOrigins`) - à restreindre en production

---

Cette documentation reflète l'état actuel de votre API telle que décrite dans le code source fourni. Les principales différences avec l'ancienne documentation incluent :
- La suppression des endpoints obsolètes (`/name`, `/dob`, `/labels`)
- L'ajout des nouveaux endpoints (`/auth/register`, `/auth/publickey`)
- La mise à jour du mécanisme d'authentification
- L'ajout de la récupération de l'adresse et du genre
