# Zotero Integration API

Base URL: `{{base_url}}/api/v1/integrations/zotero`

All endpoints require authentication via Bearer token.

```
Authorization: Bearer {{access_token}}
```

---

## 1. Connect — Initiate OAuth

Memulai OAuth flow. Backend generate request token dari Zotero dan return authorization URL.

**Endpoint:** `POST /connect`

**Headers:**
```
Authorization: Bearer {{access_token}}
```

**Request Body:** none

**Response (200):**
```json
{
  "oauth_token": "abc123",
  "oauth_token_secret": "secret456",
  "authorization_url": "https://www.zotero.org/oauth/authorize?oauth_token=abc123"
}
```

**Response (500):**
```json
{
  "detail": "error message"
}
```

**Frontend flow:**
1. Simpan `oauth_token_secret` di state/localStorage
2. Redirect user ke `authorization_url`
3. User approve di Zotero → redirect balik ke callback URL dengan `oauth_token` + `oauth_verifier`

---

## 2. Callback — Complete OAuth

Frontend kirim `oauth_token`, `oauth_token_secret`, dan `oauth_verifier` yang didapat dari OAuth redirect. Backend exchange ke access token final dan simpan ke database.

**Endpoint:** `POST /callback`

**Headers:**
```
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

**Request Body:**
```json
{
  "oauth_token": "abc123",
  "oauth_token_secret": "secret456",
  "oauth_verifier": "verifier789"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| oauth_token | string | yes | Dari callback URL param |
| oauth_token_secret | string | yes | Dari response `/connect` (disimpan di step 1) |
| oauth_verifier | string | yes | Dari callback URL param |

**Response (200):**
```json
{
  "status": "success",
  "message": "Zotero connected successfully"
}
```

**Response (500):**
```json
{
  "detail": "error message"
}
```

---

## 3. Status — Check Connection

Cek apakah user sudah terkoneksi dengan Zotero.

**Endpoint:** `GET /status`

**Headers:**
```
Authorization: Bearer {{access_token}}
```

**Request Body:** none

**Response (200) — Connected:**
```json
{
  "status": "CONNECTED",
  "external_user_id": "12345",
  "username": "johndoe"
}
```

**Response (200) — Not connected:**
```json
{
  "status": "NOT_CONNECTED"
}
```

---

## 4. Disconnect

Putuskan koneksi Zotero. Token dihapus dari database, record tetap ada dengan status DISCONNECTED.

**Endpoint:** `DELETE /disconnect`

**Headers:**
```
Authorization: Bearer {{access_token}}
```

**Request Body:** none

**Response (200):**
```json
{
  "status": "success",
  "message": "Zotero disconnected"
}
```

---

## 5. Sync Pull — Zotero to AIris

Tarik referensi dari library Zotero user ke project folder di AIris. Harus sudah CONNECTED.

**Endpoint:** `POST /sync/pull`

**Headers:**
```
Authorization: Bearer {{access_token}}
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| project_folder_id | UUID | yes | Target project folder di AIris |
| zotero_collection_key | string | no | Filter koleksi tertentu di Zotero. Kalau kosong, pull semua items |

**Example:**
```
POST /sync/pull?project_folder_id=550e8400-e29b-41d4-a716-446655440000&zotero_collection_key=ABC123
```

**Request Body:** none

**Response (200):**
```json
{
  "total_fetched": 50,
  "synced": 48,
  "failed": 2
}
```

**Response (400):**
```json
{
  "detail": "Zotero not connected"
}
```

---

## 6. Sync Push — AIris to Zotero

Push referensi dari AIris ke library Zotero user. Harus sudah CONNECTED.

**Endpoint:** `POST /sync/push`

**Headers:**
```
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

**Request Body:**
```json
[
  "550e8400-e29b-41d4-a716-446655440000",
  "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| project_reference_ids | UUID[] | yes | List ID referensi dari AIris yang mau dipush ke Zotero |

**Response (200):**
```json
{
  "pushed": 3,
  "updated": 1,
  "failed": 0
}
```

**Response (400):**
```json
{
  "detail": "Zotero not connected"
}
```

---

## Flow Lengkap

```
1. POST /connect
   ↓ dapet authorization_url + oauth_token_secret
2. Redirect user ke authorization_url
   ↓ user approve → redirect balik dengan oauth_token + oauth_verifier
3. POST /callback { oauth_token, oauth_token_secret, oauth_verifier }
   ↓ backend exchange + simpan → CONNECTED
4. GET /status → cek status
5. POST /sync/pull → tarik dari Zotero
6. POST /sync/push → push ke Zotero
7. DELETE /disconnect → putus koneksi
```
