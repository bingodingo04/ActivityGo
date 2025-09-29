# Case: ActiviGo – bokningsplattform för aktiviteter

Kommunen vill lansera **ActiviGo**, en tjänst där invånare kan boka allt från **inomhusaktiviteter** (t.ex. klättervägg, pingis) till **utomhusaktiviteter** (t.ex. padelbanor utomhus, utegym‑pass).

- För **utomhusaktiviteter** ska bokningsflödet visa **väderprognos** för tidsintervallet och platsen.
- Administratörer skapar aktiviteter, platser och tider, sätter kapacitet och regler.
- Användare kan söka, filtrera, boka och avboka pass.

---

# Vad ni ska göra

<aside>
**Funktionalitet**

Mål: Bygg ett robust backend‑API med tydlig lagerindelning och en React‑frontend som levererar ett sammanhållet användarflöde. Integrera väderdata via extern webbtjänst.

## Funktionalitet (API)

1. **Autentisering & Auktorisering**
    - Registrera användare och logga in (JWT Bearer).
    - Roller: **User** och **Admin**.
    - Förnya token (refresh‑token eller re‑inlogg) (Extrauppgift)
2. **Aktiviteter & Platser**
    - CRUD för **Aktiviteter** (namn, beskrivning, kategori, inomhus/utomhus, standardlängd, pris, bild‑URL).
    - CRUD för **Aktivitetsplatser** (namn, adress, lat/long, inomhus/utomhus, kapacitet/banor).
    - Koppla aktiviteter till plats(er).
3. **Tillfällen & Bokning**
    - Skapa **ActivityOccurrences** (starttid, sluttid, plats, kapacitet).
    - **Boka** (en plats/slot) samt **Avboka** före cutoff.
    - Visa mina bokningar, status (kommande, passerade, avbokade).
    - **Dubbelbokningsskydd**: användare kan inte dubbelboka överlappande tider.
    - **Kapacitetskontroll**: boka endast om lediga platser finns.
4. **Sök & filtrering**
    - Filtrera på datumintervall, kategori, inomhus/utomhus, plats, lediga platser.
5. **Administrationsfunktioner (endast Admin)**
    - CRUD för aktiviteter, platser, tillfällen och kategorier.
    - Inaktivera/aktivera aktiviteter/platser.
    - Enkel **statistik**: antal bokningar per aktivitet/plats i valfritt tidsintervall
</aside>

<aside>
**Tekniska krav**

## Funktionalitet (Frontend – React)

1. **Publik vy**
    - Lista och sök aktiviteter med filter (datum, kategori, inne/ute, ledigt, antal platser).
    - Detaljsida för aktivitet med kommande tillfällen.
2. **Bokningsflöde**
    - Logga in/registrera, boka plats, visa väder, bekräftelse.
    - Mina bokningar: kommande/historik, avboka.
3. **Adminvy**
    - CRUD för aktiviteter, platser och tillfällen.
    - Översikt med enkel statistik och beläggning.
4. **UI/UX**
    - Responsivt gränssnitt.
    - Tydlig felhantering och formulärvalidering.
</aside>

<aside>
**Integration**

## Integration: Väder (extern webbtjänst)

- Hämta prognos för **lat/long + tidsintervall** (t.ex. OpenWeatherMap/SMHI).
- Visa **temperatur, nederbörd, vind** i bokningsflödet och på detaljer.
- **Caching** av vädersvar (t.ex. 30–60 min per plats/tidsfönster) för att minska API‑anrop. (Extrauppgift)
</aside>

---

# Tekniska krav

<aside>
**Arkitektur**

- **N‑tier** med tydlig separation:
    - **API**: Controllers, request/response‑modeller, autentisering.
    - **Application**: Services (affärslogik), DTO:er, mapping, validering.
    - **Domain**: Entiteter, regler, interfaces.
    - **Infrastructure**: EF Core DbContext, Repositories, Migrations, externa klienter (väder‑API).
- **Repository pattern** för dataåtkomst, **Service pattern** för domänlogik.
- **Unit of Work** (implicit via DbContext eller explicit implementation).
</aside>

<aside>
**Databas**

- **SQL Server eller PostgreSQL**.
- **Code‑first** med migrationer.
- **Seed‑data** för utveckling (minst: 3 platser, 8 aktiviteter, 20+ tillfällen kommande veckor, minst 3 testanvändare för User-roll och minst en admin).
</aside>

