# Risikovurdering - Læringsplattform LMS for Statens vegvesen

**Saksnummer**: 25/223727
**Dato**: 2026-03-22
**Versjon**: 1.0

---

## Innhold

1. [Sammendrag og topp-10 kritiske risikoer](#1-sammendrag-og-topp-10-kritiske-risikoer)
2. [Risikomatrise (Heat Map)](#2-risikomatrise-heat-map)
3. [Tekniske risikoer](#3-tekniske-risikoer)
4. [Leveranserisikoer](#4-leveranserisikoer)
5. [Kommersielle risikoer](#5-kommersielle-risikoer)
6. [Compliance-risikoer](#6-compliance-risikoer)
7. [Operasjonelle risikoer](#7-operasjonelle-risikoer)
8. [Samlet risikovurdering og anbefaling](#8-samlet-risikovurdering-og-anbefaling)

---

## Metodikk

Risikoer vurderes etter sannsynlighet (S) og konsekvens (K) på en skala Lav/Medium/Høy, som gir en risikoscore:

| Score | S x K kombinasjoner |
|-------|---------------------|
| **Kritisk** | Høy x Høy |
| **Høy** | Høy x Medium, Medium x Høy |
| **Medium** | Medium x Medium, Høy x Lav, Lav x Høy |
| **Lav** | Lav x Medium, Medium x Lav, Lav x Lav |

---

## 1. Sammendrag og topp-10 kritiske risikoer

| Rang | ID | Risiko | Score | Kategori |
|------|----|--------|-------|----------|
| 1 | T-01 | Integrasjonskompleksitet (6+ systemer) | Kritisk | Teknisk |
| 2 | L-01 | Tidspress mot go-live 01.01.2027 | Kritisk | Leveranse |
| 3 | T-03 | Migreringsrisiko fra Kilden (297 kurs + historikk) | Høy | Teknisk |
| 4 | L-03 | Avhengighet av SVVs interne team for integrasjoner | Høy | Leveranse |
| 5 | K-01 | Budsjettforbehold - SVV kan kansellere | Høy | Kommersiell |
| 6 | T-02 | SCIM-provisjonering og ForgeRock-integrasjon | Høy | Teknisk |
| 7 | C-01 | GDPR/personvern - databehandleravtale og sub-processors | Høy | Compliance |
| 8 | L-04 | Avhengighet av eksisterende leverandor (Storyboard AS) | Høy | Leveranse |
| 9 | K-03 | SSA-L dagbot og SLA-sanksjoner | Høy | Kommersiell |
| 10 | O-01 | SLA-forpliktelser over 6 ar | Høy | Operasjonell |

---

## 2. Risikomatrise (Heat Map)

```
                         K O N S E K V E N S
                    Lav           Medium          Hoy
              +---------------+---------------+---------------+
              |               |               |               |
   Hoy        |   T-06        |   T-02, L-03  |   T-01, L-01  |
              |               |   L-04, K-03  |               |
S             |               |               |               |
A             +---------------+---------------+---------------+
N             |               |               |               |
N   Medium    |   O-04        |   T-05, L-05  |   T-03, K-01  |
S             |               |   C-03, O-03  |   C-01        |
Y             |               |   K-04        |               |
N             +---------------+---------------+---------------+
L             |               |               |               |
I   Lav       |   O-05        |   T-07, L-06  |   T-04, C-02  |
G             |               |   K-05        |   C-04, O-02  |
H             |               |               |               |
E             +---------------+---------------+---------------+
T
```

**Legende:**
- Hoy x Hoy = KRITISK (rod) - Umiddelbar handlingsplan pakrevd
- Hoy x Medium / Medium x Hoy = HOY (oransje) - Aktiv mitigering
- Medium x Medium / Hoy x Lav / Lav x Hoy = MEDIUM (gul) - Overvaking
- Lav x Medium / Medium x Lav / Lav x Lav = LAV (gronn) - Akseptert

---

## 3. Tekniske risikoer

### T-01: Integrasjonskompleksitet

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-01 |
| **Tittel** | Integrasjonskompleksitet med 6+ systemer |
| **Kategori** | Teknisk |
| **Beskrivelse** | Losningen krever samtidige integrasjoner mot IAM/ForgeRock (SCIM, OAuth2, OIDC, SAML 2.0), Kursoppslag (proxy-API med eksisterende kontrakt), ServiceNow (jobbfamilier, roller, laeringslogg), PowerBI (dataeksport), eTeori (eksamen, opsjon) og Mime (arkiv, opsjon). Hver integrasjon har unike tekniske krav, autentiseringsmetoder og datamapping. Feil i en integrasjon kan blokkere andre leveranser og forsinke godkjenningsproven. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Hoy |
| **Risikoscore** | **KRITISK** |
| **Mitigerende tiltak** | 1) Prioriter integrasjoner sekvensielt: IAM forst (blokkerer alt annet), deretter Kursoppslag, ServiceNow, PowerBI. 2) Etabler integrasjonstestmiljo tidlig i oppstartsfasen (L01). 3) Bruk contract testing (f.eks. Pact) for API-grensesnitt. 4) Dedikert integrasjonsarkitekt pa teamet. 5) Krev tilgang til SVVs testmiljoer og sandkasser for alle systemer senest 2 uker etter kontraktsignering. 6) Bygg Kursoppslag-adapteren forst siden den ma beholde eksisterende API-kontrakt. |
| **Residualrisiko** | Medium - kompleksiteten forblir, men tidlig oppstart og systematisk tilnaerming reduserer forsinkelsesrisiko |
| **Eier** | Teknisk leder / Integrasjonsarkitekt |

---

### T-02: SCIM-provisjonering og ForgeRock-integrasjon

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-02 |
| **Tittel** | SCIM-provisjonering paalitelighet og edge cases |
| **Kategori** | Teknisk |
| **Beskrivelse** | Brukerens livslop (opprettelse, endringer, deprovisjonering) skal kun vedlikeholdes i SVVs masterkilder via ForgeRock AM/IDM. SCIM-provisjonering av organisasjonshierarki, roller, statuser og identifikatorer krever robust haandtering av edge cases: samtidige endringer, organisasjonsomstruktureringer, midlertidige ansettelser, konsulenter med begrenset tilgang, og re-aktivering av brukere. SSO via OAuth2/OIDC og SAML 2.0 ma fungere for bade interne (SSO) og eksterne (ID-porten/BankID) brukere. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Medium |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Krev detaljert SCIM-mapping-dokument fra SVVs IAM-team i oppstartsfasen. 2) Identifiser alle edge cases tidlig gjennom workshop med SVVs IAM-ansvarlige. 3) Implementer idempotent SCIM-endpoint med robust feilhaandtering og retry-logikk. 4) Test med realistisk brukervolum (5300+ interne + 1100 konsulenter). 5) Etabler monitoring og alerting pa SCIM-synkronisering. 6) Definer SLA for synkroniseringsforsinkelse (maks minutter, ikke timer). |
| **Residualrisiko** | Lav - med riktig forberedelse og testing er SCIM en moden standard |
| **Eier** | Integrasjonsarkitekt |

