# Odyssey — AI-Powered Claim Journey

Odyssey is a healthcare claims dashboard and AI assistant designed to help members understand their claims and help representatives resolve issues. It brings claim status, costs, explanations, and next steps into one experience, with the goal of reducing avoidable member and provider calls.

The experience follows the JourneyAI design brief in [`ui/src/imports`](ui/src/imports): make the claim journey easy to follow and answer five questions:

> What happened? Why did it happen? Who owns the next step? What action is required? When will it be resolved?

## Presentation

[**Open the JourneyAI feature and UI presentation (PDF)**](ui/src/imports/Humana_JourneyAI_Features_UI.pdf)

The link opens the PDF file in the repository on GitHub, where you can preview or download it.

## App functionality

### Member dashboard

- **Overview:** see total claims, claims under review, completed claims, and estimated amounts owed.
- **Plan benefits:** view a quick summary of the member's plan benefits.
- **Claims:** browse claim cards with service, provider, date, status, costs, and issue indicators.
- **Issues:** find claims that need attention and open their details.
- **Claim details:** review the current issue, suggested action, owner, displayed resolution estimate, and a plain-language “What Happened” explanation.
- **Cost summary:** compare the amount billed, the amount paid by the plan, and the amount the member may owe.
- **Next steps:** follow the displayed authorization and review workflow, from provider submission through claim reprocessing.

### Representative experience

Switch to the representative view to access the claims work queue. The interface asks for the caller's name, the member's name, and the member's date of birth, then checks Release of Information (ROI) authorization through the call-assist backend.

After a successful check, the interface loads the authorized member's claims for review. Representatives can inspect claim issues and recommended actions, then clear the authorization when finished. This is a demo authorization workflow; it is not an account login or complete API access-control system.

### Odyssey AI assistant

The floating chat assistant routes questions to specialist agents for:

| Area | What it helps with |
| --- | --- |
| Claims | Claim status, denial explanations, potential issues, and resolution guidance |
| Benefits and coverage | Procedure coverage, prior authorization requirements, and plan rules |
| Compliance | ROI gaps and operational or compliance flags |

Chat retains conversation context through an in-memory session until the conversation is cleared or the server restarts.

Example questions from the app:

- “What does claim CLM000377 entail?”
- “Does CPT 29881 need prior auth on DSNP?”
- “What ROI gaps need attention?”

### Accessibility

The header includes controls for normal, large, and extra-large text, a high-contrast mode, and a simplified view. The design uses rounded cards, clear status indicators, and a green healthcare palette inspired by the imported Humana references.

## Design references and implementation status

The imported material provides the visual direction and proposed product experience:

- [JourneyAI design brief](ui/src/imports/pasted_text/humana-claims-refactor.md): dashboard structure, explanations, claim journeys, provider workflows, and accessibility requirements.
- [Feature and UI reference PDF](ui/src/imports/Humana_JourneyAI_Features_UI.pdf).
- [Mobile dashboard reference](ui/src/imports/Screenshot_2026-07-16_112128.png) and [mobile benefits reference](ui/src/imports/Screenshot_2026-07-16_112137.png).

These images are design references, not screenshots of the current Odyssey app.

The current interface supports **member and representative views**. The brief also proposes a dedicated provider view, advanced search/filter/sort controls, searchable and pinned claim selection, a detailed claim timeline, an agent activity feed, multilingual content, and voice narration. Those features are not currently implemented in the main dashboard. Member search and authorization-upload buttons are presentational controls without connected actions. Some claim explanations, workflow steps, and resolution estimates are fixed demo content rather than claim-specific live output.

## How it works

The React dashboard loads a bundled, precomputed claim dataset for a quick demo. It can also request dashboard data from the Flask API. Chat and representative ROI verification use the live backend.

The Python backend uses Google's Agent Development Kit (ADK) and Gemini through Vertex AI. A coordinating agent routes questions to claim, benefits, or compliance specialists. The claim pipeline summarizes claims, detects issues, researches coverage and authorization context, and produces resolution or escalation guidance. Its call-risk score is a deterministic heuristic, not a trained prediction model.

The backend reads six CSV tables: claims, members, providers, coverage rules, ROI authorizations, and compliance flags. Care-gap and Stars datasets in the folder belong to a separate use case and are not loaded by this app's data layer.

For the full agent design, see [ARCHITECTURE.md](ARCHITECTURE.md).

## Run locally

### Dashboard demo

Install Node.js and npm, then run:

```bash
cd ui
npm install
npm run dev
```

Open **http://localhost:5173**. The member dashboard can display the bundled data without the backend; AI chat and representative verification require the API server.

### Live AI backend

From the repository root, create a Python environment and install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

For live agent calls, use a Google Cloud project with Vertex AI enabled, access to the configured Gemini model, and application-default credentials. With the Google Cloud CLI installed, configure your own project:

```bash
gcloud auth application-default login
export GOOGLE_GENAI_USE_VERTEXAI=TRUE
export GOOGLE_CLOUD_PROJECT="your-google-cloud-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"
python3 -m odyssey.api_server
```

Keep this terminal running and start the UI in another terminal. Vite proxies `/api` requests to **http://localhost:5000**. Live dashboard generation runs agents for the member's claims and can take several minutes, depending on model latency and quota.

### Build and optional tools

Build the frontend:

```bash
cd ui
npm run build
```

From the repository root, check the data tools without model calls:

```bash
python3 -m odyssey.smoke_test
```

With live AI credentials configured, run a claim pipeline or regenerate the bundled dashboard data:

```bash
python3 -m odyssey.run CLM000377
python3 -m odyssey.precompute
```

Precomputation makes model calls and overwrites `ui/src/data/claims.json`.

## Project layout

| Folder | Purpose |
| --- | --- |
| `ui/` | React, TypeScript, Vite, and Tailwind dashboard |
| `ui/src/imports/` | JourneyAI design brief, reference images, and PDF |
| `ui/src/data/claims.json` | Bundled dashboard data |
| `odyssey/` | Flask API, ADK agents, data tools, and claim pipeline |
| `adk_apps/` | Standalone ADK entry points for claims, benefits, and compliance |
| `data/` | Structured CSVs and supporting demo material |

## Demo data

The included report labels the dataset as synthetic hackathon data. The demo data and design references are included with the project. Keep credentials and local environment files out of Git.