<aside>
**DTO & Mappning**

- Separera **Entities** och **DTO:er** (Request/Response).
- Använd **AutoMapper** (el. dyl.) eller skriv egen mappning.
</aside>

<aside>
**Validering**

- **FluentValidation** för inkommande DTO:er
- Domänregler valideras i services (t.ex. dubbelbokning, kapacitetsgräns, cutoff för avbokning).
</aside>

<aside>
**Autentisering & Auktorisering**

- **JWT Bearer** med stark konfiguration (issuer, audience, signing key).
- Rollbaserad auktorisering (**User**, **Admin**) på endpoints/policys.
- (Rekommenderas) **Refresh‑tokens** eller kortlivade access‑tokens.
</aside>

<aside>
**Dokumentation & Observability**

- **OpenAPI/Swagger** för samtliga endpoints och modeller.
- **Global exception‑middleware** med standardiserade felobjekt (t.ex. RFC 7807). (Extrauppgift)
- **Logging** (struktur, korrelations‑ID per request). (Extrauppgift)
</aside>

---

# API‑design (minimikrav)

## Regler & affärslogik

- **Avboknings‑cutoff**: standard 2 timmar före start (konfigurerbart).
- **Kapacitet**: max antal samtidiga bokningar per tillfälle.
- **Dubbelbokning**: förhindra överlappande tider för samma användare.
- **Ute‑flagga**: om `IsOutdoor` (aktivitet eller plats) ⇒ hämta väder.

---

# Frontend (minimikrav)

- **Teknik**: React (Vite eller annan build-tool), JavaScript (gärna TypeScript), React Router.
- **UI**: formulär med klientvalidering, tabeller/listor, responsiv layout för åtminstone mobil och dator.
- **State/Kommunikation**: React Hooks eller egen fetch‑lösning med token‑hantering.
- **Funktioner**:
    - **Utforska**: Lista/filtrera aktiviteter och tillfällen.
    - **Detalj**: Se aktivitet + tillfällen + (ute) väder för valt tidsintervall.
    - **Bokning**: Boka/avboka, visa fel/konflikter.
    - **Mina sidor**: Mina bokningar (kommande/historik), check‑in/ut.
    - **Admin**: CRUD för aktiviteter, platser och tillfällen, samt enkel statistik.

---

# Inspirations‑ och extrauppgifter (frivilliga)

- **Docker Compose**: paketera API + DB + frontend + (om ni vill) ngn reverse proxy.
- **CI/CD**: GitHub Actions för build, tester, linting, docker‑images.
- **Integrationstester**: WebApplicationFactory + test‑DB.
- **Caching**: MemoryCache/Distributed cache för väder och tunga GETs.
- **Rate‑limiting**: skydda `/auth/login` och väderproxy.
- **Optimistic concurrency**: skydda bokningar från race conditions.

---

# Inlämning och redovisning

- **GitHub‑repo** som innehåller **backend** och **frontend**.
- **README** med:
    - Översikt över arkitektur och lager.
    - Instruktioner för lokal körning (inkl. migrations/seed, env‑variabler för väder‑API‑nyckel).
    - API‑dokumentation och beskrivning av centrala flöden.
    - Gruppens arbetsprocess (branch‑strategi, PR‑rutiner, kodstandard).
- **Muntlig demo (20-30 min)** där ni visar:
    - Arkitektur och arbetsprocess.
    - Inloggning + rollbaserad åtkomst.
    - **Fullt bokningsflöde**: sök → detalj (väder) → boka → check‑in/ut → avboka.
    - Admin: skapa aktivitet/plats/tillfälle och hur väder kopplas in för ute.
    - Ytterligare funktioner ni önskar visa.
    - Reflektioner kring arbetet på gruppnivå.

---

# Betygsättning

**Godkänt (G)** kräver att följande implementeras korrekt:

- Fullt funktionellt **REST‑API** med JWT‑autentisering och rollbaserad auktorisering.
- Arkitektur enligt specifikation (lager, repository/service, DTO/mapping, validering).
- Databas med migrations, seed‑data och centrala affärsregler.
- **Väderintegration**: hämtning för ute‑tillfällen.
- React‑frontend som stödjer de definierade användarflödena (inkl. formulärvalidering och felmeddelanden).
- Swagger‑dokumentation och README.