---

### T-03: Migreringsrisiko fra Kilden

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-03 |
| **Tittel** | Migrering av 297 kurs og historisk data fra Kilden |
| **Kategori** | Teknisk |
| **Beskrivelse** | L08 krever fullverdig migrering av alt laeringsinnhold (297 aktive e-laeringskurs), brukere, metadata, tilgangsstrukturer og historikk fra Kilden (Storyboard AS). Innholdet ma konverteres og standardiseres for ny plattform. Estimert pa 150 timer (medgatt tid) - dette kan vaere for lavt gitt kompleksiteten. Risiko for tap av historiske gjennomforingsdata, brutte SCORM/xAPI-pakker, og feil i metadatamapping. Kursformater (Articulate 360/Storyline/Rise) kan kreve reeksport eller tilpasning. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Hoy |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Gjennomfor migreringskartlegging i oppstartsfasen (L01) - klassifiser alle 297 kurs etter format, kompleksitet og prioritet. 2) Utfor en pilotmigrering med 20-30 representative kurs i ulike formater tidlig. 3) Etabler automatisert valideringsscript for a verifisere innholdsintegritet etter migrering. 4) Avtal eksplisitt samarbeid med Storyboard AS (ref. L08 krav). 5) Bygg inn buffer i timeestimat - 150t er trolig for lavt, beregn 250t realistisk. 6) Prioriter historisk data: definer hva som er pliktig a migrere vs. hva som kan arkiveres. |
| **Residualrisiko** | Medium - migrering har iboende risiko, men pilotmigrering og systematisk tilnaerming reduserer vesentlig |
| **Eier** | Prosjektleder / Migreringsansvarlig |

---

### T-04: Datasuverenitet og EU/EOS-lagring

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-04 |
| **Tittel** | Datasuverenitet - EU/EOS lagring uten uautoriserte overforsler |
| **Kategori** | Teknisk |
| **Beskrivelse** | PEO-kravene krever at all data lagres innenfor EU/EOS, uten uautoriserte overforsler til tredjeland. SaaS-losninger har ofte underdatabehandlere (sub-processors) med globale operasjoner. Support-tilgang fra utenfor EU/EOS, CDN-trafikk, AI-treningsdataflyt, og backup-replikering kan utgjore utilsiktede overforsler. Kravet om dokumentasjon av alle dataflyter og forhandsgodkjenning av underdatabehandlere med 30 dagers varsel er krevende. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Hoy |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Velg LMS-plattform med dokumentert EU-only datasentre og residency-konfigurasjon. 2) Kartlegg alle underdatabehandlere og deres lokasjoner for kontraktsignering. 3) Krev kontraktsfestet geo-fencing av all dataprosessering. 4) Forhandsavklar med SVVs personvernjurist og IT-sikkerhet. 5) Etabler periodisk audit av dataflyter. |
| **Residualrisiko** | Lav - med riktig plattformvalg og kontraktuell forankring |
| **Eier** | Personvernansvarlig / Juridisk |

---

### T-05: Kursoppslag API backward compatibility

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-05 |
| **Tittel** | Kursoppslag proxy-API ma beholde eksisterende kontrakt |
| **Kategori** | Teknisk |
| **Beskrivelse** | Kursoppslag er et proxy-API som abstraherer LMS for interne konsumenter. Eksisterende API-kontrakt har 6 spesifikke endepunkter med definerte felt (kursdeltakere, kursinfo, laeringsplanoppslag). Ny LMS ma returnere alle felt slik at Kursoppslag beholder sin funksjonalitet. Datamapping mellom ny LMS sitt datamodell og Kursoppslagets forventede format kan vaere utfordrende. Bade v1 og v2 av API-et ma stottess. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Kartlegg alle Kursoppslag-endepunkter og felt i detalj under oppstartsfasen. 2) Bygg adapter/mapping-lag mellom ny LMS API og Kursoppslag-formatet. 3) Bruk contract testing mot eksisterende Kursoppslag-spesifikasjon. 4) Test med reelle interne konsumenter for funksjonsbekreftelse. 5) Avklar om bade v1 og v2 ma stottess i parallell. |
| **Residualrisiko** | Lav - API-kontrakten er veldokumentert og forutsigbar |
| **Eier** | Integrasjonsarkitekt |

---

