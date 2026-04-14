Full Endpoint List (Grouped by Functionality)
Berikut adalah daftar lengkap endpoint AIris Backend yang telah dikelompokkan secara logis:

1. Authentication & Identity
Mengelola login, token akses, dan informasi profil user yang sedang aktif.

POST /api/v1/auth/login : Login user (menggunakan email/password) untuk mendapatkan JWT.
GET /api/v1/auth/me : Mendapatkan informasi profil user yang sedang login.
POST /api/v1/auth/logout : Logout user (client-side).
2. User Management
Administrasi data user dalam sistem.

POST /api/v1/users/ : Registrasi user baru.
GET /api/v1/users/ : List semua user aktif (admin/pagination).
GET /api/v1/users/{user_id} : Detail user berdasarkan UUID.
PATCH /api/v1/users/{user_id} : Update profil/password user.
DELETE /api/v1/users/{user_id} : Soft delete user.
3. Project Management
Wadah utama untuk riset, menyimpan referensi, dan folder.

GET /api/v1/projects/ : List semua project milik user.
POST /api/v1/projects/ : Buat project baru.
GET /api/v1/projects/{project_id} : Detail project.
PATCH /api/v1/projects/{project_id} : Update nama/deskripsi project.
DELETE /api/v1/projects/{project_id} : Hapus project beserta isinya.
GET /api/v1/projects/{project_id}/chats : List semua sesi chat yang terkait dengan project ini.
3a. Project Folders (Organisasi Referensi)
GET /api/v1/projects/{project_id}/folders : List folder dalam project.
POST /api/v1/projects/{project_id}/folders : Buat folder baru.
PATCH /api/v1/projects/{project_id}/folders/{folder_id} : Update nama folder.
DELETE /api/v1/projects/{project_id}/folders/{folder_id} : Hapus folder.
3b. Project References (Paper/Bahan Riset)
GET /api/v1/projects/{project_id}/references : List semua referensi dalam project.
POST /api/v1/projects/{project_id}/references : Tambah referensi manual.
POST /api/v1/projects/{project_id}/references/bulk : Tambah banyak referensi sekaligus.
PATCH /api/v1/projects/{project_id}/references/bulk : Pindah banyak referensi ke folder.
DELETE /api/v1/projects/{project_id}/references : Hapus banyak referensi sekaligus.
POST /api/v1/projects/{project_id}/references/ingest : Ingest referensi ke Knowledge Base (RAG).
4. Chat Systems
Terdiri dari dua jenis: General Chat (umum) dan Paper Discussion (khusus referensi).

4a. General Chat (Umum)
GET /api/v1/general-chat/sessions/ : List semua sesi chat umum.
POST /api/v1/general-chat/sessions/ : Buat sesi chat baru.
GET /api/v1/general-chat/sessions/{session_id} : Detail sesi.
DELETE /api/v1/general-chat/sessions/{session_id} : Hapus sesi.
POST /api/v1/general-chat/messages/sse/create_message : Chat Utama (Streaming SSE, mendukung file upload & RAG).
4b. Paper Discussion (Diskusi Referensi Project)
GET /api/v1/projects/{project_id}/library/sessions : List sesi diskusi paper.
POST /api/v1/projects/{project_id}/library/sessions : Buat sesi diskusi baru.
GET /api/v1/projects/{project_id}/library/sessions/{session_id} : Detail sesi diskusi.
POST /api/v1/projects/{project_id}/library/sessions/{session_id}/chats : Diskusi Paper (SSE, fokus pada DOI tertentu).
5. Research Tools & Search
Alat bantu riset bertenaga AI.

GET /api/v1/search/ : Global Unified Search (cari di project & chat).
POST /api/v1/suggestion : Query Suggestion (Saran pertanyaan riset).
POST /api/v1/projects/{project_id}/research : Melakukan riset web/akademik otomatis.
6. Storage & Utilities (MinIO)
Manajemen file fisik di cloud storage.

POST /api/v1/minio/upload : Upload file dengan progres SSE.
GET /api/v1/minio/list/{session_id} : List file dalam sesi/project.
DELETE /api/v1/minio/delete/{filename} : Hapus file.
GET /api/v1/status/ : Cek kesehatan API (Health Check).