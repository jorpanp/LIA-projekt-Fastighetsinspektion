# PropertyInspection – LIA-projekt Fastighetsinspektion

Ett automatiserat system för AI-driven fastighetsinspektion byggt i **UiPath**. Systemet tar emot inspektionsdata, analyserar fynd med hjälp av AI-agenter och genererar strukturerade skaderapporter – med stöd för mänsklig granskning vid eskalering.

---

## Syfte

Projektet syftar till att automatisera hanteringen av fastighetsbesiktningar. Inspektionsdata (iakttagelser, bilder, adressinformation) skickas in till systemet som klassificerar skadetyp, bedömer allvarlighetsgrad och producerar en rapport. Vid hög risk eller identifierad hälsorisk eskaleras ärendet till en mänsklig handläggare via UiPath Action Center.

---

## Arkitektur

Systemet består av fyra huvudkomponenter som samverkar i ett BPMN-flöde:

```
┌──────────────────────────────────────────────┐
│              Main Flow (BPMN)                │
│                                              │
│  Inkommande data → Analyze Inspection        │
│         │                                    │
│         ├── Elskada?  → Electricity Agent    │
│         └── Vattenskada? → Water & Moisture  │
│                  │                           │
│         Eskalering krävs?                    │
│         ├── Ja  → Action Center (Human Review│
│         └── Nej → Skicka rapport via Gmail   │
└──────────────────────────────────────────────┘
```

### Komponenter

**Main Flow** (`Main Flow/MainFlow.bpmn`)
Orkestreringsflödet som styr hela processen. Tar emot inspektionsdata, anropar rätt agent baserat på skadekategori, hanterar eskalering och skickar slutrapport via e-post (Gmail-integration).

**Analyze Inspection** (`Analyze Inspection/Main.xaml`)
UiPath-process som tar emot inkommande inspektionsdata och identifierar om och vilken typ av skada som föreligger (el eller vatten/fukt). Skickar vidare relevanta fält: iakttagelser, bildnyckel, adress och ärendenummer.

**Electricity – Agent** (`Electricity - Agent/agent.json`)
AI-agent specialiserad på elinstallationer och elrelaterade risker. Analyserar iakttagelser mot gällande lagstiftning, bedömer kritikalitet (Hög/Medel/Låg) och hälsorisk, beräknar åtgärdsperiod och genererar en HTML-rapport.

**Water and Moisture – Agent** (`Water and Moisture - Agent/agent.json`)
AI-agent specialiserad på vattenskador och fuktskador. Samma analyslogik som elagenten men anpassad för fukt- och vattenskaderelaterade regelverk. Returnerar även ett strukturerat `EscalationInfo`-objekt för vidare beslutslogik.

### Integrationer

- **UiPath Data Service** – lagring av fastighets- och ärendedata
- **Gmail** – mottagning av inspektionsmail och utskick av rapport
- **UiPath Action Center** – eskalering till mänsklig handläggare via HITL-uppgifter (Human-In-The-Loop)
- **Storage Bucket** – lagring av bifogade inspektionsbilder

---

## Kom igång

> **Förutsättningar:** Tillgång till en UiPath Orchestrator-instans med stöd för Agenter, Action Center och Data Service.

### 1. Klona repot

```bash
git clone https://github.com/jorpanp/LIA-projekt-Fastighetsinspektion.git
cd LIA-projekt-Fastighetsinspektion/PropertyInspection
```

### 2. Konfigurera anslutningar

Anslutningar (connections) är inte inkluderade i repot av säkerhetsskäl. Dessa behöver sättas upp manuellt i Orchestrator:

- Gmail-anslutning (för att ta emot och skicka e-post)
- UiPath Data Service-anslutning
- Storage Bucket-anslutning (för bildhantering)

Uppdatera sedan `bindings_v2.json` i `Main Flow/` med dina connection-IDs.

### 3. Konfigurera resurser

Exkluderade resurser (se nedan) behöver återskapas lokalt:

- Lägg till dina context-filer för respektive agent under `resources/Context_*/`
- Återskapa tenant- och appversionsinformation via Orchestrator

### 4. Publicera till Orchestrator

Öppna `PropertyInspection.uipx` i UiPath Studio och publicera de ingående processerna till din Orchestrator-tenant i rätt ordning:
1. `Analyze Inspection`
2. `Electricity - Agent`
3. `Water and Moisture - Agent`
4. `Main Flow`

---

## Exkluderade filer (.gitignore)

Följande filer och mappar är medvetet exkluderade från repot av säkerhets- och integritetsskäl:

| Sökväg | Anledning |
|--------|-----------|
| `resources/solution_folder/connection/` | Anslutningar kopplade till personliga konton |
| `resources/solution_folder/bucket/` | Storage bucket-konfiguration |
| `debug_overwrites.json` | Innehåller känslig tenant-information |
| `**/resources/Context_*/` | Kundspecifika kontextfiler för AI-agenterna |
| `resources/solution_folder/appVersion/` | Tenant-specifik appversionsinformation |

---

## Projektstruktur

```
PropertyInspection/
├── Analyze Inspection/        # Process för analys och kategorisering
├── Electricity - Agent/       # AI-agent för elskador
│   ├── agent.json
│   ├── evals/                 # Evalueringsset för agenten
│   └── resources/             # Kontextdokument och eskaleringsinstruktioner
├── Water and Moisture - Agent/ # AI-agent för vatten- och fuktskador
│   ├── agent.json
│   ├── evals/
│   └── resources/
├── Main Flow/                 # BPMN-orkestreringsflöde
├── resources/                 # Gemensamma resurser (lösningspaket)
├── SolutionStorage.json       # Projektregister
└── PropertyInspection.uipx    # UiPath-lösningsfil
```

---

*Utvecklat som del av LIA-projekt inom UiPath-automatisering.*
