**English** | [Français](README.fr.md)

# Belgian eID Card API - ASP.NET Project

This project provides an API to interact with the Belgian eID card using a C# wrapper (`EidSamples`). It can read personal data from the eID card, authenticate users through cryptographic signatures, and sign data.

## Table of contents
1. [Features](#features)
2. [Dependencies](#dependencies)
3. [Installation](#installation)
4. [API endpoints](#api-endpoints)
5. [Usage examples](#usage-examples)
6. [Security](#security)

---

## Features
- Reading personal data:
  - Photo
  - Full name
  - National number
  - Date of birth
  - Gender
  - Address
- Authentication through a cryptographic signature
- Signing arbitrary data
- Retrieving the authentication certificate
- Retrieving the public key

---

## Dependencies
1. **EidSamples**: C# wrapper to interact with the Belgian eID card
2. **ASP.NET Core**: framework used to build the API
3. **Belgian eID middleware**: must be installed on the system

---

## Installation
1. **Clone the repository**:
```bash
   git clone https://github.com/JeremyMarbaise/eid-mw.git
```
2. **Open the sln file with Visual Studio**

  The sln file is located at [doc/sdk/examples/CS/EidSamples.sln](examples/CS/EidSamples.sln)

3. **Run the project**
   In Visual Studio, select TestApi as the startup project and run it
---


The API will be available at `http://localhost:7043` (default port).

---

## API endpoints

### 1. Registration data
- **Endpoint**: `GET /auth/register`
- **Description**: Returns all the personal data needed for registration
- **Response**:
```json
{
  "name": "Doe John",
  "nationalNumber": "12345678901",
  "DateOfBirth": "1980-01-01",
  "gender": "M",
  "address": "Rue de la Loi 16, 1000 Bruxelles"
}
```

### 2. Photo
- **Endpoint**: `GET /photo`
- **Description**: Returns the photo as a JPEG
- **Response**: JPEG binary (Content-Type: image/jpeg)

### 3. Authentication
- **Endpoint**: `POST /auth/authentication`
- **Description**: Signs a concatenation of challenge, timestamp and salt
- **Request**:
```json
{
  "challenge": "base64_encoded_challenge",
  "salt": "base64_encoded_salt"
}
```
- **Response**: Binary signature (returned as a byte array)

### 4. Data signing
- **Endpoint**: `POST /auth/sign`
- **Description**: Signs arbitrary data
- **Request**:
```json
{
  "data": "base64_encoded_data"
}
```
- **Response**: Binary signature

### 5. Certificate
- **Endpoint**: `GET /auth/certificate`
- **Description**: Returns the authentication certificate
- **Response**:
```json
{
  "certificate": "base64_encoded_certificate"
}
```

### 6. Public key
- **Endpoint**: `GET /auth/publickey`
- **Description**: Returns the public key
- **Response**:
```json
{
  "pubkey": "base64_encoded_public_key"
}
```

---

## Usage examples

### 1. Registering a new user
```javascript
// Fetch the data
const response = await fetch('http://localhost:5000/auth/register');
const userData = await response.json();

console.log(userData);
```

### 2. Authentication flow
```javascript
// Generate the values
const challenge = crypto.getRandomValues(new Uint8Array(32));
const salt = crypto.getRandomValues(new Uint8Array(16));

// Send them to the API
const authResponse = await fetch('http://localhost:5000/auth/authentication', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    challenge: btoa(String.fromCharCode(...challenge)),
    salt: btoa(String.fromCharCode(...salt))
  })
});

const signature = await authResponse.json();
// Verify the signature...
```

### 3. Signing a document
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

## Security
1. **Time window**: authentication uses 5-minute time windows to prevent replay attacks
2. **Secure concatenation**: the data to sign is a concatenation of:
   - The provided challenge
   - The timestamp of the current window
   - A random salt
3. **CORS**: the API is configured with a permissive CORS policy (`AllowAllOrigins`) - restrict it in production

---

Changes since the previous version:
- Removed obsolete endpoints (`/name`, `/dob`, `/labels`)
- Added new endpoints (`/auth/register`, `/auth/publickey`)
- Updated the authentication mechanism
- Added address and gender retrieval
