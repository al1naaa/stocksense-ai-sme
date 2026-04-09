# IT4IT Framework and Reflective Summary

**Project:** StockSense AI - Smart Inventory and Demand Forecaster
**Client type:** Small and Medium Enterprise (SME), retail and B2B
**AI Model:** Google Gemini 2.0 Flash
**Deployment:** Firebase Hosting (Google Cloud)
**Interface:** Web browser - desktop and mobile

---

## Part 1 - IT4IT Value Stream Mapping

### 1. Strategy to Portfolio (S2P) - The Business Problem

**Problem statement**

Small and medium retail businesses across Kazakhstan face a recurring operational crisis: inventory is managed manually in Excel spreadsheets or basic POS systems. This creates two costly failure modes:

- Stockouts - running out of popular items and losing sales directly. A single missed sale of an iPhone 15 Pro at T 650,000 covers the AI operating cost for over 5 years.
- Overstock - capital is locked in slow-moving goods that occupy storage and lose value over time.

These problems are amplified by strong seasonal demand spikes that are hard to anticipate manually: Nowruz in March, summer electronics peak in June-August, back-to-school in September, Black Friday in November, and New Year in December.

**Why AI is the right investment**

Traditional solutions are inaccessible to SME:
- Hiring a supply chain analyst: from T 400,000 per month
- Enterprise ERP software (SAP, Oracle): from $20,000 implementation cost

StockSense AI costs under $2 per month in API fees (Gemini 2.0 Flash at $0.075 per million tokens). It delivers capabilities previously available only to large enterprises: natural language inventory analysis, seasonal demand forecasting, Economic Order Quantity calculations, and automatic stockout alerts.

**Portfolio fit**

StockSense AI is a zero-infrastructure B2B SaaS tool accessible from any web browser. Each business owner gets a private account with their own inventory stored in Firebase Firestore. No IT department required.

ROI is immediate and measurable: the smart alert system automatically calculates when stock is getting dangerous based on daily sales rate and supplier lead time - without any manual threshold setting from the user.

---

### 2. Requirement to Deploy (R2D) - Architecture and Development

**Solution architecture**

StockSense AI is a zero-backend Single Page Application. The entire product runs as a single HTML file:

- React 18 handles the UI layer (loaded via CDN, no build pipeline required)
- Chart.js 4 renders all data visualizations
- Google Gemini 2.0 Flash is called directly via REST API from the browser
- Firebase Authentication manages user accounts
- Cloud Firestore stores per-user inventory with real-time sync
- Firebase Hosting serves the file globally via CDN with HTTPS

**Key architectural decisions**

Gemini 2.0 Flash over GPT-4 or alternatives:
- 10x cheaper per token ($0.075/1M vs $0.75/1M for GPT-4o-mini)
- Sufficient reasoning for structured inventory analysis
- Native Russian language support
- Fast response times for real-time chat

Auto-calculated smart minimum stock (no manual input required):
- Formula: daily sales rate x (lead time days + 7 day buffer)
- Eliminates the need for users to understand inventory theory
- Updates automatically when sales data changes

Dynamic context injection over RAG or fine-tuning:
- Every Gemini API call receives the complete inventory JSON with pre-computed fields
- daysUntilStockout, status (CRITICAL/LOW/OK), eoqRecommended calculated in JavaScript before injection
- Eliminates arithmetic hallucination risk entirely

**Development process - Antigravity Orchestration**

The codebase was built through 5 directed architectural sessions using Claude as the AI agent. The architect defined system structure, data models, UI requirements, and system prompt strategy. The agent generated all implementation code. Full session records are in `logs/antigravity_session_log.md`.

---

### 3. Request to Fulfill (R2F) - How the SME Uses the Tool

**Access method:** Web browser on desktop or mobile. No app installation, no IT setup.
**URL:** https://stocksense-ai-sme.web.app

**User journey for a first-time business owner**

1. Owner opens the URL and creates an account with email and password
2. Dashboard immediately shows KPI cards - if any products are critically low, red alerts appear at the top
3. Owner goes to Inventory, clicks "Add Product", enters name, how many they have, how many they sell per month, and delivery time - the system calculates the alert threshold automatically
4. Owner types a question in the AI chat: "What should I order before Nowruz?" or in Russian: "Что нужно срочно заказать?"
5. Gemini responds in the same language with specific recommendations grounded in actual stock data
6. Owner navigates to Demand Forecast, selects a product, sees 6-month projection with seasonal peaks highlighted
7. Owner checks Analytics to review overall inventory health, sales ranking, and how much the AI has cost this session

