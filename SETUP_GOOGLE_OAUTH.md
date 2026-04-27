# Setup Google OAuth untuk Admin Panel JKT48

## ✅ Apa yang Sudah Diupdate

File `admin.html` sudah dilengkapi dengan:
- ✓ **Google OAuth Button** - tombol login dengan Google
- ✓ **Session Management** - auto-login jika user sudah punya Google session
- ✓ **Secure Logout** - logout dari Supabase auth + clear localStorage
- ✓ **Fallback Password Login** - tetap support password login biasa
- ✓ **Provider Detection** - system tahu user login via Google atau password

---

## 🔧 Setup di Supabase Dashboard (5 Menit)

### Step 1: Enable Google OAuth

1. Buka **Supabase Dashboard** → pilih project Anda
2. Pergi ke **Authentication → Providers**
3. Cari **Google** dan klik **Enable**

### Step 2: Setup Google Cloud Console

1. Buka [Google Cloud Console](https://console.cloud.google.com/)
2. Buat project baru atau gunakan yang existing
3. Pergi ke **APIs & Services → Credentials**
4. Klik **+ Create Credentials → OAuth 2.0 Client ID**
5. Pilih **Web application**
6. Beri nama: `JKT48 Admin OAuth`

### Step 3: Set Authorized Redirect URIs

Di Google Cloud Console, tambahkan Authorized redirect URIs:

```
https://[YOUR-PROJECT].supabase.co/auth/v1/callback
```

**Contoh:**
```
https://xyzabc123.supabase.co/auth/v1/callback
```

> Cari project ID di Supabase Dashboard → Settings → General

### Step 4: Copy Credentials ke Supabase

1. Di Google Cloud Console, copy:
   - **Client ID**
   - **Client Secret**

2. Kembali ke Supabase Dashboard → Authentication → Providers → Google
3. Paste:
   - **Client ID** ke field "Client ID"
   - **Client Secret** ke field "Client secret"
4. Klik **Save**

### Step 5: Set Redirect URL (Opsional tapi Recommended)

Di Supabase Dashboard → Authentication → URL Configuration

Tambahkan:
- **Redirect URLs**: 
  - `http://localhost:5500/admin.html` (untuk development)
  - `https://yourdomain.com/admin.html` (untuk production)

---

## 🔐 Keamanan & Best Practices

### 1. Email Whitelist (Recommended)

Jika ingin restrict siapa aja yang bisa login:

```javascript
// Di function checkGoogleSession(), tambah:
const ALLOWED_EMAILS = [
    'admin@jkt48.com',
    'manager@jkt48.com'
];

if (session && session.user) {
    if (!ALLOWED_EMAILS.includes(session.user.email)) {
        showNotification('error', 'Access Denied', 'Email tidak terdaftar');
        await db.auth.signOut();
        return false;
    }
    // ... lanjut login
}
```

### 2. Database User Table (Advanced)

Untuk tracking admin yang login:

```sql
-- Di Supabase SQL Editor, buat table:
CREATE TABLE admin_users (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  email TEXT UNIQUE,
  name TEXT,
  role TEXT DEFAULT 'admin',
  created_at TIMESTAMP DEFAULT now(),
  last_login TIMESTAMP DEFAULT now()
);

-- Buat policy untuk security
ALTER TABLE admin_users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Only authenticated users" ON admin_users
  FOR SELECT USING (auth.uid() = id);
```

### 3. Session Timeout

Untuk auto-logout setelah X menit:

```javascript
// Di DOMContentLoaded, tambah:
const SESSION_TIMEOUT_MINUTES = 30;

setInterval(function() {
    var loginTime = localStorage.getItem('jkt48_admin_login_time');
    if (!loginTime) return;
    
    var elapsed = (new Date().getTime() - parseInt(loginTime)) / 1000 / 60;
    if (elapsed > SESSION_TIMEOUT_MINUTES) {
        // Auto logout
        confirmLogout();
    }
}, 60000); // Check setiap 1 menit
```

---

## 🧪 Testing

### Test 1: Google Login
1. Buka `admin.html`
2. Klik tombol **"Login dengan Google"**
3. Login dengan Google account
4. Seharusnya redirect kembali ke admin panel & auto-login

### Test 2: Password Fallback
1. Logout
2. Login dengan password (optional)
3. Harusnya tetap berfungsi

### Test 3: Session Persistence
1. Login dengan Google
2. Refresh halaman
3. Seharusnya tetap login (auto-restore session)

### Test 4: Logout
1. Klik logout
2. Session di Supabase auth + localStorage harus clear
3. Login modal harus muncul

---

## 🐛 Troubleshooting

### Error: "redirect_uri_mismatch"
**Penyebab:** Redirect URI di Google Cloud Console tidak match
**Solusi:** 
- Pastikan exact sama (case-sensitive)
- Gunakan HTTPS untuk production
- Format: `https://project.supabase.co/auth/v1/callback`

### Error: "Client is unauthorized"
**Penyebab:** Client ID/Secret salah atau belum di-copy
**Solusi:** 
- Re-copy credentials dari Google Cloud Console
- Pastikan enable Google OAuth di Supabase
- Clear browser cache & try again

### User tidak bisa login meski credentials benar
**Penyebab:** Google account belum di-authorize
**Solusi:** 
- Buka [Google OAuth Consent Screen](https://console.cloud.google.com/apis/consent)
- Set production status atau add test users
- Pastikan app belum under verification (untuk development, OK)

### Session timeout tidak work
**Penyebab:** localStorage corrupt atau session tidak tercatat
**Solusi:** 
```javascript
// Clear dan restart
localStorage.clear();
location.reload();
```

---

## 📊 Code Structure

### Authentication Flow

```
User klik "Login dengan Google"
    ↓
loginWithGoogle() dipanggil
    ↓
db.auth.signInWithOAuth({ provider: 'google' })
    ↓
Browser redirect ke Google login page
    ↓
User login → Google redirect kembali ke admin.html
    ↓
checkGoogleSession() cek session
    ↓
Jika valid → set localStorage & hide login modal
    ↓
loadData() & start admin panel
```

### Key Functions

| Function | Purpose |
|----------|---------|
| `loginWithGoogle()` | Trigger Google OAuth flow |
| `checkGoogleSession()` | Check & restore session setelah redirect |
| `confirmLogout()` | Logout dari auth + clear storage |
| `finishLogout()` | Reset form & show login modal |

---

## 📱 Mobile Support

Google OAuth bekerja sempurna di mobile. Pastikan:
- ✓ Redirect URI support http:// untuk localhost development
- ✓ HTTPS untuk production
- ✓ Viewport meta tag sudah ada (✓ sudah ada di admin.html)

---

## 🚀 Next Steps

1. **Setup Google Cloud Console** → Generate OAuth credentials
2. **Enable Google Provider** → Di Supabase Dashboard
3. **Test di localhost** → Coba login & logout
4. **Deploy ke production** → Update redirect URIs
5. *(Optional)* **Add email whitelist** → Restrict access
6. *(Optional)* **Setup session timeout** → Auto logout

---

## 📞 Support

- **Supabase Docs**: https://supabase.com/docs/guides/auth/social-login/auth-google
- **Google OAuth Docs**: https://developers.google.com/identity/protocols/oauth2
- **Troubleshooting**: Check browser console (F12 → Console tab)

---

**Last Updated:** April 2026  
**Tested with:** Supabase JS v2, Google OAuth 2.0
