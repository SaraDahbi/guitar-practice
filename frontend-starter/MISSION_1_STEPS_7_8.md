# Mission 1 - Steps 7 & 8: Testing & Network Verification

## 📋 Step 7: Network Testing Checklist

### How to Test in DevTools

1. **Open DevTools:** F12
2. **Go to Network tab**
3. **Filter:** XHR/fetch
4. **Then test each scenario below**

---

## ✅ Test 1: Successful Login

### Action:
Navigate to `/login` → Fill demo credentials → Click "Se connecter"

```
Email: demo@example.com
Password: Demo1234!
```

### Expected in Network:

**Request:**
```
Method: POST
URL: /api/auth/login
Headers: (No Authorization - public endpoint)
Body: {
  "email": "demo@example.com",
  "password": "Demo1234!"
}
```

**Response:**
```
Status: 200
Headers: Content-Type: application/json
Body: {
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "...",
    "name": "Demo User",
    "email": "demo@example.com",
    "createdAt": "2026-01-15T10:30:00Z"
  }
}
```

### What Changes in App:
- ✅ localStorage gets `gpc_token`
- ✅ Signals updated: `token()`, `currentUser()`
- ✅ Redirects to `/tracks`
- ✅ Header shows: "Backing tracks | Profil | Déconnexion"

---

## ❌ Test 2: Failed Login (Wrong Password)

### Action:
Navigate to `/login` → Wrong credentials → Click "Se connecter"

```
Email: demo@example.com
Password: WrongPassword
```

### Expected in Network:

**Request:**
```
Method: POST
URL: /api/auth/login
Body: {
  "email": "demo@example.com",
  "password": "WrongPassword"
}
```

**Response:**
```
Status: 401
Body: {
  "message": "Invalid credentials"
}
```

### What Changes in App:
- ✅ Error message appears: "Invalid credentials"
- ✅ Form NOT submitted
- ✅ NO redirect (stays on /login)
- ✅ localStorage unchanged

---

## 🔒 Test 3: Protected Route (GET /api/users/me)

### Action:
Login successfully → Navigate to `/profile`

### Expected in Network:

**Request:**
```
Method: GET
URL: /api/users/me
Headers: 
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```
Status: 200
Body: {
  "id": "...",
  "name": "Demo User",
  "email": "demo@example.com",
  "createdAt": "2026-01-15T10:30:00Z"
}
```

### What Changes in App:
- ✅ Profile page loads user info
- ✅ Shows: Name, Email, Join Date
- ✅ Can edit name

---

## ✏️ Test 4: Profile Update (PUT /api/users/me)

### Action:
On profile page → Change name → Click "Enregistrer"

```
Old name: Demo User
New name: Mon Nouveau Nom
```

### Expected in Network:

**Request:**
```
Method: PUT
URL: /api/users/me
Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Body: {
  "name": "Mon Nouveau Nom"
}
```

**Response:**
```
Status: 200
Body: {
  "id": "...",
  "name": "Mon Nouveau Nom",
  "email": "demo@example.com",
  "createdAt": "2026-01-15T10:30:00Z"
}
```

### What Changes in App:
- ✅ Name updated on page
- ✅ Button shows "Enregistrement..." during save
- ✅ Signals updated

---

## 🚪 Test 5: 401 Error (Expired Token)

### Action:
Login → Open DevTools → Go to Application → localStorage → Delete `gpc_token` → Go to `/profile` → Try to access profile

### Expected in Network:

**Request:**
```
Method: GET
URL: /api/users/me
Headers: 
  Authorization: Bearer null  (or missing)
```

**Response:**
```
Status: 401
Body: {
  "message": "Unauthorized"
}
```

### What Changes in App:
- ✅ authInterceptor catches 401
- ✅ Calls `auth.logout()`
- ✅ Redirects to `/login`
- ✅ Header shows only: "Connexion" link

---

## 📤 Test 6: File Upload (POST /api/tracks)

### Action:
Go to `/tracks` → Select audio file → Enter title → Click "Envoyer"

### Expected in Network:

**Request:**
```
Method: POST
URL: /api/tracks
Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Body: FormData
  - audio: [File object]
  - title: "My Track"