### T-06: Ytelse ved skalering

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-06 |
| **Tittel** | Ytelse ved 5300+ interne og 15000+ eksterne brukere med sesongtopper |
| **Kategori** | Teknisk |
| **Beskrivelse** | KKP-portalen (opsjon L21) har sesongbasert bruksmoenster med topper knyttet til arbeidsvarsling og vinterdrift. Kombinert med interne brukere som gjennomforer obligatoriske HMS-kurs med felles frister, kan det oppsta betydelige samtidige belastninger. Ytelseskrav (TI-01 til TI-05) spesifiserer responstider og oppetid. Eksamenssituasjoner krever null nedetid (ingen vedlikehold under eksamener). |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Lav |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Velg SaaS-plattform med dokumentert auto-skalering. 2) Gjennomfor lasttesting med realistiske brukerprofiler for godkjenningsprove. 3) Etabler eksamenssesong-kalender med vedlikeholdsforbud. 4) Definer tydelig capacity planning basert pa sesongmoenster. 5) Implementer CDN for statisk innhold og e-laeringsmoduler. |
| **Residualrisiko** | Lav - moderne SaaS-plattformer haandterer denne skalaen godt |
| **Eier** | Teknisk leder |

---

### T-07: AI-funksjonalitet og treningsdata

| Felt | Beskrivelse |
|------|-------------|
| **ID** | T-07 |
| **Tittel** | AI-krav til personalisering, innholdsgenerering og forklarbarhet |
| **Kategori** | Teknisk |
| **Beskrivelse** | KI-01 krever beskrivelse av AI-teknologi, treningsdata og GDPR-samsvar. SVV onsker personlig tilpasset laering, anbefalinger og AI-generert kursinnhold. AI-funksjonalitet i LMS-plattformer er fortsatt umodent, og treningsdata fra norsk offentlig sektor er begrenset. Krav til forklarbarhet og etikk etter norske standarder kan begrense valg av AI-modeller. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Medium |
| **Risikoscore** | **LAV** |
| **Mitigerende tiltak** | 1) Posisjonere AI som evolusjonsoer funksjonalitet som bygges ut over kontraktsperioden. 2) Start med regelbasert personalisering og anbefalinger, utvid med AI over tid. 3) Dokumenter AI-arkitektur, treningsdata og bias-mitigering tydelig. 4) Sikre at AI-funksjonalitet kan deaktiveres per modul. 5) Henvis til plattformens AI-roadmap. |
| **Residualrisiko** | Lav - AI-kravene er evaluererbare, ikke absolutte |
| **Eier** | AI/ML-spesialist |

---

## 4. Leveranserisikoer

### L-01: Tidspress mot go-live 01.01.2027

| Felt | Beskrivelse |
|------|-------------|
| **ID** | L-01 |
| **Tittel** | Tidspress - kontraktsignering september 2026, mal om go-live 01.01.2027 |
| **Kategori** | Leveranse |
| **Beskrivelse** | SVV har som mal at ny laeringsplattform tas i bruk fra 01.01.2027, med kontraktsignering 01.09.2026. Dette gir kun 4 maneder for oppstart (L01), behovsanalyse (L02), konfigurasjon (L03), IAM-integrasjon (L04), ServiceNow-integrasjon (L05), PowerBI-integrasjon (L06), laeringsflater (L07), migrering (L08), opplaering (L09) og akseptansetest (L10). Godkjenningsproven alene krever 4 uker (20 virkedager). Dette er en ekstremt stram tidslinje for en losning med 6+ integrasjoner. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Hoy |
| **Risikoscore** | **KRITISK** |
| **Mitigerende tiltak** | 1) Forhandel realistisk tidslinje i forhandlingsrundene - foreslaa go-live Q1/Q2 2027 med pilotering i Q4 2026. 2) Prioriter en MVP-tilnaerming: lanser med kjernefunksjonalitet (SSO, basiskurs, grunnleggende administrasjon) og lever integrasjoner inkrementelt. 3) Parallelliser leveranser der mulig (L02 kan starte samtidig med L03/L04). 4) Start forberedende arbeid (arkitektur, migreringskartlegging) umiddelbart etter kontraktsignering. 5) Definer en "leveringsdag 1" med minstekrav og "leveringsdag 2" for fulle integrasjoner. 6) Avklar med SVV at 01.01.2027 er et mal, ikke en absolutt frist. |
| **Residualrisiko** | Medium - tidspress forblir, men inkrementell leveranse reduserer risikoen for total forsinkelse |
| **Eier** | Prosjektleder |

---

### L-02: Ressurstilagjengelighet og nokkelpersonell

| Felt | Beskrivelse |
|------|-------------|
| **ID** | L-02 |
| **Tittel** | Ressurstilajengelighet og nokkelpersonell-krav |
| **Kategori** | Leveranse |
| **Beskrivelse** | SVV krever CV-godkjenning av nokkelpersonell for kontraktsignering, inkludert mulighet for intervju og case-oppgaver. Ved bytte av nokkelpersonell ma nytt personell ha tilsvarende eller bedre kompetanse, med 10-20 virkedagers overleveringsperiode betalt av leverandoren. Prosjektleder skal vaere SPOC gjennom hele avtaleperioden (potensielt 6 ar). Over sa lang tid er personellroasjon uunngaelig. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Identifiser backup-personell for alle nokkelloller fra start. 2) Dokumenter all konfigurasjon og beslutninger for a redusere personavhengighet. 3) Bygg kunnskapsbase internt som muliggjor raskere onboarding. 4) Beregn kostnaden for 20-dagers kompetanseoverforinger i prismodellen. 5) Vurder om SPOC-rollen kan overfolres til en dedikert kundeansvarlig etter etableringsfasen (eksplisitt tillatt i kontrakten). |
| **Residualrisiko** | Lav - med god dokumentasjon og forberedte backup-ressurser |
| **Eier** | Prosjektleder / HR |

