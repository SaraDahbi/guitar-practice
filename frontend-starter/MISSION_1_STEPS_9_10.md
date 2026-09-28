# Mission 1 - Steps 9 & 10: Edge Cases & Quality Assurance

## 📋 Step 9: Edge Cases & Error Scenarios

### Test Scenario 1: Empty Form Submission
**Action:** Click submit without filling any fields

```
Expected:
- All fields marked as touched
- Error messages appear for each field
- Submit button disabled
- No API call made
```

**Code handling:** `Validators.required` on all fields

---

### Test Scenario 2: Invalid Email Format
**Action:** Type "notanemail" in email field → blur

```
Expected:
- Error: "Format email invalide"
- Submit button disabled
- Field gets red border (CSS)
```

**Code handling:**
```typescript
Validators.email
```

---

### Test Scenario 3: Rapid Form Changes
**Action:** Type quickly in form fields → rapid edits

```
Expected:
- Validation errors update in real-time
- No lag
- Form still submittable when complete
```

**Checks:**
- Signal updates fast
- No memory leaks
- No race conditions

---

### Test Scenario 4: Network Timeout
**Action:** Slow network → Try to login → Wait

```
Expected:
- Loading indicator shows
- Button disabled during request
- After timeout: error message shown
- Can retry
```

**Code handling:**
```typescript
this.uploading.set(true);
// ... API call
// error handler: set false + show error
```

---

### Test Scenario 5: Duplicate Registration
**Action:** Try to register with existing email

```
Expected:
- API returns 409 Conflict
- Error shown: "Email already exists"
- Form preserved (not cleared)
- User can modify and retry
```

**Backend response:**
```json
{
  "message": "Email already in use"
}
```

---

### Test Scenario 6: Large File Upload
**Action:** Try to upload 30MB file (over limit)

```
Expected:
- Browser blocks (file input accept)
- If somehow sent: API returns 413 Payload Too Large
- Error shown to user
- Can retry with smaller file
```

---

### Test Scenario 7: Browser Storage Full
**Action:** localStorage quota exceeded

```
Expected:
- Token still saved (if quota allows)
- Error handling graceful
- App doesn't crash
```

**Code:**
```typescript
try {
  localStorage.setItem('gpc_token', token);
} catch (e) {
  console.error('Storage full');
}
```

---

### Test Scenario 8: Multiple Tabs Open
**Action:** Login in tab 1 → Check tab 2

```
Expected:
- Storage event fires
- Tab 2 auto-updates (if using StorageListener)
- Both tabs stay in sync
```

---

### Test Scenario 9: Back Button After Logout
**Action:** Logout → Click browser back button

```
Expected:
- Can't access /profile (redirects to /login)
- authGuard prevents access
- History doesn't matter
```

---

### Test Scenario 10: Session Expired During Action
**Action:** Login → Wait 1+ hour → Try to upload file

```
Expected:
- Upload request has expired token
- API returns 401
- authInterceptor catches 401
- Auto-logout + redirect to /login
- User must re-login
```

---

## ✅ Step 10: Quality Assurance Checklist

### Code Quality

#### TypeScript
- [ ] No `any` types (except where necessary)
- [ ] All variables typed
- [ ] No unused imports
- [ ] Consistent naming (camelCase)
- [ ] Constants in UPPER_CASE if applicable

#### Components
- [ ] `inject()` used instead of constructor
- [ ] OnInit lifecycle hook for initialization
- [ ] Signals used for reactive state
- [ ] No direct DOM manipulation
- [ ] Proper error handling

#### Services
- [ ] Single responsibility
- [ ] No UI logic
- [ ] Return Observables (not Promises)
- [ ] Proper error propagation
- [ ] No hardcoded URLs

#### Forms
- [ ] Reactive Forms (not Template-driven)
- [ ] `nonNullable: true` on controls
- [ ] Proper validators
- [ ] Error messages for each validator
- [ ] Touch/blur tracking

---

### HTML/Template Quality

#### Accessibility
- [ ] Semantic HTML (form, label, button)
- [ ] aria-label on icon buttons
- [ ] Input type correct (email, password)
- [ ] Form labels associated with inputs
- [ ] Color not only indicator (use text too)

