# Oppsett av brukere med passord (Firebase)

Hver person kan lage en bruker med brukernavn og passord. Stjernemerkede flashcards, «kan»-status og egne flashcards lagres på brukeren og kommer tilbake når man logger inn på en annen enhet. Uten innlogging lagres alt bare i nettleseren.

Brukernavnet gjøres internt om til en e-postadresse (`brukernavn@brukere.syg1000-eksamenstrening.app`) fordi Firebase krever det. Det sendes aldri e-post, og passordet kan derfor ikke tilbakestilles av brukeren selv. Du kan slette en bruker under **Authentication → Users**, så kan personen lage den på nytt.

## Engangsoppsett i Firebase-konsollen

Prosjektet `syg1000-eksamenstrening` finnes allerede. Gå til https://console.firebase.google.com og åpne det.

1. **Build → Authentication → Get started → Sign-in method → Email/Password → Enable → Save.** (Bare den første bryteren, ikke «Email link».)
2. **Authentication → Settings → Authorized domains:** sjekk at `josefinereichgjer.github.io` står i listen. Legg den til hvis ikke.
3. **Firestore Database → Rules:** erstatt alt med dette og trykk **Publish**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       // Nye brukere med passord: bare eieren kan lese og skrive sitt eget dokument.
       match /accounts/{uid} {
         allow read: if request.auth != null && request.auth.uid == uid;
         allow create, update: if request.auth != null && request.auth.uid == uid
           && request.resource.data.keys().hasOnly(['name', 'known', 'stars', 'custom', 'updated'])
           && request.resource.data.known is list && request.resource.data.known.size() <= 3000
           && request.resource.data.stars is list && request.resource.data.stars.size() <= 3000
           && request.resource.data.custom is list && request.resource.data.custom.size() <= 500;
         allow delete: if false;
       }
       // Bilder på flashcards: bare eieren kan lese, lagre og slette sine egne bilder.
       match /accounts/{uid}/images/{img} {
         allow read, delete: if request.auth != null && request.auth.uid == uid;
         allow create, update: if request.auth != null && request.auth.uid == uid
           && request.resource.data.keys().hasOnly(['data', 'updated'])
           && request.resource.data.data is string
           && request.resource.data.data.size() < 400000;
       }
       // Gamle brukernavn uten passord: kan bare leses, slik at fremgangen kan flyttes over.
       match /users/{name} {
         allow get: if name.matches('[a-z0-9_-]{3,30}');
         allow list, write: if false;
       }
     }
   }
   ```

## Flytting fra gamle brukernavn

Den som hadde et brukernavn uten passord, velger **Lag bruker** med samme navn. Da hentes stjernene og «kan»-statusen fra det gamle navnet automatisk.

## Sikkerhet

`apiKey` i en nettside-app er ikke hemmelig. Det er reglene over som beskytter dataene: hver bruker kan bare se og endre sitt eget dokument. Ikke lagre sensitive opplysninger i egne kort.
