# How to Run & Test Mission 1

## 🚀 Quick Start

### 1. Start Backend
```bash
cd backend
npm start
```
Check: `http://localhost:3000/api/health` → `{status: "ok"}`

### 2. Start Frontend
```bash
cd frontend-starter
npm start
```
Opens: `http://localhost:4200`

---

## 🧪 Test Scenarios (5 minutes each)

### Scenario 1: Register New User
```
1. Click "Créer un compte"
2. Fill:
   - Nom: Your Name
   - Email: yourname@example.com
   - Mot de passe: Password123!
3. Click "Créer mon compte"
4. Should redirect to profile page
5. Check localStorage has `gpc_token`
```

### Scenario 2: Login with Demo Account
```
1. Go to /login
2. Fill:
   - Email: demo@example.com
   - Password: Demo1234!
3. Click "Se connecter"
4. Should show profile/logout in header
5. Redirects to /tracks
```

### Scenario 3: Edit Profile
```
1. Click "Profil" in header
2. Profile loads automatically
3. Change name in form
4. Click "Enregistrer"
5. Name updates (button shows "Enregistrement...")
```

### Scenario 4: Upload Audio File
```
1. Go to /tracks
2. Select audio file (MP3/WAV/OGG)
3. Enter title
4. Click "Envoyer"
5. File appears in list (button shows "Envoi...")
```

### Scenario 5: Logout
```
1. Click "Déconnexion" button
2. Redirects to /login
3. localStorage cleared (`gpc_token` gone)
4. Header shows only "Connexion" link
```

---

## 🔍 DevTools Network Inspection

### Open DevTools Network Tab
1. **Press F12**
2. **Click "Network" tab**
3. **Filter: XHR/fetch**
4. **Perform actions above**

### What to Verify

#### Login Request:
```
✅ Method: POST
✅ URL: /api/auth/login
✅ No Authorization header (public)
✅ Status: 200 (or 401 if wrong password)
✅ Response has token
```

#### Protected Route Request:
```
✅ Method: GET/PUT/POST
✅ URL: /api/users/me or /api/tracks
✅ Authorization header present
✅ Authorization: Bearer [token]
✅ Status: 200 (or 401 if token expired)
```

---

## ⚙️ Key Files to Know

### Frontend Structure:
```
frontend-starter/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── app/ (root)
│   │   │   ├── login-page/
│   │   │   ├── register-page/
│   │   │   ├── profile-page/
│   │   │   └── tracks-page/
│   │   ├── shared/
│   │   │   ├── services/
│   │   │   │   ├── auth.service.ts
│   │   │   │   └── track.service.ts
│   │   │   ├── interceptors/
│   │   │   │   └── auth.interceptor.ts
│   │   │   ├── guards/
│   │   │   │   └── auth.guard.ts
│   │   │   └── models/
│   │   ├── routes.ts
│   │   └── main.ts
```

### Important Files Changed:
- `auth.service.ts` - Login/Register/Profile API calls
- `auth.interceptor.ts` - Adds JWT + handles 401
- `auth.guard.ts` - Protects routes
- All `*-page.html/.ts/.css` files

---

## 🐛 Debugging Tips

### Issue: Login doesn't work
**Check:**
1. Backend running? (`http://localhost:3000/api/health`)
2. Email/password correct? (demo@example.com / Demo1234!)
3. Network tab shows 401? = Wrong creds
4. Network tab shows error? = Backend issue

### Issue: Profile won't load
**Check:**
1. Logged in? Check if token in localStorage
2. Network shows Authorization header?
3. Response status 401? = Token expired → logout
4. Network shows 500? = Backend error

### Issue: Upload fails
**Check:**
1. File selected? Button should be enabled
2. File type valid? (MP3, WAV, OGG, M4A only)
3. File size < 25MB?
4. Network shows 201 response? = Success

### Issue: Token not persisting
**Check:**
1. localStorage enabled in browser?
2. Check DevTools → Application → Storage → localStorage
3. See `gpc_token` key? = Good
4. Click logout → cleared? = Good

---

## ✅ Verification Checklist

Run through this before submitting:

### Authentication
- [ ] Register page has validation messages
- [ ] Login page has validation messages
- [ ] Login redirects to /tracks
- [ ] Register redirects to /profile
- [ ] Wrong password shows error
- [ ] Token saved to localStorage

### Profile
- [ ] Profile auto-loads on /profile page
- [ ] Can edit name
- [ ] Save shows "Enregistrement..."
- [ ] Name updates in Signal

### Navigation
- [ ] Header shows "Connexion" when logged out
- [ ] Header shows "Backing tracks | Profil | Déconnexion" when logged in
- [ ] Logout button works
- [ ] Logout clears localStorage + token

### Tracks
- [ ] Can upload audio file
- [ ] Upload button shows "Envoi..."
- [ ] File appears in list
- [ ] Can play audio
- [ ] Pagination works
- [ ] Refresh button works

### Security
- [ ] No password in console
- [ ] No token in console
- [ ] No token in Network request body (only in header)
- [ ] 401 errors redirect to /login
- [ ] Protected routes redirect to /login if no token

### Styling
- [ ] Error messages are visible
- [ ] Buttons have hover effects
- [ ] Forms are properly spaced
- [ ] Audio player shows
- [ ] Layout responsive

---

## 📸 Screenshots to Take (For Report)

1. **Successful Login**
   - Show Network tab with POST /api/auth/login
   - Show token in response (without full token, just first few chars)

2. **Failed Login**
   - Show Network tab with 401 status
   - Show error message on page

3. **Protected Route**
   - Show Network tab with Authorization header
   - Show user data loaded

4. **Profile Update**
   - Show Network tab with PUT /api/users/me
   - Show updated name on page

---

## 📝 Notes

- Demo account: `demo@example.com` / `Demo1234!`
- Backend API base: `http://localhost:3000/api`
- Frontend proxy: `proxy.conf.json` handles `/api` redirects
- All timestamps in ISO 8601 format
- File size shown in KB

---

## 🎯 Success Criteria

**Mission 1 is complete when:**
1. ✅ All 4 pages work (login, register, profile, tracks)
2. ✅ All validations show errors
3. ✅ All API calls have Authorization header
4. ✅ Profile auto-loads
5. ✅ Logout clears state
6. ✅ 401 errors redirect to login
7. ✅ Upload/download works
8. ✅ All files documented

---

**Ready?** Start with `npm start` in both backend & frontend! 🚀
