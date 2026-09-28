# Mission 1 - Steps 3 & 4: Profile + Logout + 401 Handling

## 📋 Files Changed

---

### 1️⃣ **src/app/components/profile-page/profile-page.ts**

#### AVANT (Before):
```typescript
export class ProfilePageComponent {
  readonly auth = inject(AuthService);
  readonly form = new FormGroup({
    name: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
  });

  load(): void {
    this.auth.profile().subscribe({
      next: (user) => {
        console.debug('[ProfilePage] Profil chargé', user.id);
        this.form.setValue({ name: user.name });
      },
      error: (error) => console.error('[ProfilePage] Chargement impossible', error),
    });
  }

  save(): void {
    this.auth.update(this.form.getRawValue().name).subscribe({
      next: (user) => console.debug('[ProfilePage] Profil enregistré', user.id),
      error: (error) => console.error('[ProfilePage] Enregistrement impossible', error),
    });
  }
}
```

#### MAINTENANT (Now):
```typescript
export class ProfilePageComponent implements OnInit {
  readonly auth = inject(AuthService);
  
  readonly loadError = signal('');
  readonly saveError = signal('');
  readonly saving = signal(false);
  
  readonly form = new FormGroup({
    name: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
  });

  ngOnInit(): void {
    this.load();  // Auto-load on init
  }

  load(): void {
    this.loadError.set('');
    this.auth.profile().subscribe({
      next: (user) => {
        this.form.setValue({ name: user.name });
      },
      error: () => {
        this.loadError.set('Impossible de charger le profil');
      },
    });
  }

  save(): void {
    this.saveError.set('');
    this.saving.set(true);
    this.auth.update(this.form.getRawValue().name).subscribe({
      next: () => {
        this.saving.set(false);
      },
      error: () => {
        this.saving.set(false);
        this.saveError.set('Erreur lors de la sauvegarde');
      },
    });
  }
}
```

#### ✅ CE QU'ON A FIXÉ (What We Fixed):
- ✅ Implements `OnInit` - auto-load profile when page loads
- ✅ Added `loadError` Signal for load errors
- ✅ Added `saveError` Signal for save errors
- ✅ Added `saving` Signal to show loading state
- ✅ Better error handling (no console logs visible to users)

---

### 2️⃣ **src/app/components/profile-page/profile-page.html**

#### AVANT (Before):
```html
<section class="card">
  <h1>Mon profil</h1>
  <button type="button" (click)="load()">Charger mon profil</button>
  @if (auth.currentUser(); as user) {
    <p>
      <b>{{ user.name }}</b><br />{{ user.email }}<br />
      Membre depuis {{ user.createdAt.slice(0, 10) }}
    </p>
    <form [formGroup]="form" (ngSubmit)="save()">
      <label>Nouveau nom<input formControlName="name" /></label>
      <button type="submit">Enregistrer</button>
    </form>
  }
</section>
```

#### MAINTENANT (Now):
```html
<section class="card">
  <h1>Mon profil</h1>

  @if (loadError()) {
    <p class="error">{{ loadError() }}</p>
  }

  @if (auth.currentUser(); as user) {
    <div class="user-info">
      <p>
        <b>{{ user.name }}</b><br />
        <small>{{ user.email }}</small><br />
        <small>Membre depuis {{ user.createdAt.slice(0, 10) }}</small>
      </p>
    </div>

    <form [formGroup]="form" (ngSubmit)="save()">
      <div class="form-group">
        <label>Nouveau nom<input formControlName="name" /></label>
        @if (form.get('name')?.touched && form.get('name')?.invalid) {
          <span class="error-message">Le nom est requis</span>
        }
      </div>

      @if (saveError()) {
        <p class="error">{{ saveError() }}</p>
      }

      <button type="submit" [disabled]="form.invalid || saving()">
        {{ saving() ? 'Enregistrement...' : 'Enregistrer' }}
      </button>
    </form>
  } @else {
    <p>Chargement du profil...</p>
  }
</section>
```

#### ✅ CE QU'ON A FIXÉ:
- ✅ Removed manual "Charger mon profil" button (auto-loads now)
- ✅ Added load error display
- ✅ Added `user-info` styling section
- ✅ Added field validation message for name
- ✅ Added save error display
- ✅ Button shows loading state: "Enregistrement..."
- ✅ Submit disabled during save

---

### 3️⃣ **src/app/components/profile-page/profile-page.css**

#### AVANT (Before):
```css
:host { display: block; max-width: 700px; margin: 0 auto; }
```

#### MAINTENANT (Now):
```css
:host { display: block; max-width: 700px; margin: 0 auto; }

.user-info { margin: 1rem 0; padding: 1rem; background: #f8f9fa; border-radius: 4px; }
.user-info p { margin: 0; }
.user-info b { font-size: 1.1rem; }
.user-info small { color: #666; }

.form-group { margin-bottom: 1rem; display: flex; flex-direction: column; }
.form-group label { display: flex; flex-direction: column; margin-bottom: 0.5rem; }
.form-group input { padding: 0.5rem; border: 1px solid #ccc; border-radius: 4px; font-size: 1rem; }
.form-group input:focus { outline: none; border-color: #007bff; box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25); }

.error-message { color: #dc3545; font-size: 0.875rem; margin-top: 0.25rem; }
.error { color: #dc3545; margin: 1rem 0; padding: 0.75rem; background: #f8d7da; border: 1px solid #f5c6cb; border-radius: 4px; }
```

