# Issue Report CC-FA — Web App

Aplikasi Issue Report CC-FA yang sebelumnya berjalan di Google Apps Script,
sekarang dikonversi ke **static web app** menggunakan **Supabase** sebagai backend
dan **GitHub Pages** sebagai hosting.

---

## 🗂 Struktur Project

```
ccfa-webapp/
├── index.html                        ← Aplikasi utama (satu file)
├── supabase/
│   └── migrations/
│       └── 001_schema.sql            ← SQL schema untuk Supabase
└── README.md
```

---

## ⚙️ Setup — Langkah demi Langkah

### 1. Buat Supabase Project

1. Buka [supabase.com](https://supabase.com) → **New Project**
2. Catat **Project URL** dan **anon public key** (Settings → API)

### 2. Jalankan SQL Schema

1. Di Supabase Dashboard → **SQL Editor**
2. Copy isi `supabase/migrations/001_schema.sql`
3. Klik **Run**

### 3. Buat Storage Bucket

1. Supabase Dashboard → **Storage** → **New Bucket**
2. Nama: `ccfa-attachments`
3. ✅ Public bucket = **ON**

### 4. Buat User Akun

Di Supabase Dashboard → **Authentication** → **Users** → **Add User**:

| Email | Password | Keterangan |
|---|---|---|
| `admin@ccfa.local` | `Admin1` | Administrator |
| `aldi@ccfa.local` | `1` | User biasa |

Setelah user dibuat, catat UUID masing-masing dari tabel `auth.users`,
lalu jalankan SQL berikut di **SQL Editor** (ganti UUID sesuai):

```sql
INSERT INTO profiles (id, user_id, display_name, role) VALUES
  ('<UUID_ADMIN>', 'Admin', 'Administrator', 'admin'),
  ('<UUID_ALDI>',  'Aldi',  'Aldi',          'user');
```

> **Cara dapat UUID:** Authentication → Users → klik user → copy ID

### 5. Isi Config di index.html

Buka `index.html`, cari bagian ini di dekat atas file:

```html
<script>
  window.__SB_URL  = 'https://YOUR_PROJECT_REF.supabase.co';
  window.__SB_ANON = 'YOUR_ANON_KEY';
</script>
```

Ganti:
- `YOUR_PROJECT_REF` → Project URL dari Supabase (Settings → API)
- `YOUR_ANON_KEY` → anon/public key dari Supabase (Settings → API)

### 6. Deploy ke GitHub Pages

#### a. Buat GitHub Repository

```bash
git init
git add .
git commit -m "Initial commit: CC-FA web app"
git remote add origin https://github.com/USERNAME/ccfa-webapp.git
git push -u origin main
```

#### b. Aktifkan GitHub Pages

1. Repository → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main`, Folder: `/ (root)`
4. Klik **Save**
5. Tunggu beberapa menit → URL akan muncul:
   `https://USERNAME.github.io/ccfa-webapp/`

---

## 🔐 Keamanan

- Semua data disimpan di Supabase (PostgreSQL)
- Auth menggunakan Supabase Auth (email + password)
- Row Level Security (RLS) aktif di semua tabel
- File attachment disimpan di Supabase Storage
- Anon key aman untuk ditaruh di frontend (hanya bisa akses sesuai RLS policy)

---

## 🔄 Perbedaan dari Versi GAS

| Fitur | Google Apps Script | Web App (Supabase) |
|---|---|---|
| Auth | Session di PropertiesService | Supabase Auth (JWT) |
| Data | Google Sheets | PostgreSQL |
| File | Google Drive | Supabase Storage |
| Hosting | Apps Script URL | GitHub Pages |
| Holidays | PropertiesService | Tabel `holidays` |
| Offline | ❌ | ❌ (butuh internet) |
| Realtime | ❌ | ✅ (bisa diaktifkan) |

---

## 🛠 Troubleshooting

**Login gagal "ID atau Password salah"**
→ Pastikan email format `userid@ccfa.local` sudah dibuat di Supabase Auth
→ Pastikan profile sudah diinsert di tabel `profiles`

**Data tidak muncul**
→ Cek RLS policy sudah ter-apply (jalankan ulang SQL schema)
→ Cek anon key sudah benar di `index.html`

**File upload gagal**
→ Pastikan bucket `ccfa-attachments` sudah dibuat dan Public = ON
→ Cek di Supabase Storage → Policies → ada policy untuk authenticated users

**CORS error**
→ Supabase sudah handle CORS otomatis
→ Jika ada error, tambahkan domain GitHub Pages di Supabase → Settings → API → Allowed origins
