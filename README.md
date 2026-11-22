# Driver Registration / تسجيل السائقين

A bilingual (Arabic / English) registration link for taxi and delivery drivers.
The driver opens the link, fills in the form, and the record lands in your Firebase database.

| File | What it is |
|---|---|
| `index.html` | The form you send to drivers |
| `admin.html` | Your private page: list, review, approve/reject, export CSV |
| `firebase-config.js` | Your Firebase project keys (you edit this) |
| `firestore.rules` | Database security rules (you paste this into Firebase) |
| `styles.css` | Shared styling |

## Setup (about 15 minutes, free plan, no credit card)

### 1. Create the Firebase project
1. Go to <https://console.firebase.google.com> → **Add project**.
2. **Build → Firestore Database → Create database** → *Production mode*.
3. Open the **Rules** tab, replace everything with the contents of `firestore.rules`, and **Publish**.

### 2. Create your admin login
1. **Build → Authentication → Get started → Email/Password → Enable**.
2. **Users → Add user**: your email and a strong password. Copy the **User UID** shown in the list.
3. **Firestore Database → Start collection** named `admins` → **Document ID = that UID** → add any field (e.g. `role` = `admin`).

Repeat steps 2–3 for each person who should see the applications.

### 3. Connect the form
1. **Project settings (gear icon) → Your apps → Web (`</>`)** → register an app.
2. Copy the values shown into `firebase-config.js`.

### 4. Publish on GitHub Pages
1. Create a new **public** repository on GitHub and upload all five files plus this README.
2. **Settings → Pages → Source: Deploy from a branch → `main` / `(root)` → Save**.
3. After a minute your links are:
   - Driver form: `https://YOUR-USER.github.io/YOUR-REPO/` (add `?lang=en` to open in English)
   - Admin page: `https://YOUR-USER.github.io/YOUR-REPO/admin.html`
4. In Firebase: **Authentication → Settings → Authorized domains → Add** `YOUR-USER.github.io`.

Send one test application yourself and check that it appears in the admin page.

## How the data is stored (for linking to your app)

```
drivers/{driverId}
  firstName, lastName, fatherName      string
  birthDate                            "YYYY-MM-DD"
  birthPlace, currentAddress           string
  phone                                digits, optional leading +
  email                                string (may be empty)
  serviceType                          "taxi" | "delivery" | "both"
  status                               "new" | "approved" | "rejected"
  lang                                 "ar" | "en"
  createdAt                            timestamp

drivers/{driverId}/files/idPhoto | licensePhoto | selfie
  data                                 JPEG as a data URL (base64)
  createdAt                            timestamp
```

Your mobile app can read approved drivers with any Firebase SDK (Android, iOS, Flutter, web),
for example by querying `drivers` where `status == "approved"`. Add a rule for the app's own
access when you get to that step; at the moment only admins can read.

## Things to know

- **Photos are stored in Firestore, not Firebase Storage.** Firebase Storage now requires a
  billing account. The form shrinks each photo in the browser (about 100–500 KB) so it fits in
  a database document. The free plan holds 1 GB, roughly 1,000–2,000 drivers. Past that, move
  to the Blaze plan and Storage.
- **The keys in `firebase-config.js` are meant to be public.** Protection comes from
  `firestore.rules`: the public can only *create* an application, never read one.
- **The form is open to anyone with the link.** To block bots and spam submissions, turn on
  **App Check** with reCAPTCHA in the Firebase console.
- **You are holding ID documents.** Keep the admin list short, use strong passwords, and delete
  rejected applications you no longer need.
