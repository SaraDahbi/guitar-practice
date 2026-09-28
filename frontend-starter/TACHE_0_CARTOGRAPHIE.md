# Tâche 0 - Cartographie de l'Application

## 📋 Résumé Exécutif

Le projet est une application **Angular + Express** pour gérer l'authentification et les profils utilisateurs d'un portail Guitar Amp.

**Flux de données:** Angular Component → Service → HttpClient → Express API → MongoDB

---

## 1️⃣ Composant Racine

**Fichier:** `src/app/components/app/app.ts`

```typescript
@Component({
  selector: 'app-root',
  imports: [RouterLink, RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {}
```

- Composant **standalone** (Angular 22)
- Utilise `RouterOutlet` pour afficher les routes
- Point d'entrée de toute l'application

---

## 2️⃣ Configuration des Routes

**Fichier:** `src/app/routes.ts`

| Route | Composant | Protégée | Description |
|-------|-----------|----------|-------------|
| `/login` | LoginPageComponent | ❌ Non | Formulaire de connexion |
| `/register` | RegisterPageComponent | ❌ Non | Formulaire d'inscription |
| `/profile` | ProfilePageComponent | ✅ Oui | Profil utilisateur + modification nom |
| `/tracks` | TracksPageComponent | ✅ Oui | Bibliothèque audio |
| `/` | → `/tracks` | - | Redirection par défaut |

**Authentification:** Routes protégées utilisent `authGuard` qui redirige vers `/login` si pas de token.

---

## 3️⃣ Enregistrement de HttpClient

**Fichier:** `src/main.ts`

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
  ],
}).catch(console.error);
```

- **HttpClient** enregistré au niveau racine (disponible partout via `inject()`)
- **Interceptor:** `authInterceptor` appliqué à TOUTES les requêtes
- **Mode standalone:** Pas de NgModule

---

## 4️⃣ Mécanisme d'Authentification JWT

### 🔐 AuthInterceptor (Ajoute le token)

**Fichier:** `src/app/shared/interceptors/auth.interceptor.ts`

```typescript
export const authInterceptor: HttpInterceptorFn = (request, next) => {
  const token = inject(AuthService).token();
  return next(
    token
      ? request.clone({
          setHeaders: { Authorization: `Bearer ${token}` },
        })
      : request,
  );
};
```

**Ce qu'il fait:**
- Lit le token Signal depuis `AuthService.token()`
- Si token existe → ajoute `Authorization: Bearer <token>` à la requête
- Si pas de token → envoie la requête sans header (pour `/login` et `/register`)

**Appliqué à:** TOUTES les requêtes HTTP automatiquement

---

### 🛡️ AuthGuard (Protège les routes)

**Fichier:** `src/app/shared/guards/auth.guard.ts`

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.token() ? true : router.createUrlTree(['/login']);
};
```

**Ce qu'il fait:**
- Vérifie si `AuthService.token()` existe
- Si oui → permet l'accès à la route
- Si non → redirige vers `/login`

---

### 📦 AuthService (Gère l'état d'authentification)

**Fichier:** `src/app/shared/services/auth.service.ts`

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  readonly currentUser = signal<User | null>(null);
  readonly token = signal<string | null>(localStorage.getItem('gpc_token'));

  login(email: string, password: string) { /* POST /api/auth/login */ }
  register(name: string, email: string, password: string) { /* POST /api/auth/register */ }
  profile() { /* GET /api/users/me */ }
  update(name: string) { /* PUT /api/users/me */ }
  logout(): void { /* Nettoie tout */ }
}
```

**Responsabilités:**
- Stocke le JWT dans un **Signal** `token`
- Sauvegarde le JWT dans `localStorage` sous la clé `'gpc_token'`
- Stocke l'utilisateur connecté dans un **Signal** `currentUser`
- Appelle les endpoints d'authentification

---

## 5️⃣ Modèles de Données

### User
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: string;
}
```

### AuthResponse
```typescript
interface AuthResponse {
  token: string;  // JWT à stocker
  user: User;     // Données utilisateur
}
```

---