---

### L-03: Avhengighet av SVVs interne team

| Felt | Beskrivelse |
|------|-------------|
| **ID** | L-03 |
| **Tittel** | Avhengighet av SVVs interne team for integrasjoner |
| **Kategori** | Leveranse |
| **Beskrivelse** | Leveransene L04-L06 krever tett samarbeid med SVVs IAM-team (ForgeRock), ServiceNow-team og PowerBI-team. Disse teamene har egne prioriteringer og kapasitetsbegrensninger. SVV har sagt at de vil ferdigstille interne forberedelser (endepunkter, API-dokumentasjon, nokkler/sertifikater) for integrasjonsarbeidet starter, men dette er en lovnad uten kontraktsfestet garanti. Forsinkelser fra SVV-siden pavirker leverandorens tidslinje direkte. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Medium |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Definer klare avhengigheter og tidsfrister i prosjektplanen med SVVs forpliktelser eksplisitt markert. 2) Etabler eskaleringsmekanisme via SPOC og bemyndiget representant. 3) Bruk mock-tjenester og sandkasser for a kunne utvikle integrasjoner uavhengig av SVVs klargjoring. 4) Definer "blocker"-rapportering i ukentlige statusmoter. 5) Bygg inn buffer for SVV-avhengigheter i tidsplanen. |
| **Residualrisiko** | Medium - avhengigheten kan ikke elimineres, kun mitigeres gjennom tydelig kommunikasjon |
| **Eier** | Prosjektleder |

---

### L-04: Avhengighet av eksisterende leverandor (Storyboard AS)

| Felt | Beskrivelse |
|------|-------------|
| **ID** | L-04 |
| **Tittel** | Samarbeid med Storyboard AS for migrering og parallellkjoring |
| **Kategori** | Leveranse |
| **Beskrivelse** | L07 og L08 spesifiserer at arbeidet skal utfores "i samarbeid med SVVs kompetanseteam og, ved behov, eksisterende leverandor av Kilden" (Storyboard AS). Storyboard AS er en konkurrent som mister kontrakten og har begrenset insentiv til a samarbeide. Datatilgang, API-dokumentasjon og teknisk stotte fra Storyboard kan bli vanskelig a fa. Kilden-kontrakten utloper 2027, sa det ma vaere en parallellkjoringsperiode. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Medium |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Avklar med SVV tidlig hvilke kontraktuelle forpliktelser Storyboard AS har for dataeksport og samarbeid. 2) Be SVV om a hente ut all migreringsdata (kurs, brukere, historikk) i standardformater (SCORM-pakker, CSV/JSON) for ny leverandor starter. 3) Reduser avhengigheten av Storyboard ved a bruke SVVs egne data-uttrekk. 4) Planlegg for at samarbeidet kan vaere minimalt og forbered alternative migreringsstategier. 5) Etabler parallelldrift med tydelig cutover-plan. |
| **Residualrisiko** | Medium - avhengigheten kan reduseres men ikke elimineres helt |
| **Eier** | Prosjektleder |

---

### L-05: Endringshaandtering pa tvers av 6 divisjoner

| Felt | Beskrivelse |
|------|-------------|
| **ID** | L-05 |
| **Tittel** | Endringshaandtering pa tvers av divisjoner med ulike behov |
| **Kategori** | Leveranse |
| **Beskrivelse** | SVV bestar av Vegdirektoratet og 6 divisjoner (Trafikant og kjoretoy, Transport og samfunn, Utbygging, Drift og vedlikehold, Fellesfunksjoner og HR, IT) med ulike laeringsbehov. L02 krever behovsanalyse og forankring med alle divisjoner. KKP-portalen eies av divisjon Transport og samfunn, mens Kilden eies av Fellesfunksjoner og HR. Interessekonflikter mellom divisjoner kan forsinke beslutninger om laeringsflater, roller og innholdsforvaltning. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Foreslaa en strukturert interessentkartlegging og governance-modell tidlig i L02. 2) Bruk fasiliterte workshops med tydelig beslutningsmandat. 3) Definer laeringsarkitekturen med fleksibilitet per divisjon innenfor felles rammeverk. 4) Identifiser og haandter konflikter mellom KKP (Transport og samfunn) og Kilden (HR) tidlig. 5) Utnevn divisjonsambassadorer som change champions. |
| **Residualrisiko** | Lav - med tydelig governance og god fasilitering |
| **Eier** | Prosjektleder / Endringsleder |

---

### L-06: Parallell drift og overgansperiode

| Felt | Beskrivelse |
|------|-------------|
| **ID** | L-06 |
| **Tittel** | Parallell drift mellom gammel og ny losning |
| **Kategori** | Leveranse |
| **Beskrivelse** | Under overgangsperioden ma bade Kilden og ny LMS vaere operative. Brukere, kursdata og gjennomforingshistorikk ma synkroniseres eller dupliseres. Risiko for forvirring blant brukere, dupliserte registreringer, og inkonsistente data. KKP-portalen (avtale 2023-2029) kan ha en annen overgangstidslinje enn Kilden (2027). |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Medium |
| **Risikoscore** | **LAV** |
| **Mitigerende tiltak** | 1) Definer en tydelig cutover-strategi: big-bang eller fasert per divisjon/brukergruppe. 2) Etabler tydelig kommunikasjonsplan for alle brukere. 3) Begrens parallellkjoringsperioden til maksimalt 2-4 uker. 4) Sett opp redirect fra gammel losning til ny. 5) Haandter KKP-overgang separat (opsjon L21) med egen tidslinje. |
| **Residualrisiko** | Lav |
| **Eier** | Prosjektleder |