**Why this works for SME**

- No onboarding or training required - every field has a plain-language label and hint
- Works on a phone during a warehouse walk
- AI responds in Russian or English depending on what the user writes
- All AI costs visible in the FinOps panel on the Analytics page

---

### 4. Detect to Correct (D2C) - Monitoring, FinOps, and Error Handling

**LLM cost monitoring (FinOps)**

The Analytics page includes a real-time session FinOps tracker showing:
- Number of AI questions asked in the current session
- Estimated tokens consumed (calculated as total characters divided by 4)
- Estimated session cost in USD
- Reference pricing table for Gemini 2.0 Flash

For production monitoring: Google AI Studio provides exact token counts, request history, and quota alerts. A monthly budget alert at $5 provides early warning before costs could escalate.

**Hallucination detection**

Primary safeguard is grounding: every Gemini API call includes the complete inventory JSON with pre-computed fields. The AI receives answers (daysUntilStockout, status, eoqRecommended), not raw numbers to calculate from.

Additional safeguards:
- Temperature set to 0.35 (below default) to reduce creative variation
- System prompt explicitly instructs the model to never invent data not present in the JSON
- Analytics page includes a visible accuracy note reminding users to cross-check AI stock numbers against the Inventory table
- When API quota is exceeded, a clear banner appears explaining exactly how to fix it (wait 24h, enable billing, or create a new project)

**Error handling**

- Quota exceeded: dedicated yellow banner with step-by-step fix instructions instead of a raw error code
- Firebase misconfiguration: 3.5 second timeout, then app runs in demo mode automatically
- Invalid credentials: mapped error codes to plain-language messages
- Chart rendering: every Chart.js instance stored in useRef, destroyed in useEffect cleanup before recreation
- Network failure: try/catch on all API calls with user-readable error in chat

---

## Part 2 - Reflective Summary

### What it meant to manage an AI agent rather than write code

Working as a Product Architect directing an AI agent changed the nature of the cognitive work involved in building software. The mental effort shifted from implementation details - "what is the correct React useEffect dependency array?" - to system design and failure anticipation - "what will this AI get wrong, and how do I explain a business constraint precisely enough that a language model applies it correctly?"

The most important realization was that specificity is the primary architectural skill. Vague prompts produced generic, often incorrect code. Precise technical direction produced correct code immediately. The difference between "add a chart" and "store the Chart.js instance in a useRef, call .destroy() in the useEffect cleanup before re-creating, and use the ref to track whether an instance already exists" is the difference between a bug and a working feature.

### Hardest architectural bottlenecks

**1. Gemini API format differences from OpenAI**

The agent defaulted to OpenAI API format (role: "assistant", no system_instruction field) because most of its training examples use OpenAI. Correcting this required the architect to know the Gemini API specification: system_instruction is a separate top-level field, and the assistant role is called "model" in Gemini's format. This could not be inferred from context.

**2. Smart minimum stock without user input**

The original design asked users to manually set a minimum stock threshold. The architectural insight - that this is a barrier for non-technical SME users and can be calculated automatically from daily sales rate and lead time - required rethinking the data model and the add product form entirely. The formula (daily sales x (lead time + 7)) is simple but the decision to implement it automatically rather than expose it to the user was an architectural judgment call the agent did not make unprompted.

**3. Pre-computing fields to eliminate arithmetic hallucinations**

Sending raw inventory fields and asking Gemini to calculate daysUntilStockout produced inconsistent results. The architectural decision to pre-compute this in JavaScript before injection - so the AI receives a pre-validated answer rather than raw data - required understanding why LLMs hallucinate on numerical reasoning and designing around that limitation proactively.

**4. Quota error user experience**

When the Gemini API returns a quota error, the raw error message is technical and unhelpful. Designing a separate QuotaBanner component with plain-language instructions (wait 24h, enable billing, create a new project) required anticipating this failure mode before it happened and building a graceful degradation path.

### What this means for software development

The AI agent paradigm does not eliminate the need for technical knowledge - it relocates where that knowledge matters. Implementation speed collapses from days to hours. The bottleneck becomes architectural thinking: understanding failure modes, designing around LLM limitations, and knowing when a technically correct approach is the wrong business decision.

For SMEs, this means tools that previously cost $50,000 to build can be prototyped in a weekend. The democratization is real - but it requires architects who can bridge business problems and technical constraints, not just people who can describe a feature in vague terms.
