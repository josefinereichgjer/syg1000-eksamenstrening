# Oppsett av brukernavn (Firebase)

Siden lar hver person velge et brukernavn, uten passord og uten Claude-konto. Stjernemerkede flashcards og «kan»-status lagres på brukernavnet, og kommer tilbake når samme navn skrives inn på en annen enhet.
Funksjonen er avslått til du legger inn en Firebase-konfigurasjon. Uten den lagres alt bare i nettleseren.

1. Gå til https://console.firebase.google.com og velg **Add project**. Gi prosjektet et navn, og Google Analytics kan slås av.
2. **Build → Firestore Database → Create database.** Velg en region i Europa og start i **production mode**.
3. Under **Rules** i Firestore limer du inn dette og trykker **Publish**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{name} {
         allow get: if name.matches('[a-z0-9_-]{3,30}');
         allow list, delete: if false;
         allow create, update: if name.matches('[a-z0-9_-]{3,30}')
           && request.resource.data.keys().hasOnly(['known', 'stars', 'updated'])
           && request.resource.data.known is list
           && request.resource.data.stars is list
           && request.resource.data.known.size() <= 3000
           && request.resource.data.stars.size() <= 3000;
       }
     }
   }
   ```

   Reglene gjør at ingen kan liste opp eller slette brukernavn, og at bare disse tre feltene kan skrives, med en størrelsesgrense.

4. **Project settings (tannhjulet) → General → Your apps → Web (`</>`)**. Registrer en app og kopier verdiene i `firebaseConfig`.
5. Åpne `index.html`, finn `const FIREBASE_CONFIG=` og fyll inn `apiKey`, `projectId` og `appId`. Lagre og push til GitHub.

## Sikkerhet

Det er ikke noe passord, så alle som kjenner et brukernavn kan se og endre den personens stjerner. Det er greit for flashcards, men ikke lagre noe sensitivt. Be folk velge et navn som ikke er lett å gjette, og ikke bruke fullt navn.
Hvis to personer velger samme navn, slås stjernene deres sammen.
`apiKey` i en nettside-app er ikke hemmelig. Det er reglene i punkt 3 som beskytter dataene.
