

---

## Avancerad SQL och Normalisering

### a) Komplex databasstruktur
För en komplex databasstruktur kan vi tänka oss ett bibliotekssystem som innehåller följande fyra tabeller: **Böcker**, **Författare**, **Lån**, och **Medlemmar**.

- **Böcker**: innehåller information om varje bok som titel, ISBN, och publiceringsår.
- **Författare**: innehåller författarens ID, namn, och nationalitet.
- **Lån**: registrerar alla lån i systemet med kolumner för Låne-ID, Bok-ID (främmande nyckel till Böcker), Medlems-ID (främmande nyckel till Medlemmar), och lånedatum.
- **Medlemmar**: innehåller medlemsinformation som Medlems-ID, namn, och kontaktinformation.

### b) Avancerad SQL-fråga med JOINs och sub-fråga
Här är en exempel på en SQL-fråga som hämtar information om böcker som lånas oftast av medlemmar i en viss stad, inklusive författarens namn:

```sql
SELECT b.titel, f.namn AS författare, COUNT(l.låne_id) AS antal_lån
FROM Böcker b
JOIN Författare f ON b.författare_id = f.författare_id
JOIN Lån l ON b.bok_id = l.bok_id
JOIN Medlemmar m ON l.medlems_id = m.medlems_id
WHERE m.stad = 'Stockholm'
GROUP BY b.titel, f.namn
HAVING COUNT(l.låne_id) > 5
ORDER BY antal_lån DESC;
```

#### Förklaring:
1. **JOINs**: Frågan använder tre JOINs för att koppla tabellerna **Böcker**, **Författare**, **Lån**, och **Medlemmar**. Denna struktur är nödvändig för att hämta relevanta data från varje tabell och skapa en helhetsbild av böcker och författare som lånas ofta av medlemmar från en viss stad.
2. **WHERE och GROUP BY**: WHERE-filtret specificerar medlemmar från Stockholm, och GROUP BY grupperar resultat efter titel och författare, vilket gör att vi kan beräkna antal lån per bok.
3. **HAVING och ORDER BY**: HAVING-klausulen filtrerar bort böcker med färre än fem lån, och ORDER BY ordnar resultatet efter antal lån i fallande ordning.

### c) Normalisering och analys
Strukturen är designad för att uppfylla **3NF** (tredje normalformen):
1. **1NF**: Alla attribut är atomära, och varje kolumn innehåller endast ett värde per rad.
2. **2NF**: Alla icke-primära attribut är fullt funktionellt beroende av hela primärnyckeln. Exempelvis har tabellen **Lån** en primärnyckel baserad på Låne-ID, och inga icke-primära attribut är beroende av delmängder av primärnyckeln.
3. **3NF**: Alla icke-primära attribut är direkt beroende av primärnyckeln och inte på andra icke-primära attribut. Detta förhindrar redundans och säkerställer att data är organiserade på ett logiskt sätt.

---

## Databasdesign och Alternativa Lösningar

### a) Förslag på förbättringar
1. **Normalisering av Medlemmar och deras kontaktinformation**: För att reducera redundans kan vi flytta kontaktinformation till en egen tabell, **MedlemsKontakt**. Detta gör det möjligt att ha fler typer av kontaktuppgifter (t.ex. telefon, e-post) utan att duplicera data.
   
2. **Införa Index**: Genom att indexera kolumner som ofta används i WHERE-klausuler (exempelvis stad i Medlemmar och författare_id i Böcker) kan vi förbättra prestandan i sökfrågor, särskilt om databasen växer.

### b) Alternativ databasstruktur
Ett alternativt sätt att strukturera systemet är att kombinera **Böcker** och **Författare** till en **BokInfo**-tabell som innehåller författarinformation direkt. 

#### Jämförelse:
- **Fördelar**: Denna struktur kan minska antalet JOINs som behövs och därmed göra vissa frågor mer effektiva.
- **Nackdelar**: Det introducerar risk för redundans, särskilt om samma författare har flera böcker. Vid uppdatering av författarens information, kan vi behöva uppdatera flera rader i stället för en central rad.

---

## Jämförelse mellan Relationsdatabaser och Dokumentdatabaser

### a) Struktur i MongoDB
Om vi använder MongoDB, kan strukturen representeras som en samling dokument, där varje bokdokument innehåller en författarsektion och en lånehistorik. Till exempel:

```json
{
  "titel": "Exempelbok",
  "ISBN": "123-456-789",
  "författare": {
    "namn": "Författar Namn",
    "nationalitet": "Svensk"
  },
  "lån": [
    {
      "medlems_id": "1",
      "lånedatum": "2024-01-15"
    },
    {
      "medlems_id": "2",
      "lånedatum": "2024-02-20"
    }
  ]
}
```

### b) Fördelar och nackdelar
- **Fördelar**: Dokumentdatabaser som MongoDB hanterar nestade data bra och är mer flexibla vid schemaändringar. Detta kan underlätta om strukturen ändras ofta eller om vi har många olika typer av lån- och medlemsdata.
- **Nackdelar**: Att hantera stora mängder referenser eller flera beroenden kan bli ineffektivt i en dokumentdatabas eftersom det är svårare att uppdatera eller säkerställa data-integritet jämfört med en relationsdatabas.

---

## Reflektion och Argumentation

### a) Reflektion över designval
Jag fokuserade på att designa en databasstruktur som maximerar både normalisering och flexibilitet. Genom att använda en strukturerad modell i SQL uppfyller vi kraven för dataintegritet och möjliggör komplexa frågor utan att behöva duplicera data.

### b) Argument för vald design
Denna relationsdatabasdesign är optimal för ett bibliotekssystem där vi behöver spåra data på ett strukturerat sätt och samtidigt säkerställa att data är konsekvent. Potentiella konsekvenser av designen inkluderar att JOIN-operationer kan bli resurskrävande om databasen växer, men detta kan hanteras genom indexering och optimeringstekniker.

--- 