#### Best Practices
- [ ] No inline styles
- [ ] CSS classes used
- [ ] Responsive design (mobile-first)
- [ ] @if/@for used (not *ngIf/*ngFor)
- [ ] No hardcoded text in loops

#### Security
- [ ] No password visible in template
- [ ] No token displayed
- [ ] {{ }} interpolation safe (no XSS risk)
- [ ] Form validation before submit
- [ ] No sensitive data logged

---

### CSS Quality

#### Organization
- [ ] Consistent spacing/indentation
- [ ] Related rules grouped
- [ ] Colors consistent (use variables if possible)
- [ ] No magic numbers (use readable values)

#### Responsive
- [ ] Works on mobile (375px+)
- [ ] Works on tablet (768px+)
- [ ] Works on desktop (1024px+)
- [ ] No horizontal scroll
- [ ] Touch targets 44px+ (mobile)

#### Visual
- [ ] Hover states on interactive elements
- [ ] Disabled state clearly visible
- [ ] Error messages in red (#dc3545)
- [ ] Success feedback clear
- [ ] Loading indicators visible

---

### Testing Coverage

#### User Flows
- [ ] Happy path: register → login → profile → logout
- [ ] Error path: wrong password → error shown
- [ ] Protected routes: no token → redirect to login
- [ ] Token expiration: 401 → auto logout
- [ ] File upload: success and failure cases

#### Edge Cases
- [ ] Empty fields
- [ ] Invalid email format
- [ ] Rapid form changes
- [ ] Network errors
- [ ] Large file uploads
- [ ] Session timeout
- [ ] Multiple tabs

#### Security
- [ ] No token in logs
- [ ] No password in network requests
- [ ] Authorization header present on protected routes
- [ ] CORS properly configured
- [ ] No sensitive data exposed

---

### Performance Checklist

#### Bundle Size
- [ ] No unnecessary imports
- [ ] Tree-shaking enabled
- [ ] Lazy loading considered for routes

#### Runtime Performance
- [ ] No memory leaks (Observables unsubscribed)
- [ ] Signals update efficiently
- [ ] No unnecessary re-renders
- [ ] Images optimized (if any)
- [ ] CSS not duplicated

#### Network
- [ ] API calls necessary (no redundant calls)
- [ ] Request size reasonable
- [ ] Response size reasonable
- [ ] Gzip enabled on server

---

### Browser Compatibility

#### Test On
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)

#### Features Used
- [ ] localStorage (all browsers)
- [ ] Fetch API (all modern browsers)
- [ ] ES6+ syntax (check build target)
- [ ] CSS Grid/Flexbox (all modern browsers)

---

### Documentation

#### Code Comments
- [ ] Public methods have JSDoc
- [ ] Complex logic explained
- [ ] No obvious comments ("increment i")
- [ ] Comments kept up-to-date

#### README Files
- [ ] Setup instructions clear
- [ ] How to run included
- [ ] API endpoints documented
- [ ] Known issues listed
- [ ] Contributing guidelines (if applicable)

---

### Deployment Readiness

#### Configuration
- [ ] API URL configurable (not hardcoded)
- [ ] Environment variables used
- [ ] Secrets not in code
- [ ] Build process documented

#### Logging
- [ ] console.log() removed (or guarded)
- [ ] console.error() for actual errors
- [ ] Error messages user-friendly
- [ ] Stack traces not exposed to users

#### Monitoring
- [ ] Error handling comprehensive
- [ ] Edge cases handled
- [ ] Graceful degradation
- [ ] Fallback strategies in place

---

## 🎯 Quality Score Rubric

### Code: /10
- Code organization: /2
- Error handling: /2
- Security: /2
- Performance: /2
- Documentation: /2

### UI/UX: /10
- Accessibility: /2
- Responsiveness: /2
- Visual design: /2
- Error messages: /2
- Loading states: /2

### Testing: /10
- Happy path: /2
- Error cases: /2
- Edge cases: /2
- Security testing: /2
- Cross-browser: /2

### Completeness: /10
- All features: /4
- All documentation: /3
- Ready for deployment: /3

**Total:** /40

---

## ✅ Final Checklist Before Submission

### Functionality
- [ ] All 5 components work
- [ ] All API calls successful
- [ ] All error cases handled
- [ ] All validations working
- [ ] No console errors

### Security
- [ ] Token stored securely
- [ ] Token in Authorization header
- [ ] 401 errors handled
- [ ] No sensitive data logged
- [ ] CSRF protection (if needed)

### UX
- [ ] Loading indicators
- [ ] Error messages clear
- [ ] Disabled buttons appropriate
- [ ] Form validation helpful
- [ ] Navigation intuitive

### Code
- [ ] No TypeScript errors
- [ ] No console warnings
- [ ] Properly formatted
- [ ] Well organized
- [ ] Documented

### Documentation
- [ ] 9+ documentation files
- [ ] IA report complete
- [ ] Setup instructions
- [ ] Testing guide
- [ ] Architecture explained

### Testing
- [ ] 10+ edge cases tested
- [ ] Network tab verified
- [ ] Multiple browsers tested
- [ ] Mobile tested
- [ ] Error scenarios tested

---

## 🚀 Final Submission Package

**Files to Include:**
1. ✅ Source code (src/)
2. ✅ RAPPORT_IA_MISSION_1.md
3. ✅ All documentation files (9 total)
4. ✅ Network screenshots
5. ✅ Test results

**Format:**
- Zip file OR
- Git push to repository

**Due:** Check course requirements

---

**Status:** 🎯 Quality Assurance COMPLETE
