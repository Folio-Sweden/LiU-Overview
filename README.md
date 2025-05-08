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

De python-skript som utvecklats på LiU ligger idag (2025-05-08) samlade i ett stort paket, lokalt på GitLab. Förutom att göra den uppgift de ska göra så loggas även resultatet mot en lokal övervakningstjänst. En del av skripten kan verka överflödiga då det finns motsvarande funktionalitet i Folio, men då har sannolikt verksamheten haft lite speciella krav som inte gått att få till med den inbyggda funktionaliteten.

**Status 2025-05-08:** Alla skript håller på att arbetas om för att använda pyfolioclient. Många skript brister i sakna retry-strategier mot Folio, Libris och Libris Fjärrlån.

### Libris-import

Skript som exporterar poster från Libris till MARC, som sedan läses in i Folio. Utgår från en tidsstämpel för när skriptet kördes senast och hämtar poster från denna tid till den tid då skriptet började köras. Det hämtar posterna, tar bort eventuella dubletter, gör vissa mindre justeringar av posterna, delar upp dem i mindre paket och laddar upp i Folio. Schemalagt att köras varannan minut.

### Automatiska omlån

Skript som automatiskt försöker utföra omlån på lån 3 dagar innan lånet går ut. Enligt våra regler går en påminnelse ut till låntagaren 2 dagar innan lånet går ut, därav att lånet försöker lånas om dagen inann påminnelsen går/hade gått ut.

### Lägg till uppställningsord (callNumberSuffix)

Skript som lägger till information i callNumberSuffix för valda locations i Folio om detta saknas.  

### Loggning av inloggningsnycklar till publika datorer

Lån anonymiseras relativt fort i vår instans av Folio, men för just inloggningsnycklar till våra publika datorer vill vi spara lånehistoriken lite längre. En enkel lösning för detta har utvecklats så att UUID för den som lånat en inloggningsnyckel sparas i sex månader i en JSON-fil på den server som skripten körs ifrån. Skulle vi upptäcka att något oegentligt gjorts från en publik dator har vi en viss möjlighet att koppla det till en låntagare. 

### Epost-notifieringar

- Meddela låntagare som inte är studenter eller anställda när deras konto håller på att gå ut och de behöver godkänna låneregler igen
- Sammanställning till låntagare med aktiva lån över deras lån
- Sammanställning till fjärrlånande bibliotek med aktiva lån
- Sammanställning av reservationer som inte kommer att kunna uppfyllas (ungefär att det inte finns något exemplar som inte är förlorat)
- Personalnotifieringar när verk blir tillgängliga som de lagt en kommentar på med sitt LiU-id. För att bokvården eller katalogisatörer ska kunna fånga upp verk som ska lagas eller hanteras på något sätt.

### Fjärrlån

### Registervård

Enklare registervård vars behov vuxit fram sedan vi gick live:

- Ändra callNumberTypeId för holding baserat på dess location
- Ändra lånetyp baserat på location.
- Ta bort icke-godkända taggar i Folio (taggar som skapas är synliga för alla användare och ju fler desto svårare att hitta rätt)
- Kolla om nyrad smugit sig in i callNumber (vår återlämningsmaskin kan inte sortera verket rätt om detta finns)

### Användarhantering

Alla som får ett s.k. LiU-id (studenter och personal) skapas upp som låntagare i Folio automatiskt. Det centrala systemet som hanterar användare droppar filer på en FTP som vi bevakar. Utifrån detta skapar vi nya användare, uppdaterar dem och deaktiverar dem i Folio. Lite olika skript sköter olika delar av denna process.

### Extern loggning av låneinfo för statistikunderlag

Lån anonymiseras relativt fort i vår instans av Folio vilket gör det svårt med årlig statistik till KB om man inte också lagrar delar av låneinformationen utanför Folio. Vi har en SQL-databas som löpande uppdateras med nya lån och som har koll på antalet omlån kopplade till lånet. Vi loggar här: Tid för lånet, låntagarkategorin, vart utlånet skedde, kön på personen som gjorde lånet utifrån personnummer, ålder på låntagaren då lånet gjordes, vilket typ av material som lånades, om materialet är s.k. kursbok, antalet omlån, hyllsignum för verket, krypterad och trasig info om vem som gjorde lånet (för att kunna få fram antal unika låntagare), samt lite fjärrlånespecifik information.

### Hantera mapping rules

Mapping rules i Folio är en fil som beskriver hur MARC-information ska mappas mot interna Folio-fält. Vi har två egna regler. Processen vi har när Folio uppdateras är:

1. Återställ mapping rules till den releasens default-regler. Detta för att få med eventuella ändringar som skett i releasen.
2. Ladda ner de återställda reglerna.
3. Införa våra ändringar.
4. Ladda upp de uppdaterade reglerna till Folio.

Detta skript stöttar denna process.

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

- Flera fjärrlånebeställningar av samma verk samma dag
- Fjärrlånebeställningar som är under transport
- Fjärrlånebibliotek vars konto håller på att gå ut i Folio
- Böcker vi lånat ut till andra bibliotek som inte kommit tillbaka
- Böcker vi lånat in från andra bibliotek som inte kommit tillbaka

### RTAC

### Fjärrlån

