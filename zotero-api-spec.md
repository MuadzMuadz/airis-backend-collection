# AIris — Zotero Integration API Specification

This document details the API endpoints for Zotero integration.

---

## 🔐 Authentication
All endpoints (except callback) require a valid **Bearer Token** in the `Authorization` header.

```http
Authorization: Bearer <your_airis_token>
```

---

## 🔗 Connection Management

### 1. Connect Zotero
`POST /api/v1/integrations/zotero/connect`

Initiates the OAuth 1.0a flow.

**Response:**
```json
{
  "authorization_url": "https://www.zotero.org/oauth/authorize?oauth_token=...",
  "oauth_token": "...",
  "oauth_token_secret": "..."
}
```
*Note: The frontend should redirect the user to `authorization_url`.*

---

### 2. OAuth Callback
`GET /api/v1/integrations/zotero/callback`

Endpoint where Zotero redirects the user after authorization.

**Query Parameters:**
- `oauth_token`: Token from Zotero.
- `oauth_verifier`: Verifier from Zotero.
- `oauth_token_secret`: The secret obtained from the `/connect` call.

**Response:**
```json
{
  "status": "success",
  "message": "Zotero connected successfully"
}
```

---

### 3. Check Status
`GET /api/v1/integrations/zotero/status`

Check if the current user is connected to Zotero.

**Response:**
```json
{
  "status": "CONNECTED", // or "NOT_CONNECTED", "EXPIRED", "DISCONNECTED"
  "external_user_id": "1234567",
  "username": "john_doe"
}
```

---

### 4. Disconnect
`DELETE /api/v1/integrations/zotero/disconnect`

Removes the connection and clears access tokens.

**Response:**
```json
{
  "status": "success",
  "message": "Zotero disconnected"
}
```

---

## 🔄 Synchronization

### 5. Sync Pull (Zotero → AIris)
`POST /api/v1/integrations/zotero/sync/pull`

Fetch items from Zotero library and save them to an AIris project folder.

**Body (JSON):**
```json
{
  "project_folder_id": "uuid-here",
  "zotero_collection_key": "optional-key-here" 
}
```
*If `zotero_collection_key` is omitted, it pulls the entire library.*

**Response:**
```json
{
  "total_fetched": 150,
  "synced": 145,
  "failed": 5
}
```

---

### 6. Sync Push (AIris → Zotero)
`POST /api/v1/integrations/zotero/sync/push`

Push specific reference items from AIris to the user's Zotero library.

**Body (JSON):**
```json
{
  "project_reference_ids": [
    "uuid-1",
    "uuid-2"
  ]
}
```

**Response:**
```json
{
  "pushed": 2, // New items created in Zotero
  "updated": 0, // Existing items updated in Zotero
  "failed": 0
}
```

---

## 🛠️ Data Mapping Rules

### Zotero → AIris (Pull)
- `title` → `title`
- `creators` → `author` (Joins first creator: "LastName, FirstName")
- `DOI` → `doi`
- `date` → `publication_date` (Parsed from YYYY-MM-DD or YYYY)
- `abstractNote` → `abstract`
- `url` → `url`

### AIris → Zotero (Push)
- `title` → `title`
- `author` → `creators` (Splits "LastName, FirstName" back to Zotero schema)
- `doi` → `DOI`
- `publication_date` → `date`
- `abstract` → `abstractNote`
- `url` → `url`