## 6️⃣ Flux de Connexion (Schéma Annoté)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ 1. UTILISATEUR CLIQUE "Se connecter"                                    │
│    - Remplit formulaire (email + password)                              │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────────┐
│ 2. LOGINPAGECOMPONENT (Composant)                                       │
│    - Valide le formulaire (Reactive Form)                               │
│    - Appelle authService.login(email, password)                         │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────────┐
│ 3. AUTHSERVICE.LOGIN() (Service)                                        │
│    - Crée un Observable                                                 │
│    - httpClient.post<AuthResponse>('/api/auth/login', {email,password}) │
│    - Retourne Observable avec tap() pour stocker le résultat            │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────────┐
│ 4. HTTPCLIENT (Angular Core)                                            │
│    - Crée une requête POST vers /api/auth/login                         │
│    ✋ INTERCEPTOR INTERCEPTE LA REQUÊTE                                  │
│       (Pas de token pour login → requête envoyée sans header)           │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────────┐
│ 5. API EXPRESS (Backend)                                                │
│    POST /api/auth/login {email, password}                               │
│    ↓ Vérifie les credentials                                            │
│    ↓ Génère un JWT                                                      │
│    ↓ Retourne { token: "eyJhb...", user: {...} }                        │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────────┐
│ 6. RESPONSE OBSERVABLE                                                  │
│    - authService.storeAuthentication(response)                          │
│      • localStorage.setItem('gpc_token', response.token)                │
│      • token.set(response.token)  [Signal]                              │
│      • currentUser.set(response.user)  [Signal]                         │
└─────────────────────┬───────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────────────┐
│ 7. COMPOSANT SE SUBSCRIBE                                               │
│    - next: () => router.navigateByUrl('/tracks')                        │
│    - error: (e) => affiche message d'erreur                             │
│                                                                          │
│ 🎯 RÉSULTAT: Utilisateur redirigé vers /tracks                         │
│    - Token disponible dans tous les Services                            │
│    - Prochaines requêtes ajoutent automatiquement Authorization header   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Routes API (Contrat HTTP)

### Routes Publiques (Sans JWT)
| Méthode | Route | Requête | Réponse |
|---------|-------|---------|---------|
| POST | `/api/auth/register` | `{name,email,password}` | `201 {token,user}` |
| POST | `/api/auth/login` | `{email,password}` | `200 {token,user}` |
| GET | `/api/health` | - | `{status:"ok"}` |

### Routes Protégées (Require JWT)
| Méthode | Route | Requête | Réponse |
|---------|-------|---------|---------|
| GET | `/api/users/me` | Authorization header | `200 User` |
| PUT | `/api/users/me` | `{name}` + JWT | `200 User` |
| GET | `/api/tracks?page=1&limit=5` | JWT | `Page<Track>` |
| POST | `/api/tracks` | multipart + JWT | `201 Track` |
| GET | `/api/tracks/:id/audio` | JWT | Flux audio |

---

## 8️⃣ Structure de Fichiers

```
src/app/
├── components/
│   ├── app/                      # Composant racine
│   │   ├── app.ts
│   │   ├── app.html
│   │   └── app.css
│   ├── login-page/               # Page de connexion
│   ├── register-page/            # Page d'inscription
│   ├── profile-page/             # Page profil (protégée)
│   └── tracks-page/              # Bibliothèque audio (protégée)
├── shared/
│   ├── services/
│   │   ├── auth.service.ts       # Authentification & état
│   │   └── track.service.ts      # Gestion audio
│   ├── interceptors/
│   │   └── auth.interceptor.ts   # Ajoute le JWT
│   ├── guards/
│   │   └── auth.guard.ts         # Protège les routes
│   └── models/
│       ├── user.model.ts
│       ├── auth-response.model.ts
│       ├── track.model.ts
│       └── page.model.ts
├── routes.ts                     # Configuration des routes
└── main.ts                       # Bootstrap & setup
```

---

## 9️⃣ Points Clés

### ✅ Signal vs localStorage
- **Signal (`token()`):** Valeur réactive, détecte les changements
- **localStorage:** Persiste entre les sessions de navigateur
- **Lien:** Signal est initialisé depuis localStorage au démarrage

### ✅ Pourquoi 2 Signaux?
1. **`token()`** - Signal réactif, déclenche les changements dans les composants
2. **`currentUser()`** - Affiche les infos utilisateur dans le profil et la navbar

### ✅ Sécurité
- Token JAMAIS affiché dans les logs
- Token JAMAIS commité dans Git
- Token stocké en localStorage (safe pour une app simple)
- Authorization header ajouté automatiquement par l'interceptor

---

## 🔟 Prochaines Étapes (Tâche 1)

Pour la **Mission 1**, vous devrez:
1. ✅ Ajouter le formulaire de registration
2. ✅ Compléter le formulaire de connexion (déjà en place)
3. ✅ Ajouter la page de profil avec modification du nom
4. ✅ Ajouter les validations et messages d'erreur
5. ✅ Gérer les erreurs 401 avec redirection
6. ✅ Tester dans Network DevTools

---

**Date:** 2026-09-27
**Statut:** ✅ Cartographie terminée
