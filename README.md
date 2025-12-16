# KK AI Translator

Et oversættelsesværktøj med taletransskription, AI-oversættelse og tekst-til-tale. Platformen består af en Python Flask API backend og en React frontend.

Platformen er udviklet til brug til hverdags tolkning/oversættelse. Løsningen er første version bygget i Kalundborg og baseret sig på modeller hosted i Azure (Whisper v3 og GPT4o). Whisper kan også køres lokalt og der kan bruges en lokal model som Mistral  (Kalundborg har en v2 som køre lokalt)

## Funktioner

- **Taletransskription** - Konverterer tale til tekst ved hjælp af Azure Speech Services eller Whisper
- **AI-oversættelse** - Oversætter mellem 90+ sprog ved hjælp af GPT-4o eller lokal LLM
- **Tekst-til-tale** - Stemmesyntese på 40+ sprog via Azure Cognitive Services
- **Sessionstyring** - Sporer samtalehistorik med sessioner
- **Sessionresumé** - Genererer sammenfatninger af samtaler på begge sprog
- **REST API** - Swagger/OpenAPI dokumentation er inkluderet
- **Docker support** - Multi-stage builds til API-only eller full-stack deployment

## Understøttede Sprog

15 sprog er aktiveret som standard:

| Sprog | Kode |
|----------|------|
| Arabic (Saudi Arabia) | ar-SA |
| Danish | da-DK |
| Dutch | nl-NL |
| English (UK) | en-GB |
| English (US) | en-US |
| French | fr-FR |
| German | de-DE |
| Italian | it-IT |
| Polish | pl-PL |
| Portuguese | pt-PT |
| Romanian | ro-RO |
| Spanish | es-ES |
| Swedish | sv-SE |
| Turkish | tr-TR |
| Ukrainian | uk-UA |

Yderligere sprog kan aktiveres via Indstillingssiden. Alle sprog fra Azure Speech Services stemmeliste er tilgængelige.

## Hurtig Start

### Forudsætninger

- Docker og Docker Compose
- Azure-konto med:
  - Cognitive Services (Speech)
  - OpenAI Service (GPT-modeller)
- Lokalt
Kræver hardware ala Nvidia L40s og køre bedst på en Linux maskine med vLLM

### 1. Klon og Konfigurer

```bash
git clone https://github.com/kalundborgkommune/KK-AI-Translator-V1-OS.git
cd KK-AI-Translator-V1-OS

# Kopiér eksempel-miljøfil
cp .env.example .env

# Rediger .env med dine Azure-legitimationsoplysninger (påkrævet)
nano .env
```

**Påkrævet i `.env`:**
```bash
AZURE_SPEECH_KEY=your_azure_speech_key
AZURE_SPEECH_REGION=westeurope
AZURE_OPENAI_KEY=your_azure_openai_key
MODEL_AZURE_GPT4OMINI_URL=https://your-resource.openai.azure.com/...
```

### 2. Kør med Docker Compose

```bash
docker-compose up -d
```

Dette starter oversætter-appen (API + React frontend) og en MySQL database.

Applikationen er tilgængelig på: **http://localhost:8080**

API-dokumentation: **http://localhost:8080/api/docs**

Indstillingsside: **http://localhost:8080/settings**

### 3. Kør Uden Docker

```bash
# Påkræver: Python 3.11+, Node.js 20+, MySQL database
make dev
```

Se [SETUP.md](SETUP.md) for manuelle opsætningsinstruktioner.

### 4. Docker Uden Compose

```bash
# Byg billedet
make build-full

# Kør (påkræver ekstern MySQL)
make run-full
```

## API Dokumentation

Swagger UI er tilgængelig på `/docs` når serveren kører.

### Endpoints

| Endpoint | Metode | Beskrivelse |
|----------|--------|-------------|
| `/api/v1/sessions/start-session` | POST | Opret en ny oversættelsessession |
| `/api/v1/sessions/select-language` | POST | Sæt kildesproget for en session |
| `/api/v1/sessions/translate` | POST | Transskriber og oversæt lyd/tekst |
| `/api/v1/sessions/transcribe` | POST | Transskriber kun lyd til tekst |
| `/api/v1/sessions/tts` | POST | Konverter tekst til tale |
| `/api/v1/sessions/recap` | GET | Få AI-sammenfatning af en session |
| `/api/v1/sessions/available-languages` | GET | List understøttede sprog |
| `/api/v1/languages/` | GET | List alle sprogindstillinger |
| `/api/v1/languages/seed` | POST | Seed sprog fra Azure stemmer |
| `/api/v1/languages/bulk` | PUT | Opdater sprogindstillinger i bulk |
| `/api/v1/misc/ping` | GET | Sundhedstjek |

