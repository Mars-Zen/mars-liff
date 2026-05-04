# Prince Mars — Project Overview & Migration Guide

## What Is This Project?

**Prince Mars (星語守護者)** is a LINE chatbot platform for astrology and spiritual services. Users interact entirely through LINE (a messaging app popular in Taiwan/Asia). The system is 100% built on **n8n** — there is no traditional application codebase.

---

## How It Works (Architecture)

```
User sends LINE message
        ↓
LINE Messaging API → n8n Webhook (entry workflow)
        ↓
n8n routes the message to the appropriate workflow
        ↓
Workflow calls external APIs (Gemini AI, Weather, etc.)
        ↓
n8n writes/reads from internal Data Tables
        ↓
n8n sends reply back to user via LINE Reply API
```

Frontend (LIFF pages) are static HTML files hosted on GitHub Pages, opened inside LINE via LIFF (LINE Front-end Framework). They call n8n webhooks directly via HTTP POST.

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Automation engine | n8n (self-hosted) |
| Messaging | LINE Messaging API |
| Frontend | Static HTML/CSS/JS via LINE LIFF, hosted on GitHub Pages |
| AI | Google Gemini 2.5 Flash |
| Database | n8n built-in Data Tables |
| File storage | Google Cloud Storage (pet/character images) |
| Internal AI model | Local LLM service at `host.docker.internal:3000` (must run on same VM as n8n) |
| Astrology calculations | Internal Python API (`PY_ASTRO_API_KEY`) |

---

## n8n Workflows (45 total, 29 main workflows listed below)

### Core Entry
| Workflow | Description |
|----------|-------------|
| `Divination_set_test` | Main LINE Webhook entry point — routes messages by type |

### AI & Conversation
| Workflow | Description |
|----------|-------------|
| `Guardian-Agent` | Main conversation engine, calls Gemini API |

### User Setup
| Workflow | Description |
|----------|-------------|
| `Natal` | Receives birth data, calculates natal chart, writes user profile |

### Daily Scheduled Services
| Workflow | Description |
|----------|-------------|
| `Routine` | Daily trigger — sends today's fortune to users |
| `Daily QA` | Manages daily quiz (max 3/day), records interactions |
| `Daily Meal Advice` | Recommends food based on user's weakest element |

### On-Demand Services
| Workflow | Description |
|----------|-------------|
| `OnDemand Today Fortune_v2` | Generates personalized daily fortune |
| `OnDemand Daily Outfit` | Generates outfit recommendations |
| `OnDemand Solar Return_v2` | Calculates solar return chart |
| `OnDemand Solar Return_Line_send` | Pushes solar return report via LINE |
| `OnDemand Build Natal Basic Card` | Generates natal chart summary card |

### Energy System
| Workflow | Description |
|----------|-------------|
| `Energy_Decay_Monitor` | Monitors element energy decay, sends push notifications |
| `Figure` | Calculates and updates element decay values |

### Pet/Companion System
| Workflow | Description |
|----------|-------------|
| `Create Pet` | AI-generates companion creature image and personality |
| `Summon Pet` | Awakens/summons user's companion |
| `Pet Point` | Manages aura points and evolution logic |
| `Pet Redirection` | Routes pet-related messages |

### Group & Social Features
| Workflow | Description |
|----------|-------------|
| `API_Group_Create` | POST `/group-create` — create a star group |
| `API_Group_Join` | POST `/group-join` — join via invite code |
| `API_Group_Leave` | POST `/group-leave` — leave group (auto-deletes empty groups) |
| `API_Group_List` | POST `/group-list` — list user's groups |
| `API_Group_Members` | POST `/group-members` — get member data with LINE avatars |
| `API_Group_Rankings` | POST `/group-rankings` — weekly leaderboard |
| `API_Group_Rename` | POST `/group-rename` — rename group |
| `API_Journal` | POST `/journal-*` — personal mood journal + group diary |
| `API_Battle` | POST `/battle-*` — quiz battle game (7 endpoints) |
| `Count Image` | Counts weekly image uploads per group member |

### Analytics & Utilities
| Workflow | Description |
|----------|-------------|
| `Waveform Chart` | Generates energy waveform chart |
| `WF_Daily_User_Interaction_Analysis` | Daily interaction analysis summary |

---

## LIFF Frontend Pages (9 pages, GitHub Pages)