#### ✅ CE QU'ON A FIXÉ:
- ✅ Styled `.user-info` box (gray background)
- ✅ Styled form fields with proper spacing
- ✅ Added input focus styling
- ✅ Added error message & error box styling

---

### 4️⃣ **src/app/components/app/app.ts**

#### AVANT (Before):
```typescript
export class AppComponent {}
```

#### MAINTENANT (Now):
```typescript
export class AppComponent {
  readonly auth = inject(AuthService);
  private readonly router = inject(Router);

  logout(): void {
    this.auth.logout();
    void this.router.navigateByUrl('/login');
  }
}
```

#### ✅ CE QU'ON A FIXÉ:
- ✅ Injected AuthService to check if user is logged in
- ✅ Added logout() method that:
  - Calls auth.logout() (clears localStorage + Signals)
  - Redirects to /login page

---

### 5️⃣ **src/app/components/app/app.html**

#### AVANT (Before):
```html
<header>
  <div>
    <b>Guitar Practice Cloud</b>
    <small>Le cloud qui manque à votre ampli</small>
  </div>
  <nav aria-label="Navigation principale">
    <a routerLink="/tracks">Backing tracks</a>
    <a routerLink="/profile">Profil</a>
    <a routerLink="/login">Connexion</a>
  </nav>
</header>

<main>
  <router-outlet />
</main>
```

#### MAINTENANT (Now):
```html
<header>
  <div>
    <b>Guitar Practice Cloud</b>
    <small>Le cloud qui manque à votre ampli</small>
  </div>
  <nav aria-label="Navigation principale">
    @if (auth.token()) {
      <a routerLink="/tracks">Backing tracks</a>
      <a routerLink="/profile">Profil</a>
      <button type="button" class="logout-btn" (click)="logout()">Déconnexion</button>
    } @else {
      <a routerLink="/login">Connexion</a>
    }
  </nav>
</header>

<main>
  <router-outlet />
</main>
```

#### ✅ CE QU'ON A FIXÉ:
- ✅ Shows "Backing tracks" + "Profil" + "Déconnexion" button when logged in
- ✅ Shows only "Connexion" link when NOT logged in
- ✅ Logout button calls logout() method

---

### 6️⃣ **src/app/components/app/app.css**

#### AVANT (Before):
```css
:host { display: block; }
```

#### MAINTENANT (Now):
```css
:host { display: block; }

header { display: flex; justify-content: space-between; align-items: center; padding: 1rem; background: #f8f9fa; border-bottom: 1px solid #dee2e6; }
header > div { flex: 1; }
header b { display: block; font-size: 1.5rem; }
header small { color: #666; font-size: 0.9rem; }

nav { display: flex; gap: 1rem; align-items: center; }
nav a { text-decoration: none; color: #007bff; padding: 0.5rem 1rem; border-radius: 4px; transition: background 0.2s; }
nav a:hover { background: #e7f1ff; }

.logout-btn { background: #dc3545; color: white; border: none; padding: 0.5rem 1rem; border-radius: 4px; cursor: pointer; font-size: 1rem; transition: background 0.2s; }
.logout-btn:hover { background: #c82333; }
```

#### ✅ CE QU'ON A FIXÉ:
- ✅ Styled header (flexbox, spacing)
- ✅ Styled nav links (color, hover effect)
- ✅ Styled logout button (red, hover state)

---

### 7️⃣ **src/app/shared/interceptors/auth.interceptor.ts**

#### AVANT (Before):
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

#### MAINTENANT (Now):
```typescript
export const authInterceptor: HttpInterceptorFn = (request, next) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  const token = auth.token();

  const req = token
    ? request.clone({
        setHeaders: { Authorization: `Bearer ${token}` },
      })
    : request;

  return next(req).pipe(
    catchError((error) => {
      if (error.status === 401) {
        auth.logout();
        void router.navigateByUrl('/login');
      }
      return throwError(() => error);
    }),
  );
};
```

#### ✅ CE QU'ON A FIXÉ:
- ✅ Added error handling with `catchError()`
- ✅ If API returns 401 (Unauthorized):
  - Logout user (clear localStorage + Signals)
  - Redirect to /login page

---

## 📊 RÉSUMÉ (Summary)

| Feature | Before | After |
|---------|--------|-------|
| Profile load | Manual button | Auto-load on page load |
| Error display | Console only | User sees error messages |
| Save state | No loading indicator | Shows "Enregistrement..." |
| Logout | No logout button | Red "Déconnexion" button in header |
| Conditional nav | Always shows all links | Shows links only when logged in |
| 401 handling | No handling | Auto-logout + redirect to login |

---

## 🎯 RÉSULTAT (Result)

✅ **Profile page auto-loads user data**
✅ **Users can update their name with validation**
✅ **Logout button in header with clean state**
✅ **Navigation shows only relevant links based on login state**
✅ **401 errors automatically logout and redirect**
✅ **Better UX with loading states and error messages**
