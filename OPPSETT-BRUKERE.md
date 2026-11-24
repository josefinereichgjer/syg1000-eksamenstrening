# Oppsett av brukere (Firebase)

Siden har innlogging med e-post og passord, slik at hver bruker kan lagre stjernemerkede flashcards og «kan»-status.
Den er avslått til du legger inn en Firebase-konfigurasjon. Uten den lagres alt bare i nettleseren.

1. Gå til https://console.firebase.google.com og velg **Add project**. Gi prosjektet et navn, og Google Analytics kan slås av.
2. **Build → Authentication → Get started → Sign-in method → Email/Password → Enable → Save.**
3. **Build → Firestore Database → Create database.** Velg en region i Europa og start i **production mode**.
4. Under **Rules** i Firestore limer du inn dette, og trykker **Publish**. Hver bruker kan da bare lese og skrive sitt eget dokument:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{uid} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

5. **Authentication → Settings → Authorized domains → Add domain:** `josefinereichgjer.github.io`.
6. **Project settings (tannhjulet) → General → Your apps → Web (`</>`)**. Registrer en app og kopier verdiene i `firebaseConfig`.
7. Åpne `index.html`, finn `const FIREBASE_CONFIG=` og fyll inn `apiKey`, `authDomain`, `projectId` og `appId`. Lagre og push til GitHub.

`apiKey` i en nettside-app er ikke hemmelig. Det er reglene i punkt 4 som beskytter dataene.