Repository: `https://github.com/Mars-Zen/mars-liff`  
Live URL: `https://mars-zen.github.io/mars-liff/`

| File | Function |
|------|----------|
| `index.html` | User registration (birth data, name, gender) |
| `group_home.html` | Star group home — list groups, create/join |
| `group_room.html` | Group room — members, rankings, journal, quiz battle |
| `quiz_battle.html` | Quiz battle game interface |
| `daily_QA.html` | Daily quiz interface |
| `quiz.html` | Quiz game |
| `aura.html` | Element energy visualization |
| `encyclopedia.html` | Astrology knowledge reference |
| `achievement.html` | Achievement celebration page |

All pages call n8n webhooks at:  
`https://webbermars.app.n8n.cloud/webhook/` ← **this URL must be updated after migration**

---

## n8n Data Tables (16 total)

These are n8n's built-in Data Tables (stored internally in n8n, not external DB).  
They must be **manually recreated** in the new n8n instance after migration.

| Table | ID (current instance) | Purpose |
|-------|----------------------|---------|
| `elements_state_v1` | `avAjv7LIuE54URAe` | Core user profiles, natal chart, element scores |
| `users_pet` | `QdHYEVOmSKSAyDXO` | User companion creature data |
| `star_groups` | `18r6DFIidiDMTMBJ` | Star group records |
| `star_group_members` | `HQafcxn3Rk0CMX7n` | Group membership + weekly stats |
| `mood_journal` | — | Personal mood journal entries |
| `quiz_battles` | `Z4jKM6rK1ZR3lRIx` | Battle room records |
| `quiz_battle_players` | `1SmyUt20MUixZyhV` | Battle participants |
| `quiz_battle_answers` | `jnriabkOQyveUlJ3` | Battle answer submissions |
| `chat_sessions` | — | Full conversation history (90k+ rows) |
| `daily_user_digest` | — | Daily interaction analysis |
| `qa_logs` | — | Daily quiz logs |
| `quiz_bank` | — | Quiz question bank |
| `outfit_lookbook` | — | Outfit recommendation database |
| `solar_return_yearly` | — | Annual solar return data |

> Note: Table IDs are hardcoded in the workflow JSONs. After migration, new IDs will be assigned and workflows must be updated accordingly.

---

## Credentials & Variables Required

### n8n Credentials (managed by MIS/admin)
| Credential | Used In |
|------------|---------|
| LINE Channel Access Token | All LINE reply/push workflows |
| Google Gemini API | Guardian-Agent, Natal, OnDemand Fortune, etc. |
| OpenWeatherMap API | OnDemand Daily Outfit |
| Google Cloud Storage OAuth2 | Figure workflow (image storage) |

### n8n Variables (managed by MIS/admin)
| Variable | Used In |
|----------|---------|
| `GEMINI_API_KEY` | Create Pet (direct HTTP call to Gemini) |
| `PY_ASTRO_API_KEY` | Natal, OnDemand Solar Return, OnDemand Fortune, etc. |

### Internal Services (must run on same VM as n8n)
| Service | Description |
|---------|-------------|
| Local LLM at `host.docker.internal:3000` | Internal AI model (`gpt-oss:20b`) used by several workflows |
| Python Astrology API | Internal astrology calculation service |

---

## Migration Steps Summary

### Infrastructure person's responsibility:
1. Provision a VM on GCP
2. Install Docker and run n8n via Docker Compose
3. Set up domain and HTTPS (LINE requires HTTPS, no plain IP)
4. Provide: new n8n URL + admin login credentials

### Developer's responsibility (after infra is ready):
1. Export all 29 workflow JSONs from current n8n Cloud
2. Import into new n8n instance
3. Recreate all Data Tables and re-export/import data from CSV backups
4. Update Data Table IDs in all workflows (they will change)
5. Set up Credentials and Variables (or hand off to MIS)
6. Update LINE Developers Console Webhook URL to new n8n URL
7. Update API base URL in all HTML files from old to new n8n domain
8. Test all critical flows

---

## Important Notes

- The LINE bot Channel is currently registered under a personal LINE account — ownership should be transferred to a company account before going live
- The local LLM service (`host.docker.internal:3000`) must be running on the **same VM** as n8n, or the URL must be updated in affected workflows
- n8n Cloud instance (`webbermars.app.n8n.cloud`) can remain active in parallel during migration for safety
