# Automated Phone Receptionist

> Intelligent AI phone receptionist built on n8n — answers inbound calls, handles FAQs, and books appointments to Google Calendar. No human required.

---

## stack

| service | role | cost |
|---|---|---|
| [Twilio](https://twilio.com) | Inbound phone number + call routing | ~$1/mo per number |
| [ElevenLabs](https://elevenlabs.io) | Voice AI + natural conversation | Free tier / paid for production |
| [n8n](https://n8n.io) | Automation backbone + agent logic | Self-host free / cloud from $20/mo |
| [Claude 3.5 Sonnet](https://console.anthropic.com) | Scheduling AI + decision-making | ~$0.003 per 1K tokens |
| [Google Calendar](https://calendar.google.com) | Appointment storage + invites | Free |
| [Redis](https://upstash.com) | Conversation memory per session | Free tier available |

---

## how it works

```
caller dials Twilio number
  └─► Twilio routes call → ElevenLabs Conversational AI agent
        └─► ElevenLabs handles small talk + FAQs         (< 1s)
              └─► calendar action needed?
                    └─► HTTP POST → n8n webhook
                          └─► Claude 3.5 Sonnet reasons over request
                                ├─► Redis            (loads session memory)
                                ├─► Google Calendar  (checks availability)
                                └─► Google Calendar  (creates event + emails invite)
                                      └─► n8n returns response → ElevenLabs speaks it
```

---

## performance

| response type | latency | notes |
|---|---|---|
| ElevenLabs-only (greetings, FAQs) | `< 1s` | never touches n8n |
| n8n tool calls (calendar check / create) | `2–4s` | normal operating range |
| Redis disconnected | `5s+` | **unusable** — configure Redis first |

> **Redis is essential.** Without it, Claude must reparse the entire conversation transcript on every turn, causing severe lag and broken multi-turn flows.

---

## workflow nodes

| node | type | purpose |
|---|---|---|
| `Webhook: Receive User Request` | Webhook | Receives POST from ElevenLabs when a calendar action is needed |
| `Webhook: Return AI Response` | Respond to Webhook | Sends agent reply back to ElevenLabs as HTTP response |
| `Voice AI Agent` | LangChain Agent | Core Claude-powered agent — processes requests and decides actions |
| `Anthropic Chat Model` | LangChain LLM | Powers agent with Claude 3.5 Sonnet |
| `Redis Chat Memory` | LangChain Memory | Maintains conversation context across turns via session ID |
| `Reasoning Tool (Think)` | LangChain Tool | Forces agent to reason before acting — **do not remove** |
| `Calendar: Check Availability` | Google Calendar Tool | Queries open slots in a given time range |
| `Calendar: Create Appointment` | Google Calendar Tool | Creates event + sends calendar invite to caller |
| `Update an event` | Google Calendar Tool | Modifies an existing booking (reschedule or update) |

---

## setup

Follow these steps in order. Each section specifies exactly what to copy and where to paste it in n8n.

### step 1 — twilio

1. Sign up at [twilio.com](https://twilio.com) and verify your email and phone number
2. From the Console Dashboard, copy your **Account SID** and **Auth Token**
3. Go to **Phone Numbers → Manage → Buy a Number** — filter by Voice capability and purchase
4. No Twilio credential is needed in n8n — Twilio simply forwards calls to ElevenLabs

### step 2 — elevenlabs

1. Create an account at [elevenlabs.io](https://elevenlabs.io) (free tier works for testing)
2. Go to **Profile → API Keys → Create API Key** — name it `n8n Receptionist` and copy immediately
3. Navigate to **Products → Conversational AI → Create Agent**
4. Choose a voice, configure your agent persona and FAQ knowledge
5. In **Agent Settings → Tools / Client Tools**, add three tools pointing to your n8n webhook URL:

```
check_availability     POST  https://your-n8n-domain.com/webhook/your-path
create_appointment     POST  https://your-n8n-domain.com/webhook/your-path
update_appointment     POST  https://your-n8n-domain.com/webhook/your-path
```

6. Copy your **Agent ID** from the agent URL — needed in step 3

> No ElevenLabs credential node in n8n — ElevenLabs calls n8n, not the reverse. Your webhook URL is the integration point.

### step 3 — connect twilio → elevenlabs

**Option A (recommended):**
1. In ElevenLabs, open your agent → **Telephony / Twilio Integration** tab
2. Enter your Twilio Account SID and Auth Token → click **Connect**
3. Select your Twilio phone number from the dropdown — ElevenLabs configures the webhook automatically

**Option B (manual):**
1. In Twilio Console → **Phone Numbers → Your Number → Voice Configuration**
2. Set `A call comes in` → **Webhook** → paste your ElevenLabs agent's Twilio-compatible URL

### step 4 — anthropic (claude 3.5 sonnet)

1. Sign up or log in at [console.anthropic.com](https://console.anthropic.com)
2. Go to **API Keys → Create Key** — name it `n8n Phone Receptionist`
3. Copy the key immediately (starts with `sk-ant-` — shown only once)
4. Add a payment method under **Billing** (Claude 3.5 Sonnet ≈ $3 per 1M input tokens; typical calls use only a few thousand)

```
n8n → Credentials → New → Anthropic
  API Key: sk-ant-...
```

> The workflow is pre-configured to use `claude-3-5-sonnet-20241022`.

### step 5 — google calendar (oauth2)

1. Go to [console.cloud.google.com](https://console.cloud.google.com) → create a new project (e.g. `n8n Phone Receptionist`)
2. **APIs & Services → Library** → search `Google Calendar API` → **Enable**
3. **APIs & Services → OAuth Consent Screen** → External → fill in app name and emails → add scope:

```
https://www.googleapis.com/auth/calendar
```

4. **APIs & Services → Credentials → Create Credentials → OAuth Client ID** → Web Application
5. Under **Authorized Redirect URIs**, add:

```
https://your-n8n-domain.com/rest/oauth2-credential/callback
```

6. Copy the **Client ID** and **Client Secret**

```
n8n → Credentials → New → Google Calendar OAuth2 API
  Client ID:     ...
  Client Secret: ...
  → Connect → authorize with your Google account
```

7. Find your **Calendar ID**: Google Calendar → three dots next to your calendar → **Settings → Integrate calendar**
   - Format: `xxx@group.calendar.google.com` (or your email for the primary calendar)
8. Paste this ID into all three calendar nodes: `Check Availability`, `Create Appointment`, `Update an event`

### step 6 — redis (conversation memory)

1. Sign up at [upstash.com](https://upstash.com) → **Create Database** → choose the region closest to your n8n server → leave TLS enabled
2. Copy the **host**, **port**, and **password** from the Upstash dashboard

```
n8n → Credentials → New → Redis
  Host:     content-calf-12345.upstash.io
  Port:     6379
  Password: ...
  TLS:      enabled
```

3. Click **Test Connection** before proceeding — a failed connection means no memory and broken conversations
4. The `Redis Chat Memory` node uses ElevenLabs `sessionId` as the session key and stores 20 messages per call — adjust `contextWindowLength` if needed

> Alternatives: [Redis Cloud](https://redis.com/try-free) or self-hosted Redis.

---

## workflow configuration

### 1 — import

1. In n8n go to **Workflows → Import from File**
2. Upload `Automated_Phone_Receptionist.json`

### 2 — set webhook path

Open `Webhook: Receive User Request` and replace `REPLACE ME` in the **Path** field:

```
/elevenlabs-voice-scheduler
```

Full URL:

```
https://your-n8n-domain.com/webhook/elevenlabs-voice-scheduler
```

> Use a non-obvious path — treat it as a secret token. Never use `/webhook` or `/schedule` alone.

### 3 — set calendar IDs

Open each of the three calendar nodes and replace `REPLACE ME` with your Calendar ID:

- `Calendar: Check Availability`
- `Calendar: Create Appointment`
- `Update an event in Google Calendar`

### 4 — fill prompt placeholders

Open `Voice AI Agent` and replace every `[PLACEHOLDER]` in the system prompt. Never modify `{{ $json.body.* }}` expressions.

| placeholder | purpose | example |
|---|---|---|
| `[TIMEZONE]` | Your local timezone | `America/New_York` |
| `[APPOINTMENT_DURATION]` | Slot length in minutes | `60` |
| `[START_TIME]` | Opening time | `9am` |
| `[END_TIME]` | Closing time | `5pm` |
| `[OPERATING_DAYS]` | Days you accept bookings | `Monday through Friday` |
| `[BLOCKED_DAYS]` | Days never available | `Saturday and Sunday` |
| `[MINIMUM_LEAD_TIME]` | Min minutes before a booking | `60` |
| `[REQUIRED_FIELDS]` | Fields to collect | `email, address` |
| `[REQUIRED_FIELDS_NATURAL_LANGUAGE]` | How agent asks for them | `your email and service address` |
| `[REQUIRED_FIELDS_LIST]` | Comma-separated list | `email, address, phone` |
| `[SERVICE_TYPE]` | Name of your service | `Plumbing Service` |
| `[PRIMARY_IDENTIFIER]` | Appointment title suffix | `[Customer Name]` |
| `[REQUIRED_NOTES_FIELDS]` | What to store in event notes | `Problem description, email, address, phone` |

### 5 — activate and test

1. **Save** → toggle workflow to **Active**
2. Copy your webhook URL from the webhook node
3. Paste it into all three ElevenLabs tool configurations (`check`, `create`, `update`)
4. Use ElevenLabs's built-in webhook tester to verify n8n responds
5. Make a live test call to your Twilio number and walk through a full booking

---

## troubleshooting

| symptom | fix |
|---|---|
| Slow responses (5s+) | Redis not connected — check credentials, click **Test Connection** |
| `REPLACE ME` errors on execution | Search all nodes for `REPLACE ME` — a Calendar ID or path is still unset |
| Webhook returns 404 | Workflow is not Active — toggle the Active switch |
| Agent books at wrong times | `[TIMEZONE]` in prompt must exactly match your Google Calendar timezone |
| Caller receives no invite | Confirm `Create Appointment` node includes customer email in attendees field |
| Agent asks same question twice | Redis session ID mismatch — confirm ElevenLabs sends consistent `sessionId` |
| Google Calendar auth fails | Re-authorise Google OAuth2 in n8n Credentials (tokens expire after 1hr without use) |
| ElevenLabs webhook times out | n8n must respond within 4–5s — ensure Redis is fast and Claude API latency is normal |

---

## best practices

### voice prompt design

- Keep every agent response to **one short sentence** — callers lose the thread with longer answers
- Spell out emails and numbers with hyphens so ElevenLabs vocalises them clearly (`j-o-h-n at gmail dot com`)
- Always end with `"Is there anything else I can help you with?"` to signal turn completion
- Run a full test call before going live — voice AI behaves differently to text chat

### security

- Use a long random webhook path — treat it as a secret token
- Enable n8n webhook authentication if your instance is publicly accessible
- Never log full call transcripts without a clear data retention policy
- Rotate your Anthropic API key every 90 days

### scaling

- For high call volume, upgrade Redis to a dedicated instance (not shared free tier)
- n8n Cloud handles concurrency automatically; self-hosted may need queue mode enabled
- Monitor Anthropic API usage weekly — set a spending cap in the Anthropic console

---

## references

| resource | url |
|---|---|
| n8n Docs | https://docs.n8n.io |
| ElevenLabs Conversational AI | https://elevenlabs.io/docs/conversational-ai |
| Twilio Voice Quickstart | https://www.twilio.com/docs/voice/quickstart |
| Anthropic Console | https://console.anthropic.com |
| Google Calendar API Setup | https://developers.google.com/calendar/api/quickstart |
| Upstash Redis (free) | https://upstash.com |