# Signal vs localStorage - Explication

## 📊 Comparaison

| Aspect | Signal | localStorage |
|--------|--------|---------------|
| **Réactivité** | Réactif (détecte changements) | Passif (stockage brut) |
| **Persiste entre sessions** | ❌ Non | ✅ Oui |
| **Visibilité** | Disponible dans composant | Disponible partout (global) |
| **Performance** | Très rapide (en mémoire) | Plus lent (stockage disque) |
| **Synchronisation** | Auto entre composants | Besoin de sync manuelle |
| **Taille limite** | ~5 MB | ~5-10 MB |
| **Use case** | État de l'app | Données persistantes |

---

## 🔐 Cas d'Usage dans AuthService

### localStorage: Persistence
```typescript
// Sauvegarde le JWT sur disque pour survivre aux rechargements
localStorage.setItem('gpc_token', response.token);
```

**Pourquoi?**
- L'utilisateur rafraîchit la page → token disparu sans localStorage
- Token persiste même après fermer/rouvrir le navigateur
- Permet les "se souvenir de moi" fonctionnalités

---

### Signal: Réactivité
```typescript
readonly token = signal<string | null>(localStorage.getItem('gpc_token'));
```

**Pourquoi?**
- Composants réagissent IMMÉDIATEMENT quand token change
- Les templates rerender automatiquement
- L'interceptor peut accéder la valeur actuelle: `token()`

---

## 🔄 Flux Complet

```
1. Page charge → Signal lit localStorage
   token = signal(localStorage.getItem('gpc_token'))

2. Utilisateur se connecte
   - API retourne { token: "...", user: {...} }

3. Nous sauvegardons DEUX fois:
   localStorage.setItem('gpc_token', token)  // Persistence
   token.set(token)                          // Réactivité

4. Composants voient le changement via Signal
   @if (auth.token()) { /* affiche logout */ }

5. Chaque requête HTTP utilise Signal
   const token = inject(AuthService).token()  // Valeur actuelle
   Authorization: Bearer ${token}

6. Page fermée → localStorage gardé
   
7. Page réouverte → Signal lit localStorage à nouveau
   → Utilisateur reste connecté!
```

---

## ⚙️ Exemple: Logout

```typescript
logout(): void {
  localStorage.removeItem('gpc_token');  // Efface disque
  this.token.set(null);                   // Met à jour Signal
  this.currentUser.set(null);             // Met à jour Signal
}
```

**Immédiatement:**
- Templates rerender
- Navigation change (logout button disparaît)
- Prochaines requêtes n'ont pas de token
- 401 handled par interceptor

---

## 🎯 Résumé

**Signal = Comment l'app SAIT que token existe**
**localStorage = Où token PERSISTE**

Les deux travaillent ensemble:
- **localStorage** = Disque (persistent)
- **Signal** = RAM (réactif)

Signal lit localStorage au démarrage, puis reste en sync.