```

**Response:**
```
Status: 201
Body: {
  "id": "...",
  "title": "My Track",
  "originalName": "song.mp3",
  "size": 2048,
  "createdAt": "2026-09-27T..."
}
```

### What Changes in App:
- ✅ Button shows "Envoi..." while uploading
- ✅ Track appears in list after upload
- ✅ Form clears (title + file)

---

## 🎵 Test 7: Audio Playback (GET /api/tracks/:id/audio)

### Action:
On `/tracks` → Click play button on a track

### Expected in Network:

**Request:**
```
Method: GET
URL: /api/tracks/[track-id]/audio
Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```
Status: 200
Type: audio/mp3 (blob)
(Binary audio data)
```

### What Changes in App:
- ✅ Audio player appears
- ✅ Can play/pause
- ✅ Audio streams from server

---

## 📝 Step 8: Complete Testing Report

### Checklist to Verify

- [ ] **Registration works**
  - Navigate to `/register`
  - Fill: name, email, password
  - Submit
  - Check: redirects to profile? Token in localStorage?

- [ ] **Login works (correct creds)**
  - Navigate to `/login`
  - Fill: demo@example.com, Demo1234!
  - Submit
  - Check: redirects to /tracks? Header shows logout?

- [ ] **Login fails (wrong creds)**
  - Try wrong password
  - Check: Error message shown? No redirect?

- [ ] **Protected routes work**
  - Login → Navigate to /profile
  - Check: User data loads? Can edit name?

- [ ] **401 handling works**
  - Login → Delete token from localStorage
  - Try to access /profile
  - Check: Redirects to /login? Token cleared?

- [ ] **Logout works**
  - Login → Click "Déconnexion"
  - Check: Redirects to /login? Token deleted? Header shows only "Connexion"?

- [ ] **Tracks page works**
  - Login → Go to /tracks
  - Upload file
  - Check: File appears in list? Can play?

- [ ] **Network headers correct**
  - Check all protected routes have: `Authorization: Bearer [token]`
  - Check login/register have NO Authorization header

---

## 🔍 What to Look For in Network Tab

### ✅ Correct Requests:
```
POST /api/auth/login - no auth header
GET /api/users/me - has auth header
PUT /api/users/me - has auth header
POST /api/tracks - has auth header
GET /api/tracks/:id/audio - has auth header
```

### ❌ Common Mistakes:
- Missing Authorization header on protected routes
- Token visible in console logs
- Password visible in Network requests
- Token not persisted in localStorage
- No error messages shown to user

---

## 📊 Summary Table

| Test | Endpoint | Method | Needs Token | Expected Status |
|------|----------|--------|-------------|-----------------|
| Login | /api/auth/login | POST | ❌ | 200 ✅ or 401 ❌ |
| Register | /api/auth/register | POST | ❌ | 201 ✅ or 409 ❌ |
| Get Profile | /api/users/me | GET | ✅ | 200 ✅ or 401 ❌ |
| Update Profile | /api/users/me | PUT | ✅ | 200 ✅ |
| List Tracks | /api/tracks | GET | ✅ | 200 ✅ |
| Upload Track | /api/tracks | POST | ✅ | 201 ✅ |
| Play Audio | /api/tracks/:id/audio | GET | ✅ | 200 ✅ |

---

## 🎯 Final Verification

Run through all 7 tests and check:

✅ **Functionality:**
- All features work as expected
- Errors shown to user
- Loading states visible

✅ **Security:**
- Token never logged to console
- Token in localStorage
- Token in Authorization header
- 401 errors handled

✅ **UX:**
- Error messages clear
- Loading indicators shown
- Buttons disabled appropriately
- Redirects work

✅ **Data:**
- User info persists
- Tracks list updates
- Changes saved

---

**Status:** 🚀 Ready for final testing
