# AI Automation Portfolio

A collection of AI-powered automation systems built with n8n, Groq, OpenAI, and modern data tools. Each project solves a real operational problem for a specific business — from real estate customer support to business intelligence to lead qualification.

---

## What I Build 👷‍♂️

I help businesses replace manual, repetitive processes with intelligent systems that run themselves. My focus is always on the full picture: the automation, the reasoning behind it, the edge cases, and the outcome for the business.

### AI Agents & Chatbots
- Conversational agents with memory and context awareness
- Slack-integrated BI assistants with live data tools
- Customer support bots with smart routing and escalation logic
- Lead qualification flows with scoring and automated outreach

### Business Process Automation
- End-to-end workflow automation with n8n
- Multi-step conditional logic and error handling
- CRM and helpdesk integration (Freshdesk, HubSpot)
- Automated notifications, summaries, and follow-up sequences

### Data & Operations
- AI-powered sentiment analysis and feedback classification
- Customer ticket triage and intelligent routing
- Google Sheets and Airtable data pipelines
- Webhook design and REST API integration

---

## Tech Stack 🔨⛏️

- **Automation:** n8n
- **AI/LLM:** Groq (Llama 3.1 70B, Llama 3.3 70B), OpenAI GPT-4, Claude, Gemini
- **Memory:** Redis (chat memory for conversational agents)
- **Vector Search:** Supabase (pgvector)
- **Helpdesk:** Freshdesk
- **Data:** Google Sheets, Airtable
- **Communication:** Slack, Gmail, Telegram, WhatsApp Business API
- **Finance APIs:** Finnhub (stock prices, company profiles, news)
- **Custom APIs:** Webhook design and REST API integration

---

## Projects 📁

Each folder contains the workflow JSON file or screenshots, a README with the problem, solution, and logic breakdown, and setup instructions.

| Project | Tools | Industry |
|---|---|---|
| [AI Lead Qualification Agent](./AI%20Lead%20Qualification%20Agent) | n8n, Groq, Gmail, Google Sheets, Slack | Sales |
| [Customer Feedback AI Analysis](./Customer%20Feedback%20AI%20Analysis) | n8n, Groq, Google Sheets | Operations |
| [Meridian BI Agent](./Meridian%20BI%20Agent) | n8n, Groq, Slack, Redis, Finnhub | Business Intelligence |
| [UrbanNest Customer Support Automation](./UrbanNest%20Customer%20Support%20Automation) | n8n, Groq, Freshdesk, Google Sheets, Gmail | Real Estate |

---

## Project Summaries

### 🤖 AI Lead Qualification Agent
An autonomous workflow that scores inbound leads 1–10 against your Ideal Customer Profile using Groq's Llama 3.1 70B, generates a personalised email response, logs everything to Google Sheets, and fires a Slack alert for hot leads (score 8+) — all without any manual review.

### 📊 Customer Feedback AI Analysis
A feedback processing pipeline that collects customer submissions via form, classifies sentiment (VERY_POSITIVE to VERY_NEGATIVE) using Groq AI, flags urgent negative feedback automatically, and stores structured results in Google Sheets for reporting and trend tracking.

### 📈 Meridian BI Agent
A Slack-integrated business intelligence assistant that listens for mentions, maintains conversation context via Redis memory, and responds with either direct knowledge-based answers or live financial data (stock prices, company profiles, news headlines) via Finnhub — using tools only when necessary.

### 🏠 UrbanNest Customer Support Automation
A full customer support triage system for a real estate e-commerce business. Incoming Freshdesk tickets are classified into 6 categories by AI, enriched with customer order history, routed to the right team, and resolved with either template-based or AI-generated responses — with smart escalation for complaints and manager requests.

---

## Using These Workflows

Each project folder contains everything you need to get it running:

- Workflow JSON file ready to import into n8n
- Setup documentation explaining how it works
- API and credential configuration guide
- Customisation instructions for your specific use case
- Troubleshooting notes from building it

---

## How to Get Started

1. Import the workflow JSON into your n8n instance
2. Add your credentials for the connected services
3. Update any IDs — Google Sheets, Supabase, API endpoints, Slack webhooks
4. Adjust the workflow logic and prompts to fit your use case
5. Test with sample data before going live
6. Deploy and monitor

---

## Why Work With Me 😁

### What You Can Expect
- Systems that are built to last
- Clear documentation so you always understand what was built and why
- Communication that is direct and honest throughout
- Solutions designed around your actual business problem, not just the tools

### Results I Focus On
- Significant reduction in repetitive work
- Faster response times to leads and customers
- Clean, organised data that teams can actually use
- Workflows that scale as your business grows

### How I Work
- I understand the problem before I touch any tools
- I build clean, maintainable workflows with proper error handling
- I document everything so your team can manage it without me
- I stay available after delivery to make sure things actually work

---

## License 🪪

MIT License — use, adapt, and build on anything here freely.

---

## Support

If any of these workflows are useful to you, a star on the repo goes a long way. And if you need something built, reach out directly.

---

## Let's Work Together 💪

I am available for freelance projects, contract work, and consulting on automation strategy.

- **LinkedIn:** [linkedin.com/in/benjaminowolabi](https://www.linkedin.com/in/benjaminowolabi)
- **Email:** owolabenjade@gmail.com
- **Twitter/X:** [@ademidowolabi](https://twitter.com/ademidowolabi)

Always building. Always learning.
