# Linköpings universitetsbibliotek (LiU)

## Allmänt

Linköpings universtetsbibliotek gick live med Folio i juli 2023. Vår egenutvecklade kod kan grovt delas in i tre delar: 
- En allmän folioklient i Python som sköter inloggning, sessionshantering, utloggning och HTTP-anrop mot Folio. Denna distribueras via PyPi och ligger publikt på GitHub
- En uppsättning python-skript som körs scehmalagt. Ligger i lokalt GitLab-repo.
- En webbserver i JavaScript som framförallt ger låntagare ett gränssnitt för att hantera lån och reservationer. Bibliotekspersonal kommer även via denna klient åt plocklistor och andra speciallistor. Dessutom utgör den ett API för EBSCO Discovery Service för att kunna göra RTAC-uppslag. Ligger i lokalt GitLab-repo.

Status 2025-05-08: Python-skripten håller på att arbetas om för att använda pyfolioclient genomgående. Mina Lån behöver arbetas om för att stödja tokens med begränsad livstid.

## pyfolioclient

I och med att Folio Folio började över till Refresh Token Rotation (RTR), alltså tokens som har en begränsad livstid, behövde en kärnkomponent i vår python-kod arbetas om. I stället för att vara inbakad i vårt lokala skript-paket lyftes denna del ut i ett eget python-paket och distribueras nu via PyPi. Denna ligger för närvarande på https://github.com/balljok/pyfolioclient.

## Folio-skript

De python-skript som utvecklats på LiU ligger idag (2025-05-08) samlade i ett stort paket, lokalt på GitLab. Förutom att göra den uppgift de ska göra så loggas även resultatet mot en lokal övervakningstjänst. 

### Libris-import

### Automatiska omlån

### Lägg till uppställningsord

### Loggning av inloggningsnycklar till publika datorer

### Epost-notifieringar

Email-notifications + overdue ref

### Fjärrlån

### Registervård

### Användarhantering

Sesam integration + deaktivera konton + remove users

### Extern loggning av låneinfo för statistikunderlag

### Hantera mapping rules

## Webbtjänsten Mina Lån

### Mina Lån - lån och reservationer

### Mina Lån - Listor för personal

### RTAC

### Fjärrlån

