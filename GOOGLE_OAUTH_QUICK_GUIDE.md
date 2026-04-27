# 🚀 Quick Setup Guide - Google OAuth (5 Menit)

## Langkah Cepat

### 1️⃣ Google Cloud Console
```
🔗 https://console.cloud.google.com/
├─ Buat project baru (atau gunakan existing)
├─ APIs & Services → Credentials
├─ + Create Credentials → OAuth 2.0 Client ID
├─ Web application
└─ Authorized redirect URIs:
   └─ https://[PROJECT-ID].supabase.co/auth/v1/callback
```

Dapat: **Client ID** dan **Client Secret**

### 2️⃣ Supabase Dashboard
```
🔗 https://app.supabase.com/
├─ Authentication → Providers
├─ Google → Enable
├─ Paste Client ID
├─ Paste Client Secret
└─ Save
```

### 3️⃣ Supabase Authentication → URL Configuration
```
Redirect URLs:
├─ http://localhost:5500/admin.html (development)
└─ https://yourdomain.com/admin.html (production)
```

### 4️⃣ Test
```
1. Buka admin.html
2. Klik "Login dengan Google"
3. Login dengan akun Google
4. Seharusnya auto-redirect ke dashboard
```

---

## Apa yang Berubah di admin.html?

### 🎨 UI Changes
- **Google OAuth Button** di login form
- Separator "atau" antara Google & password login
- Support for both OAuth dan password auth

### 🔧 Code Changes
```javascript
// New Functions:
loginWithGoogle()           // Trigger OAuth
checkGoogleSession()        // Auto-login setelah redirect
finishLogout()             // Reset form

// Modified:
confirmLogout()            // Sekarang handle OAuth logout
DOMContentLoaded()         // Sekarang cek Google session
```

### 💾 localStorage Changes
```javascript
// Sebelumnya:
jkt48_admin_email
jkt48_admin_login_time

// Sekarang tambahan:
jkt48_admin_oauth_provider  // "google" atau undefined (password)
```

---

## Fitur-Fitur

✅ **Google OAuth Login** - Secure OAuth 2.0 via Supabase  
✅ **Session Persistence** - Auto-restore login setelah refresh  
✅ **Fallback Password** - Support password login sebagai backup  
✅ **Proper Logout** - Clear session dari Supabase auth  
✅ **Provider Detection** - Tahu user login via mana  
✅ **Error Handling** - User-friendly error messages  

---

## Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| "redirect_uri_mismatch" | Copy exact URI dari Supabase ke Google Cloud |
| User tidak bisa login | Enable Google OAuth Consent Screen di Google Cloud |
| Session tidak restore | Check browser allow localStorage |
| "Client is unauthorized" | Re-copy Client ID/Secret |

---

## Support Docs

| Resource | Link |
|----------|------|
| Supabase Auth Docs | https://supabase.com/docs/guides/auth |
| Google OAuth Guide | https://developers.google.com/identity/protocols/oauth2 |
| Full Setup Guide | Lihat SETUP_GOOGLE_OAUTH.md |

---

## Next: Advanced Features (Optional)

### Email Whitelist
```javascript
// Restrict login ke specific emails
const ALLOWED_EMAILS = ['admin@company.com'];
```

### Session Timeout
```javascript
// Auto-logout setelah 30 menit
const SESSION_TIMEOUT_MINUTES = 30;
```

### Admin Audit Log
```javascript
// Track siapa login, kapan, dari mana
CREATE TABLE admin_audit_log (
  id UUID PRIMARY KEY,
  email TEXT,
  provider TEXT,
  login_time TIMESTAMP,
  ip_address TEXT
);
```

---

## File Update Summary

| File | Status | Changes |
|------|--------|---------|
| admin.html | ✅ Updated | Added Google OAuth + functions |
| SETUP_GOOGLE_OAUTH.md | 📄 New | Full documentation |
| GOOGLE_OAUTH_QUICK_GUIDE.md | 📄 New | This file |

---

**Status:** ✅ Ready to Deploy  
**Tested:** ✓ Supabase Auth, ✓ Google OAuth, ✓ Session Management  
**Time to Setup:** ~5 minutes
