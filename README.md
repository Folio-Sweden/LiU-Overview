# Linköpings universitetsbibliotek (LiU)

## Allmänt

Linköpings universtetsbibliotek gick live med Folio i juli 2023. Vår egenutvecklade kod kan grovt delas in i tre delar: 
- En allmän folioklient i Python som sköter inloggning, sessionshantering, utloggning och HTTP-anrop mot Folio. Denna distribueras via PyPi och ligger publikt på GitHub
- En uppsättning python-skript som körs scehmalagt. Ligger i lokalt GitLab-repo.
- En webbserver i JavaScript som framförallt ger låntagare ett gränssnitt för att hantera lån och reservationer. Bibliotekspersonal kommer även via denna klient åt plocklistor och andra speciallistor. Dessutom utgör den ett API för EBSCO Discovery Service för att kunna göra RTAC-uppslag. Ligger i lokalt GitLab-repo.

**Status 2025-05-08:** Python-skripten håller på att arbetas om för att använda pyfolioclient genomgående. Mina Lån behöver arbetas om för att stödja tokens med begränsad livstid.

## pyfolioclient

I och med att Folio Folio började över till Refresh Token Rotation (RTR), alltså tokens som har en begränsad livstid, behövde en kärnkomponent i vår python-kod arbetas om. I stället för att vara inbakad i vårt lokala skript-paket lyftes denna del ut i ett eget python-paket och distribueras nu via PyPi. Denna ligger för närvarande på https://github.com/balljok/pyfolioclient.

## Folio-skript

De python-skript som utvecklats på LiU ligger idag (2025-05-08) samlade i ett stort paket, lokalt på GitLab. Förutom att göra den uppgift de ska göra så loggas även resultatet mot en lokal övervakningstjänst. 

**Status 2025-05-08:** Alla skript håller på att arbetas om för att använda pyfolioclient. Många skript brister i sakna retry-strategier mot Folio, Libris och Libris Fjärrlån.

### Libris-import

Skript som exporterar poster från Libris till MARC, som sedan läses in i Folio. 

### Automatiska omlån

Skript som automatiskt försöker utföra omlån på lån 3 dagar innan lånet går ut. Enligt våra regler går en påminnelse ut till låntagaren 2 dagar innan lånet går ut, därav att lånet försöker lånas om dagen inann påminnelsen går/hade gått ut.

### Lägg till uppställningsord (callNumberSuffix)

### Loggning av inloggningsnycklar till publika datorer

Lån anonymiseras relativt fort i vår instans av Folio, men för just inloggningsnycklar till våra publika datorer vill vi spara lånehistoriken lite längre. En enkel lösning för detta har utvecklats så att UUID för den som lånat en inloggningsnyckel sparas i sex månader i en JSON-fil på den server som skripten körs ifrån. Skulle vi upptäcka att något oegentligt gjorts från en publik dator har vi en viss möjlighet att koppla det till en låntagare. 

### Epost-notifieringar

Email-notifications + overdue ref

### Fjärrlån

### Registervård

### Användarhantering

Sesam integration + deaktivera konton + remove users

### Extern loggning av låneinfo för statistikunderlag

### Hantera mapping rules

## Webbtjänsten Mina Lån

Tjänsten Mina Lån är lite missvisande då det är ett kodpaket som innefattar även annat som inte är ett gränssnitt för att låntagare ska kunna hantera sina lån och reservationer. Internt är det ett paket i JavaScript, hanterat via lokalt GitLab-repo, som heter "Foliofront Node Root". Det använder Node, Express, Passport för SAML och Embedded JavaScript Templated (EJS) för själva webbsidorna.

**Status 2025-05-08:** Behöver arbetas om för att stödja RTR. Koden behöver en större översyn. Den gör sitt jobb, men det är tydligt att strukturen, felhanteringen och att den allämnna kvaliteten på koden har utrymme för förbättring. 

### Mina Lån - lån och reservationer

Den del som möter en normal låntagare (https://services.bibl.liu.se). Här kan man som student eller anställd logga in med sitt LiU-konto. Som annan låntagare kan man ansöka om lånekort och logga in med PIN-kod. Man möts av en mobilanpassad sida med två flikar; en för aktiva lån och en för de reservationer man lagt.

### Mina Lån - Listor för personal

När man loggar in som personal har man tillgång till en tredje flik som heter Listor. För att kunna se listorna behöver man förutom att vara personal ha rätt tagg på sin användare i Folio. Detta är vårt sätt att lösa det så att bara rätt användare får se listor. Huvudlistorna är:

- Plocklista för reserverade böcker
- Lista över böcker som ska rensas från reservationshyllan

Dessutom finns listor för:

- Försenade kursreferensböcker
- Böcker som är under transport mellan våra campus (4st)

För den personal som jobbar med fjärrlån finns även listor för:

- 

### RTAC

### Fjärrlån

