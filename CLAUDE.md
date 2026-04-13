# ReMind — Claude Code Project Context

AI-powered voice assistant for Alzheimer's patients. Simple voice interface for patients, monitoring/management interface for caregivers.

## Project Structure

```
/backend    — Node.js + Express API (port 3000)
/mobile     — Expo React Native app
```

## Key Commands

### Backend
```bash
cd backend
npm install
cp .env.example .env   # fill in ANTHROPIC_API_KEY
npm run dev            # nodemon, hot reload
npm start              # production
```

### Mobile
```bash
cd mobile
npm install
npx expo start         # scan QR with Expo Go app
```

Update `mobile/src/constants/config.js` → `API_BASE_URL` to your machine's local IP when testing on a physical device.

## Environment Variables (backend/.env)

| Variable | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | Yes | Claude AI (claude-sonnet-4-6) |
| `OPENAI_API_KEY` | No | Whisper STT transcription (falls back to mock if absent) |
| `PORT` | No | Default: 3000 |

## Architecture

### Backend (`backend/src/`)
- `index.js` — Express server entry point, DB init, route mounting
- `db/database.js` — better-sqlite3 singleton, schema setup, seeds `demo-patient-1`
- `services/claude.js` — Claude API with prompt caching, context injection, state detection
- `services/notifications.js` — Expo push notification sender
- `routes/chat.js` — POST /api/chat → AI response + detected state
- `routes/transcribe.js` — POST /api/transcribe → Whisper STT
- `routes/sos.js` — POST /api/sos → push notifications to caregivers
- `routes/history.js` — GET /api/conversations/:patientId
- `routes/context.js` — GET/PUT /api/context/:patientId
- `routes/patients.js` — GET/POST /api/patients
- `routes/devices.js` — POST /api/devices/register

### Mobile (`mobile/src/`)
- `context/AppContext.js` — Global state (role, patientId), persisted to AsyncStorage
- `navigation/AppNavigator.js` — Role guard: no role → select, patient → PatientNavigator, caregiver → CaregiverNavigator
- `screens/patient/HomeScreen.js` — Core screen: mic button → record → transcribe → chat → TTS
- `screens/caregiver/ContextScreen.js` — Edit patient context (family, medications, routine, rules)
- `screens/caregiver/HistoryScreen.js` + `HistoryDetailScreen.js` — Conversation monitoring
- `screens/caregiver/AlertsScreen.js` — SOS event history
- `services/api.js` — Axios client for all backend endpoints
- `services/speech.js` — expo-speech TTS wrapper
- `services/notifications.js` — Expo push token registration

## API Contract

| Method | Path | Body / Params | Response |
|---|---|---|---|
| POST | `/api/chat` | `{patientId, message, conversationId?}` | `{response, conversationId, detectedState}` |
| POST | `/api/transcribe` | multipart `audio` field | `{transcript}` |
| POST | `/api/sos` | `{patientId}` | `{success, notificationsSent}` |
| GET | `/api/conversations/:patientId` | — | `[{id, startedAt, messages}]` |
| GET | `/api/context/:patientId` | — | context object |
| PUT | `/api/context/:patientId` | context fields | `{success}` |
| GET | `/api/patients` | — | `[{id, name, createdAt}]` |
| POST | `/api/patients` | `{name, age}` | `{patientId, name}` |
| POST | `/api/devices/register` | `{patientId, expoPushToken, role}` | `{success}` |

`detectedState` values: `"normal"` | `"confused"` | `"emergency"`

## Database (SQLite — `backend/remind.db`)

Tables: `patients`, `conversations`, `messages`, `patient_context`, `devices`, `sos_events`

Default seed: patient id `demo-patient-1`, name "Demo Patient"

## Team

- Nir Berkowitz, Meitar Barco, Raz Matzliach, Hadar Orbach
- Supervisor: Prof. Igor Rekhlin