### Autentificering

Alle API-forespørgsler kræver autentificering. Der er to muligheder:

1. **API Key Header** (til server-til-server kommunikation):
   ```
   x-api-key: your-api-key
   ```

2. **Bearer Token** (til Microsoft Entra ID integration):
   ```
   Authorization: Bearer <jwt-token>
   ```

## Projektstruktur

```
kk-ai-translator/
├── python-be/                 # Flask Backend
│   ├── src/
│   │   ├── app.py            # Applikationsindgangspunkt
│   │   ├── auth/             # Autentificeringsmiddleware
│   │   ├── config/           # Konfiguration & sprogmapping
│   │   ├── db/               # Database opsætning (SQLAlchemy)
│   │   ├── models/           # ORM modeller (Session, Translation)
│   │   ├── routes/           # API endpoints
│   │   └── services/         # Forretningslogik
│   └── tests/                # Pytest test suite
│
├── react-fe/                  # React Frontend
│   ├── src/
│   │   ├── components/       # UI komponenter
│   │   ├── hooks/            # Brugerdefinerede React hooks
│   │   └── context/          # React context providers
│   └── package.json
│
├── Dockerfile                 # Full-stack Docker build
├── docker-compose.yml         # Docker Compose til lokal udvikling
├── Makefile                   # Build & deployment automatisering
└── .env.example              # Miljøskabelon
```

## Sprogindstillinger

Via Indstillingssiden (`/settings`) kan administratorer:

1. Aktivere eller deaktivere sprog, der skal vises i oversætteren
2. Vælge TTS-stemmer fra de tilgængelige indstillinger per sprog
3. Konfigurere AI-modeller til transskription, oversættelse og sammenfatning per sprog

### Første Gangs Opsætning

Ved første opstart kan du klikke på "Seed Languages" i Indstillingssiden for at indlæse alle tilgængelige Azure stemmer i databasen. 15 almindelige sprog er aktiveret som standard.

## Konfiguration

Se [SETUP.md](SETUP.md) for detaljerede konfigurationsinstruktioner om:

- Database opsætning (MySQL)
- Azure Cognitive Services konfiguration
- Azure OpenAI deployment
- Docker deployment muligheder
- Produktionshensyn

## Udvikling

### Tests

```bash
make test
```

### Kodekvalitet

```bash
# Formater kode
black python-be/src

# Lint
flake8 python-be/src
```

## Deployment

### Azure Deployment

Der er to muligheder for at deploye til Azure:

| Mulighed | Beskrivelse |
|--------|----------|
| **Azure VM + Docker Compose** | Simpel opsætning med fuld kontrol |
| **Azure Web App + Azure MySQL** | Administreret løsning med auto-scaling |

**Deploy på Azure VM:**
```bash
# På din Azure VM (Ubuntu 22.04):
curl -fsSL https://get.docker.com | sh
git clone https://github.com/kalundborgkommune/KK-AI-Translator-V1-OS.git
cd KK-AI-Translator-V1-OS
cp .env.example .env  # Tilføj dine Azure-legitimationsoplysninger
docker compose up -d
```

**Deploy som Azure Web App:**
```bash
# Byg, push til Azure Container Registry, og deploy
make build-full
make push-full
make deploy-full
```

Se [SETUP.md](SETUP.md#azure-deployment) for detaljerede trin-for-trin instruktioner inklusive Azure MySQL opsætning, SSL-konfiguration og kontinuerlig deployment.

## Bidrag

Bidrag er velkomne. Se [CONTRIBUTING.md](CONTRIBUTING.md) for retningslinjer.

## Licens

Dette projekt er licenseret under MIT License. Se [LICENSE](LICENSE) filen for detaljer.

## Kommerciel Version

Dette er open source versionen (V1) af KK AI Translator. Kalundborg Kommune bruger en v2 som er lukket. V1 skal bruges under eget ansvar med de juridiske afklaringer der kræves for at bruge AI til sagsbehandling.