---

## 5. Kommersielle risikoer

### K-01: Budsjettforbehold

| Felt | Beskrivelse |
|------|-------------|
| **ID** | K-01 |
| **Tittel** | SVV kan kansellere dersom budsjett ikke godkjennes |
| **Kategori** | Kommersiell |
| **Beskrivelse** | SVV forbeholder seg retten til a avlyse konkurransen dersom budsjett ikke godkjennes. Med en ramme pa 20-35 MNOK over 6 ar og SVVs totale budsjett pa 55.3B NOK er dette en liten post, men offentlige budsjetter er politisk styrt. Leverandoren kan investere betydelig i tilbudsprosessen (kvalifisering, POC-demonstrasjon, forhandlinger) uten garanti for kontrakt. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Hoy |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Vurder budsjettforbeholdet som akseptabel markedsrisiko - det er standard i offentlige anskaffelser. 2) Begrens tilbudskostnader gjennom gjenbruk av eksisterende POC-miljoer og standardiserte svar. 3) Folg opp signaler fra SVV om budsjettgodkjenning gjennom tilbyderkonfersansen og sporsmalsprosessen. 4) Dimensjoner tilbudsinnsatsen etter sannsynlighet for tildeling. |
| **Residualrisiko** | Medium - risikoen er iboende i offentlige anskaffelser |
| **Eier** | Tilbudsansvarlig / Kommersiell leder |

---

### K-02: Prispress i 20-35 MNOK ramme

| Felt | Beskrivelse |
|------|-------------|
| **ID** | K-02 |
| **Tittel** | Prispress og marginer over 6 ar |
| **Kategori** | Kommersiell |
| **Beskrivelse** | 20-35 MNOK over 6 ar (3.3-5.8 MNOK/ar) skal dekke SaaS-lisenser, implementering, integrasjoner, support, opplaering og videreutvikling. Pris vektes kun 20% mot kvalitet 80%, men prisen ma likevel vaere konkurransedyktig. Prisene er faste i 2 ar, deretter KPI-justert. Timeprisene er faste til 31.12.2027. Leverandoren bar risikoen for kostnadsokninger i denne perioden. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Fokuser pa kvalitet (80%) fremfor a underprise. 2) Beregn total cost of ownership grundig inkludert sub-processor-kostnader, supportbemanning og oppgraderinger. 3) Bygg inn inflasjonsrisiko i fastprisperioden. 4) Vurder site-lisens vs. per-bruker for a optimalisere lisenskostnad. 5) Pris opsjoner (L21-L24) med tilstrekkelig margin da de ikke evalueres. |
| **Residualrisiko** | Lav - med realistisk prising og fokus pa kvalitetsdifferensiering |
| **Eier** | Kommersiell leder |

---

### K-03: SSA-L dagbot og SLA-sanksjoner

| Felt | Beskrivelse |
|------|-------------|
| **ID** | K-03 |
| **Tittel** | SSA-L kontraktsregime med okonomisk kompensasjon for SLA-brudd |
| **Kategori** | Kommersiell |
| **Beskrivelse** | Bilag 4 krever at leverandoren beskriver okonomiske kompensasjoner for brudd pa tjenesteniva. Godkjenningsproven har strenge akseptansekriterier: 0 A-feil, 3 B-feil, 20 C-feil for forste runde; 0 A-feil, 0 B-feil, 10 C-feil for andre runde. Leverandoren fakturerer forst etter godkjent godkjenningsprove, noe som betyr at all implementeringskostnad baeres uten inntekt til godkjenning. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Medium |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Foreslaa rimelige SLA-kompensasjoner med ovre tak per maned og ar. 2) Definer feilkategorisering (A/B/C) tydelig i samarbeid med SVV under L01. 3) Gjennomfor grundig intern testing for godkjenningsprove. 4) Planlegg kontantflyt for at all implementeringskostnad palopser for fakturering. 5) Forhandle delbetaling ved milepaler i stedet for alt etter godkjenningsprove hvis mulig. 6) Bygg inn prosjektbuffer for a haandtere andre runde av godkjenningsprove. |
| **Residualrisiko** | Medium - SSA-L-regimet er krevende men forutsigbart |
| **Eier** | Kommersiell leder / Prosjektleder |

---

### K-04: Nokkelpersonell-laasing og byttekostnader

| Felt | Beskrivelse |
|------|-------------|
| **ID** | K-04 |
| **Tittel** | Nokkelpersonell-krav med 10-20 dagers overleveringsperiode |
| **Kategori** | Kommersiell |
| **Beskrivelse** | Ved bytte av nokkelpersonell baerer leverandoren alle kostnader for kompetanseoverforingen (10-20 virkedager). Nytt personell kan ikke faktureres for overforingen er gjennomfort. Over en 6-ars avtaleperiode er personellrotasjon uunngaelig. Hver utskiftning medforer direkte kostnad (10-20 dagsverk x 2 personer) pluss indirekte kostnad (produktivitetstap, kundetilfredshet). |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Kalkulator inn 1-2 nokkelpersonellbytter i kontraktsperioden i prismodellen. 2) Prioriter teamstabilitet og karriereutvikling for a redusere rotasjon. 3) Dokumenter grundig for a redusere overleveringsperioden i praksis. 4) Definer tydelig hva som utgjor "nokkelpersonell" - begrens til prosjektleder og eventuelt teknisk arkitekt. |
| **Residualrisiko** | Lav - med kalkulert kostnad og god personalforvaltning |
| **Eier** | Kommersiell leder / HR |

---

### K-05: Opsjonsrisiko (KKP-migrering L21 avhenger av L22)

