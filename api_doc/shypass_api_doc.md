# Overview

## ShyPass API Documentation

Welcome to the API documentation for **ShyPass**, a secure password manager. This API allows developers to integrate ShyPass functionalities into their own applications, Sync ShyPass vault data with third-party services, and build a custom frontend for ShyPass. 

### Base URL 

`https://api.shypass.com/v1/`

###  Prerequisites
To start using the ShyPass API, you will need:
- A registered ShyPass account
- An OAuth2.0 client ID and client secret
- Access token via the OAuth2.0 authorization flow
- Basic knowledge of HTTP, RESTful APIs, and JWT

###  Target Audience
This documentation is intended for Developers integrating ShyPass into their applications.


---

## Authentication
This API uses OAuth 2.0 Authorization framework to allow third-part application to access its resources. All API requests (except `/auth/token`) require a valid **OAuth2.0 Bearer Token** in the request headers.

###  OAuth 2.0 Token Endpoint

#### `POST /auth/token`
Use this endpoint to obtain an OAuth2.0 access token.

**Request Body (application/x-www-form-urlencoded):**
```
grant_type=client_credentials&
client_id=YOUR_CLIENT_ID&
client_secret=YOUR_CLIENT_SECRET
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsIn...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Use this `access_token` as the Bearer token in all subsequent requests:
```
Authorization: Bearer <access_token>
```

---

## Vaults API

### `GET /vaults`
Retrieve all vault entries for the authenticated user.

####  Query Parameters

| Parameter   | Type   | Required | Description                         |
|-------------|--------|----------|-------------------------------------|
| `limit`     | int    | No       | Maximum number of items to return  |
| `offset`    | int    | No       | Index to start listing from        |
| `search`    | string | No       | Filter vaults by title or username |

**Request Example:**
```
GET /vaults?limit=10&offset=20&search=gmail
Authorization: Bearer <access_token>
```

**Response:**
```json
[
  {
    "id": "abc123",
    "title": "Gmail",
    "username": "myemail@gmail.com",
    "encryptedPassword": "kfj34lkj4==",
    "note": "Personal account",
    "createdAt": "2025-07-01T12:00:00Z"
  }
]
```

---

### `POST /vaults`
Create a new vault entry.

**Request Body:**
```json
{
  "title": "GitHub",
  "username": "Andyroddicks",
  "encryptedPassword": "fje83kl8==",
  "note": "Work repository access"
}
```

**Response:**
```json
{
  "message": "Vault item created successfully.",
  "id": "def456"
}
```

---

### `PUT /vaults/{id}`
Update a vault entry.

####  Path Parameters

| Parameter | Type   | Required | Description              |
|-----------|--------|----------|--------------------------|
| `id`      | string | Yes      | The ID of the vault item |

**Request Example:**
```
PUT /vaults/def456
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "title": "GitHub",
  "username": "Zackkerman",
  "encryptedPassword": "newEncryptedPass==",
  "note": "Updated note"
}
```

**Response:**
```json
{
  "message": "Vault item updated successfully."
}
```

---

### `DELETE /vaults/{id}`
Delete a vault entry.

####  Path Parameters

| Parameter | Type   | Required | Description              |
|-----------|--------|----------|--------------------------|
| `id`      | string | Yes      | The ID of the vault item |

**Request Example:**
```
DELETE /vaults/def456
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "message": "Vault item deleted successfully."
}
```

---

##  Account APIs

### `GET /me`
Get the authenticated user profile.

**Response:**
```json
{
  "id": "user123",
  "email": "user@example.com",
  "createdAt": "2025-06-01T10:20:30Z"
}
```

---

### `PUT /me`
Update the user profile.

**Request Body:**
```json
{
  "email": "newuser@example.com",
  "password": "NewPassword123"
}
```

**Response:**
```json
{
  "message": "Profile updated successfully."
}
```

---

##  File Attachment APIs

### `POST /attachments`
Upload an encrypted file.

**Form Data:**
- `file`: Encrypted file blob

**Response:**
```json
{
  "id": "file123",
  "url": "https://s3.aws.com/shypass/file123.enc"
}
```

---

### `GET /attachments/{id}`
Download an encrypted attachment.

#### Path Parameters

| Parameter | Type   | Required | Description                   |
|-----------|--------|----------|-------------------------------|
| `id`      | string | Yes      | The ID of the attachment file |

**Request Example:**
```
GET /attachments/file123
Authorization: Bearer <access_token>
```

*Returns a file stream.*

---

## Status Codes
| Code | Description           |
|------|-----------------------|
| 200  | OK                    |
| 201  | Created               |
| 400  | Bad Request           |
| 401  | Unauthorized          |
| 403  | Forbidden             |
| 404  | Not Found             |
| 500  | Internal Server Error |

---

For support, contact: [support@shypass.io](mailto:support@shypass.io)
