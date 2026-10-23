# Documentation Technique — API Réception de Leads Partenaire

**Version**: 1.0
**Endpoint base**: `POST /api/resource/receive/contact`

---

## Table des matières

1. [Authentification](#1-authentification)
2. [Endpoint](#2-endpoint)
3. [Body de la requête](#3-body-de-la-requête)
4. [Réponses](#4-réponses)
5. [Codes HTTP](#5-codes-http)
6. [Notes techniques](#6-notes-techniques)
7. [Exemple cURL](#7-exemple-curl)

---

## 1. Authentification

L'API utilise un **Bearer Token** transmis dans l'en-tête HTTP.

```http
Authorization: Bearer {votre_token}
```

> Le token est fourni par l'équipe ici-formation lors de l'onboarding du partenaire.
> Il est lié à votre compte partenaire (`societe`) dans notre système.

**En cas de token manquant ou invalide** :

```json
HTTP 401 Unauthorized
{
  "success": false,
  "message": "Token Bearer manquant ou invalide"
}
```

---

## 2. Endpoint

| Méthode | URL | Content-Type |
|---------|-----|--------------|
| `POST` | `/api/resource/receive/contact` | `application/json` |

---

## 3. Body de la requête

### Exemple complet

```json
{
  "civilite": "M.",
  "prenom": "Jean",
  "nom": "Dupont",
  "tel": "0623856788",
  "email": "jean.dupont@example.com",
  "ville": "Paris",
  "code_postal": "75001",
  "distanciel": true,
  "type": "cbc",
  "statut": "Salarie(e) du privé",
  "theme": null,
  "domaine": null
}
```

### Description des champs

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| `civilite` | string | Oui | Civilité du contact |
| `prenom` | string | Oui | Prénom |
| `nom` | string | Oui | Nom de famille |
| `tel` | string | Oui | Numéro de téléphone |
| `email` | string | Oui | Adresse email valide |
| `ville` | string | Oui | Ville |
| `code_postal` | string | Oui | Code postal français (5 chiffres) |
| `distanciel` | boolean | Oui | `true` = à distance / `false` = présentiel |
| `type` | string | Oui | Type de formation recherché (voir valeurs ci-dessous) |
| `statut` | string | Oui | Statut professionnel du contact (voir valeurs ci-dessous) |
| `theme` | string\|null | Non | Thème de formation (utilisé pour `type: "of"`) |
| `domaine` | string\|null | Non | Domaine de recherche |

### Valeurs acceptées : `civilite`

| Valeur |
|--------|
| `"M."` |
| `"Mme"` |

### Valeurs acceptées : `type`

| Valeur | Description |
|--------|-------------|
| `"cbc"` | Coaching professionnel |
| `"of"` | Formation (organisme de formation) |
| `"vae"` | Validation des Acquis de l'Expérience |
| `"opl"` | Outplacement |
| `"ace"` | Bilan de compétences |

### Valeurs acceptées : `statut`

| Valeur |
|--------|
| `"Salarie(e) du privé"` |
| `"Salarie(e) du public"` |
| `"Entreprise"` |
| `"Demandeur d'emploi"` |
| `"Profession libérale"` |
| `"Etudiant"` |
| `"RH"` |
| `"Autre"` |

---

## 4. Réponses

### Succès — HTTP 201

```json
{
  "success": true,
  "message": "contact réceptionné"
}
```

### Erreur de validation — HTTP 400

```json
{
  "success": false,
  "message": "Le champ type est obligatoire"
}
```

### JSON invalide — HTTP 400

```json
{
  "success": false,
  "message": "Invalid JSON"
}
```

### Lead doublon — HTTP 409

Renvoyé si le couple `email` + `tel` a déjà été transmis par votre partenaire.

```json
{
  "success": false,
  "message": "Lead doublon détecté : email \"jean.dupont@example.com\" et téléphone \"0623856788\" déjà transmis par {votre_societe}",
  "error_code": "DUPLICATE_LEAD"
}
```

---

## 5. Codes HTTP

| Code | Scénario |
|------|----------|
| `201 Created` | Lead reçu et traité avec succès |
| `400 Bad Request` | JSON invalide, champ manquant ou valeur incorrecte |
| `401 Unauthorized` | Token absent ou invalide |
| `409 Conflict` | Lead doublon (même email + téléphone déjà transmis) |

---

## 6. Notes techniques

### Code postal
Le code postal est normalisé automatiquement (espaces supprimés, extraction du pattern 5 chiffres). Envoyez-le au format `"75001"`.

### Déduplication
Notre système détecte les doublons en comparant `email` + `tel` pour les leads déjà transmis par votre partenaire. En cas de doublon, un HTTP 409 est retourné — aucun lead n'est créé.

### Matching automatique
Après réception, le lead est mis en relation automatiquement avec des formations disponibles correspondant au `type`, `code_postal` et `distanciel`. Si aucune formation n'est disponible, le lead est pris en charge par nos conseillers.

---

## 7. Exemple cURL

```bash
curl -X POST https://votre-domaine.fr/api/resource/receive/contact \
  -H "Authorization: Bearer VOTRE_TOKEN_ICI" \
  -H "Content-Type: application/json" \
  -d '{
    "civilite": "Mme",
    "prenom": "Marie",
    "nom": "Martin",
    "tel": "0612345678",
    "email": "marie.martin@example.com",
    "ville": "Lyon",
    "code_postal": "69001",
    "distanciel": false,
    "type": "ace",
    "statut": "Salarie(e) du privé"
  }'
```

---

**Contact onboarding** : Contacter l'équipe technique ici-formation pour l'obtention du token Bearer et de l'URL de production.