| Felt | Beskrivelse |
|------|-------------|
| **ID** | K-05 |
| **Tittel** | Opsjonsprising - KKP-migrering avhenger av eTeori-integrasjon |
| **Kategori** | Kommersiell |
| **Beskrivelse** | Opsjon L21 (KKP-portalmigrering, 250t) er avhengig av L22 (eTeori-integrasjon, 70t). KKP har 350.000 kompetansebevis og 15.000 nye arlig. Opsjonen inkluderer egen laeringsflate, migrering av innhold, grensesnitt mot nasjonalt kompetanseregister, og lisenskostnader for eksterne kursholdere og deltakere. Kompleksiteten er betydelig men opsjoner evalueres ikke pa pris. Sesongbegrensninger pavirker nar migrering kan gjennomfores. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Medium |
| **Risikoscore** | **LAV** |
| **Mitigerende tiltak** | 1) Pris opsjoner med god margin da de ikke evalueres. 2) Definer tydelige forutsetninger for oppsjonsutovelse. 3) Kartlegg sesongbegrensninger (vinterdrift-sesong) tidlig. 4) Beregn opsjon L21 uavhengig av L22 i kostnadsmodellen for a synliggjore avhengigheten. 5) Vurder KKP-migrering som et eget delprosjekt med separat tidslinje. |
| **Residualrisiko** | Lav |
| **Eier** | Kommersiell leder |

---

## 6. Compliance-risikoer

### C-01: GDPR/personvern og databehandleravtale

| Felt | Beskrivelse |
|------|-------------|
| **ID** | C-01 |
| **Tittel** | GDPR-etterlevelse, databehandleravtale og underdatabehandlerhaandtering |
| **Kategori** | Compliance |
| **Beskrivelse** | PEO-kravene (PEO-13 til PEO-29) krever omfattende GDPR-etterlevelse: databehandleravtale basert pa SSA-mal, godkjenning av SVVs personvernjurist, komplett underdatabehandlerliste med 30-dagers varsel ved endringer (SVV kan nekte uten begrunnelse), dokumentasjon av alle dataflyter, og EU/EOS-lagring. SaaS-plattformer endrer underdatabehandlere regelmessig, og SVVs vetorett kan skape operasjonelle utfordringer. Losningen behandler sensitive personopplysninger (kompetansebevis, HMS-kurs, helseattester). |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Hoy |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Forbered komplett underdatabehandlerliste med lokasjoner og roller for tilbudsinnlevering. 2) Avklar med LMS-plattformleverandor deres prosess for underdatabehandler-endringer og SVVs vetorett. 3) Bruk SSA-malen for databehandleravtale (Bilag 5 vedlegg 1) som utgangspunkt. 4) Gjennomfor DPIA (Data Protection Impact Assessment) i oppstartsfasen. 5) Etabler internkontroll for personvernlovgivning. 6) Sikre at LMS-plattformen har innebygd personvern (privacy by design). |
| **Residualrisiko** | Medium - GDPR-etterlevelse krever lopende oppmerksomhet |
| **Eier** | Personvernansvarlig / Juridisk |

---

### C-02: Arkivloven - kompetansebevis

| Felt | Beskrivelse |
|------|-------------|
| **ID** | C-02 |
| **Tittel** | Arkivplikt for kompetansebevis (arkivloven) |
| **Kategori** | Compliance |
| **Beskrivelse** | Kompetansebevis er arkivpliktige dokumenter etter arkivloven. KKP-portalen utsteder 15.000 kompetansebevis arlig. Opsjon L24 (Mime-integrasjon) dekker integrasjon mot SVVs arkivsystem, men dette er en opsjon - ikke en garantert leveranse. Uten arkivintegrasjon ma kompetansebevis arkiveres manuelt. Arkivkravene gjelder ogsa for Kilden-migrerte kursgjennomforinger. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Hoy |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Sikre at LMS-plattformen kan generere og lagre kompetansebevis i arkivformat (PDF/A). 2) Bygg inn eksportfunksjonalitet for kompetansebevis til Mime-kompatibelt format. 3) Tilby Mime-integrasjonen (L24) som en sterkt anbefalt opsjon. 4) Dokumenter arkivrutiner ogsa uten Mime-integrasjon (manuell/halv-automatisk). 5) Avklar med SVVs arkivleder hvilke dokumenttyper som er arkivpliktige. |
| **Residualrisiko** | Lav - med riktig eksportfunksjonalitet |
| **Eier** | Juridisk / Funksjonelt ansvarlig |

---

### C-03: WCAG og universell utforming

| Felt | Beskrivelse |
|------|-------------|
| **ID** | C-03 |
| **Tittel** | WCAG-etterlevelse og norsk lov om universell utforming |
| **Kategori** | Compliance |
| **Beskrivelse** | Krav 01.013 og 01.015 krever full WCAG-compliance og universell utforming i henhold til norsk lov. Gjelder bade plattform-UI og alt laeringsinnhold. Migrert innhold fra Kilden (297 kurs, mange i Articulate Storyline/Rise) er sannsynligvis ikke WCAG-tilpasset. Ansvar for eksisterende innholds tilgjengelighet vs. ny plattforms tilgjengelighet ma avklares. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Velg LMS-plattform med dokumentert WCAG 2.1 AA-samsvar og VPAT. 2) Skille mellom plattformtilgjengelighet (leverandorens ansvar) og innholdstilgjengelighet (SVVs ansvar). 3) Tilby WCAG-vurdering av migrert innhold som tilleggstjeneste. 4) Sikre at opplaeringsdokumentasjon og systemstotte ogsa er WCAG-tilpasset. 5) Planlegg for Digitaliseringsdirektoratets tilsynskontroll. |
| **Residualrisiko** | Lav - moderne SaaS-plattformer har generelt god WCAG-stotte |
| **Eier** | UX-ansvarlig |

