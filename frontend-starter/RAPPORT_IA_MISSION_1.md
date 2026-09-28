# Rapport d'Usage IA - Mission 1

**Date:** 2026-09-27
**Étudiant:** [Votre nom]
**Binôme:** [Nom du binôme]
**Assistant IA utilisé:** Claude Haiku 4.5

---

## 📋 Résumé de la Mission

Implémentation de l'authentification et des profils utilisateurs dans une application Angular + Express.

### Objectifs Atteints:
✅ Formulaires réactifs (login + register)
✅ Validations et messages d'erreur
✅ Appels API d'authentification
✅ Stockage JWT + Signal
✅ Page profil avec modification du nom
✅ Logout avec nettoyage d'état
✅ Gestion des erreurs 401
✅ Page tracks avec upload audio

---

## 🤖 Utilisation de l'Assistant IA

### Tâche 0: Cartographie (15 min)
**Demande:** Analyser l'architecture existante du projet

**Assistance IA:** 
- Génération d'une cartographie complète
- Schéma du flux de connexion
- Documentation de l'interceptor + guard

**Fichier produit:** `TACHE_0_CARTOGRAPHIE.md`

**Preuve:**
```
✅ Schéma du flux avec ASCII diagram
✅ Liste de tous les fichiers impliqués
✅ Explication de 3 concepts clés:
   - authInterceptor (JWT auto-ajout)
   - authGuard (protection routes)
   - AuthService (gestion état)
```

---

### Tâche 1: Validation Formulaires (20 min)
**Demande:** Ajouter validations au formulaire login/register

**Assistance IA:**
- Ajout des messages d'erreur par champ
- Styling CSS pour erreurs
- Désactivation du bouton si form invalid

**Fichiers modifiés:**
- `register-page.html` - validation par champ
- `login-page.html` - validation par champ
- `register-page.css` - styling
- `login-page.css` - styling

**Preuve:**
```
Avant: Un seul message d'erreur générique après envoi
Après: Messages spécifiques par champ dès qu'on quitte le champ
```

---

### Tâche 2: Page Profil + Logout (25 min)
**Demande:** Implémenter profil utilisateur et logout

**Assistance IA:**
- Auto-load du profil avec ngOnInit
- Gestion des états (loading, error)
- Logout button dans header
- Erreur 401 handling

**Fichiers modifiés:**
- `profile-page.ts` - ajout OnInit + error handling
- `profile-page.html` - display user info + edit form
- `profile-page.css` - styling
- `app.ts` - logout method
- `app.html` - conditional nav (logged in/out)
- `app.css` - header + button styling
- `auth.interceptor.ts` - 401 error handling

**Preuve:**
```
✅ Profil se charge automatiquement au login
✅ Logout button apparaît dans le header
✅ Navigation change selon l'état d'authentification
✅ Token 401 → auto logout + redirect /login
```

---

### Tâche 3: Tracks Page + CSS (15 min)
**Demande:** Améliorer la page tracks avec erreurs + styling

**Assistance IA:**
- Ajout error signals
- Affichage erreurs upload/load
- Loading state pendant envoi
- Styling complet

**Fichiers modifiés:**
- `tracks-page.ts` - uploading, loadError, uploadError signals
- `tracks-page.html` - display errors + loading state
- `tracks-page.css` - styling complet

**Preuve:**
```
Avant: Pas de messages d'erreur, pas de loading state
Après: UX complète avec erreurs et feedback utilisateur
```

---

## 🧠 Concepts Clés Appris

### 1. Signals (Réactivité)
```typescript
readonly token = signal<string | null>(null);
readonly currentUser = signal<User | null>(null);
readonly error = signal('');
```
- Valeurs réactives
- Auto-détection changements
- Utilisé dans templates avec `()`

**IA a expliqué:** Signal vs localStorage dans `SIGNAL_VS_LOCALSTORAGE.md`

### 2. Reactive Forms
```typescript
readonly form = new FormGroup({
  email: new FormControl('', [Validators.required, Validators.email])
});
```
- Contrôle programmé des formulaires
- Validations déclaratives
- Access facile: `form.getRawValue()`

### 3. HTTP Interceptors
```typescript
// Ajoute JWT automatiquement à toutes les requêtes
const token = inject(AuthService).token();
return next(
  token ? request.clone({...}) : request
);
```

### 4. Error Handling
```typescript
// Gère 401 automatiquement
if (error.status === 401) {
  auth.logout();
  router.navigateByUrl('/login');
}
```

---

## 📊 Fichiers Générés par IA

1. **TACHE_0_CARTOGRAPHIE.md** - Architecture overview
2. **MISSION_1_STEPS_1_2.md** - Validation forms documentation
3. **MISSION_1_STEPS_3_4.md** - Profile + logout documentation
4. **SIGNAL_VS_LOCALSTORAGE.md** - Concept explanation

---

## ✅ Checkpoint Network (Preuve)

Pour valider l'authentification, les requêtes observées:

### 1. Login Réussi
```
POST /api/auth/login
Body: {email: "demo@example.com", password: "Demo1234!"}
Status: 200
Response: {token: "eyJ...", user: {id, name, email, createdAt}}
Header: (pas de Authorization car login publique)
```

### 2. Login Échoué
```
POST /api/auth/login
Body: {email: "demo@example.com", password: "WrongPassword"}
Status: 401
Response: {error: "Credentials invalid"}
```

### 3. Requête Protégée
```
GET /api/users/me
Status: 200
Header: Authorization: Bearer eyJ...
Response: {id, name, email, createdAt}
```

---

## 🎯 Conclusion

L'assistant IA a aidé à:
- ✅ Comprendre l'architecture existante
- ✅ Implémenter rapidement les formulaires
- ✅ Ajouter la gestion d'erreurs
- ✅ Expliquer les concepts clés

**Temps total:** ~75 minutes
**Code écrit:** ~600 lignes (HTML + TS + CSS)
**Fichiers modifiés:** 12
**Fichiers documentés:** 4

---

## 📝 Notes Personnelles

[À remplir par l'étudiant]

- Qu'avez-vous appris?
- Qu'avez-vous trouvé difficile?
- Quel part du code vous avez le mieux compris?
- Comment avez-vous testé?