---

### C-04: Informasjonssikkerhet og penetrasjonstesting

| Felt | Beskrivelse |
|------|-------------|
| **ID** | C-04 |
| **Tittel** | Informasjonssikkerhetskrav: logging, kryptering, MFA, DDoS, pentesting |
| **Kategori** | Compliance |
| **Beskrivelse** | RO-kravene (RO-01 til RO-05) krever ROS-analyse (risiko- og sarbarhetsanalyse), sarbarhetshaandtering, penetrasjonstesting og minimal datainnsamling. APP-kravene krever ende-til-ende dataflysikkerhet og mobilkryptering. NET-13 krever DDoS-beskyttelse. LOG-04 krever full aktivitetslogging for sporbarhet. SVV kan kreve penetrasjonstesting som del av godkjenningsproven. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Hoy |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Velg SaaS-plattform med SOC 2 Type II og/eller ISO 27001-sertifisering. 2) Legg ved eksisterende penetrasjonstest-rapport fra plattformleverandor. 3) Tilby a gjennomfore dedikert pentest for SVV-installasjonen etter konfigurasjon. 4) Dokumenter DDoS-mitigering (typisk levert av cloud provider). 5) Implementer loggintegrasjon med SVVs SIEM om nodvendig. 6) Forbered ROS-analyse for tilbudet. |
| **Residualrisiko** | Lav - etablerte SaaS-plattformer har robust sikkerhet |
| **Eier** | Sikkerhetsansvarlig |

---

## 7. Operasjonelle risikoer

### O-01: SLA-forpliktelser over 6 ar

| Felt | Beskrivelse |
|------|-------------|
| **ID** | O-01 |
| **Tittel** | SLA-opprettholdelse over potensiell 6-arsperiode |
| **Kategori** | Operasjonell |
| **Beskrivelse** | Avtalen kan vare i opptil 6 ar (3+1+1+1) med SLA-krav til oppetid, responstider, brukerstotte pa norsk (1. og 2. linje) og feilrettingsfrister. Support skal vaere tilgjengelig kl. 08:00-16:00 pa virkedager. Ingen vedlikehold under eksamener. Over 6 ar kan plattformteknologi endre seg vesentlig, supportteam kan skifte, og kostnadsgrunnlaget kan endres. SLA-brudder utloeser okonomisk kompensasjon. |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Hoy |
| **Risikoscore** | **HOY** |
| **Mitigerende tiltak** | 1) Bygg supportmodell basert pa teamkapasitet, ikke enkeltpersoner. 2) Etabler SLA-dashboard for proaktiv overvaking. 3) Definer vedlikeholdsvindukalender i samarbeid med SVV (ekskluder eksamensperioder). 4) Automatiser overvaking og alerting. 5) Forhandle SLA som er realistisk a opprettholde over tid. 6) Inkluder arlige SLA-gjennomganger i samhandlingsmodellen. |
| **Residualrisiko** | Medium - langtids-SLA krever vedvarende oppmerksomhet |
| **Eier** | Driftsleder / SPOC |

---

### O-02: Plattformleverandor-avhengighet

| Felt | Beskrivelse |
|------|-------------|
| **ID** | O-02 |
| **Tittel** | Avhengighet av SaaS-plattformleverandorens prising og roadmap |
| **Kategori** | Operasjonell |
| **Beskrivelse** | Som SaaS-losning er leverandoren avhengig av underliggende plattformleverandor (f.eks. Docebo, Cornerstone, Totara, etc.). Plattformleverandoren kan endre prising, fjerne funksjoner, endre API-er, eller bli kjopt opp. SSA-L Bilag 6 tillater kun prisjustering basert pa dokumenterte endringer i skytjenestekostnader. Leverandoren baerer risikoen for gap mellom plattformendringer og kontraktsforpliktelser. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Hoy |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Velg plattform med langsiktig, stabil prising og enterprise-avtaler. 2) Siker kontraktfestet pristak fra plattformleverandor for avtaleperioden. 3) Dokumenter plattformens API-stabilitetspolicy. 4) Ha en beredskapsplan for plattformbytte (exit-strategi). 5) Overvak plattformleverandorens finansielle helse og M&A-aktivitet. |
| **Residualrisiko** | Lav - med langsiktig avtale med plattformleverandor |
| **Eier** | Kommersiell leder |

---

### O-03: Kunnskapsoverfoering ved kontraktslutt

| Felt | Beskrivelse |
|------|-------------|
| **ID** | O-03 |
| **Tittel** | Datauttrekk og transisjonsstotte ved kontraktslutt |
| **Kategori** | Operasjonell |
| **Beskrivelse** | Ved opphoor skal kunden ha tilgang til egne data, datastrukturer og metadata i minimum 3 maneder. Data skal kunne hentes ut i lesbart format uten ekstra kostnad. Over 6 ar vil datamengden vokse betydelig (kursgjennomforinger, kompetansebevis, brukerhistorikk). Eksportprosessen ma planlegges og kostnadsberegnes fra start. Konfigurasjoner og tilpasninger skal kunne overfores (bruksrett beholdes av SVV). |
| **Sannsynlighet** | Medium |
| **Konsekvens** | Medium |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Design datamodellen med eksportbarhet fra dag en. 2) Tilby standardiserte dataeksporter (CSV, JSON, SCORM-pakker). 3) Dokumenter datamodell og API-er lopende. 4) Inkluder arlig dataeksport-test i SLA-gjennomgangen. 5) Beregn transisjonskostnad og inkluder i prismodellen. |
| **Residualrisiko** | Lav - med god forberedelse |
| **Eier** | Teknisk leder |

---

### O-04: Lopende innholdsproduksjon og vedlikehold

| Felt | Beskrivelse |
|------|-------------|
| **ID** | O-04 |
| **Tittel** | Lopende kursvedlikehold og innholdsproduksjon over kontraktsperioden |
| **Kategori** | Operasjonell |
| **Beskrivelse** | SVV bruker Articulate 360, Junglemap og Vyond til innholdsproduksjon - disse er ikke integrert i LMS. Med 297 eksisterende kurs og lopende behov for nye (18.500 gjennomforinger/ar), ma plattformen effektivt stotte import og vedlikehold av SCORM/xAPI/cmi5-innhold. L12 gir 200t estimert for endringer og tillegg, som kan vaere utilstrekkelig over 6 ar. |
| **Sannsynlighet** | Hoy |
| **Konsekvens** | Lav |
| **Risikoscore** | **MEDIUM** |
| **Mitigerende tiltak** | 1) Sikre robust SCORM 1.2/2004, xAPI og cmi5-stotte i valgt plattform. 2) Dokumenter importprosesser for alle SVVs innholdsverktoy. 3) Tilby endringstimer (L12) med mulighet for paafyll etter behov. 4) Anbefal innebygd kursbygger i LMS for enklere innholdsproduksjon. 5) Etabler best-practice for innholdsforvaltning i L02/L07. |
| **Residualrisiko** | Lav |
| **Eier** | Funksjonelt ansvarlig |

---

### O-05: Brukerstotte pa norsk

| Felt | Beskrivelse |
|------|-------------|
| **ID** | O-05 |
| **Tittel** | Norskspraakling 1. og 2. linjestotte over kontraktsperioden |
| **Kategori** | Operasjonell |
| **Beskrivelse** | SVV krever at 1. og 2. linjestotte gis pa norsk, mens 3. linje kan vaere pa norsk/engelsk. Med 5.300+ interne og potensielt 15.000+ eksterne brukere ma supportkapasiteten vaere robust. Rekruttering og beholding av norskspraklig teknisk support over 6 ar kan vaere utfordrende. |
| **Sannsynlighet** | Lav |
| **Konsekvens** | Lav |
| **Risikoscore** | **LAV** |
| **Mitigerende tiltak** | 1) Etabler dedikert norskspraklig supportteam. 2) Bygg kunnskapsbase og FAQ pa norsk. 3) Implementer selvbetjent stotte (chatbot, hjelpeartikler) for a redusere supportvolum. 4) Definer tydelig grensesnitt mellom 1., 2. og 3. linjestotte. 5) Maal og rapporter supportvolum for a dimensjonere riktig. |
| **Residualrisiko** | Lav |
| **Eier** | Supportleder |

---

## 8. Samlet risikovurdering og anbefaling

### Risikoprofil

| Risikoniva | Antall | Risikoer |
|------------|--------|----------|
| **Kritisk** | 2 | T-01, L-01 |
| **Hoy** | 6 | T-02, T-03, L-03, L-04, K-01, K-03, C-01, O-01 |
| **Medium** | 9 | T-05, T-06, L-02, L-05, K-02, K-04, C-02, C-03, C-04, O-02, O-03, O-04 |
| **Lav** | 4 | T-07, L-06, K-05, O-05 |

### Samlet vurdering

Prosjektet har en **moderat til hoy risikoprofil**, preget av:

1. **To kritiske risikoer** som krever umiddelbar mitigering:
   - Integrasjonskompleksiteten (T-01) er den storste tekniske utfordringen. 6+ systemintegrasjoner med ulike teknologier (SCIM, OAuth2, SAML, REST) og avhengigheter krever en erfaren integrasjonsarkitekt og systematisk tilnaerming.
   - Tidspresset (L-01) med 4 maneder fra kontraktsignering til onsket go-live er den storste leveranserisikoen. En inkrementell MVP-tilnaerming er nodvendig.

2. **Sterke mitigerende faktorer**:
   - SVV har god dokumentasjon av krav og integrasjoner
   - SSA-L er et kjent kontraktsrammeverk
   - SaaS-modellen reduserer infrastrukturrisiko
   - Kvalitet vektes 80% - dette favoriserer erfarne leverandorer fremfor lavprisaktorer

3. **Kritiske suksessfaktorer for tilbudet**:
   - Demonstrer dyp integrasjonserfaring i POC-demonstrasjonen
   - Presenter en troverdig, inkrementell leveranseplan
   - Vis konkret erfaring med SCIM/ForgeRock og norsk offentlig sektor
   - Pris realistisk - SVV vil vurdere gjennomforingsevne hoyt

### Anbefalte prioriterte handlinger for tilbudet

| Prioritet | Handling | Tidspunkt |
|-----------|----------|-----------|
| 1 | Avklar plattformvalg og verifiser alle integrasjonsmuligheter (SCIM, Kursoppslag API-mapping, ServiceNow) | For tilbudsinnlevering |
| 2 | Utarbeid detaljert, inkrementell leveranseplan med MVP-definisjon | Tilbudet |
| 3 | Forbered komplett GDPR-dokumentasjon og underdatabehandlerliste | Tilbudet |
| 4 | Planlegg POC-demonstrasjon som viser integrasjonskompetanse | For demonstrasjon 11-12.05 |
| 5 | Gjennomfor intern kalkyle av kontantflytrisiko (ingen fakturering for godkjenningsprove) | For tilbudsinnlevering |
| 6 | Identifiser og lås nokkelpersonell med ForgeRock/IAM og ServiceNow-erfaring | For kvalifisering |
| 7 | Etabler kontakt med ForgeRock for a verifisere SCIM-stotte i valgt LMS | Umiddelbart |

---

*Dokumentet er utarbeidet som del av tilbudsarbeidet og skal oppdateres lopende gjennom tilbudsprosessen og forhandlingsrundene.*
