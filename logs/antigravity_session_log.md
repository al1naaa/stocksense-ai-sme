# StockSense AI - Antigravity Orchestration Session Log

**Project:** AI Smart Inventory and Demand Forecaster for SME
**Role:** Product Architect (zero manual coding)
**AI Agent:** Antigravity
**Total Sessions:** 5

---

## Session 1 - Problem Definition and Architecture Blueprint

### Architect Prompt:
"I need to design a B2B AI tool for a small retail business in Almaty, Kazakhstan. The problem: owners manually track inventory in Excel and constantly run out of stock or overstock, losing money. Design an architecture for an AI-powered inventory forecaster. Requirements: zero backend, deployable to Firebase, single HTML file if possible, AI model must be Google Gemini. Give me the full system architecture and tech stack recommendation."

### Agent Response (Summary):
Proposed Single Page Application architecture: React 18 via CDN (no Node.js build required), Chart.js for visualizations, Google Gemini 2.0 Flash via direct REST API from the browser.

Model cost comparison provided:
- Gemini Flash: $0.075/1M tokens
- Gemini Pro: $1.25/1M tokens
- GPT-4o-mini: $0.15/1M tokens

Agent recommended Flash for cost efficiency and adequate reasoning capability for structured inventory tasks. Also recommended Firebase Hosting free tier for static file serving.

### Architect Refinement Direction:
"Inventory data must reflect Kazakhstan context. Use Tenge (T) as currency. Build in seasonal demand factors: Nowruz peak in March, summer electronics peak June-August, back-to-school September, Black Friday November, New Year December. Define the data model."

### Agent Response:
Designed inventory item data model with fields: id, name, productCode, category, stock, minStock, maxStock, price (Tenge), sold30 (units sold last 30 days), leadDays (supplier delivery time).

Designed 8 sample SKUs across electronics, furniture, stationery, accessories with Kazakhstan-appropriate prices.

### Architectural Decisions Confirmed:
- React CDN + Gemini Flash + Firebase Hosting
- Firebase Authentication for user login
- Firestore for per-user inventory persistence
- 4 pages: Dashboard, Inventory, Forecast, Analytics
- Data model finalized

---

## Session 2 - System Prompt Engineering for Gemini

### Architect Prompt:
"Engineer the Gemini system prompt. Requirements: (1) analyze inventory JSON injected as context, (2) pre-computed fields only - no arithmetic from the AI, (3) Kazakhstan seasonal demand patterns, (4) respond in same language as user - English or Russian, (5) EOQ-based reorder recommendations, (6) never invent numbers not in the provided JSON. Write the complete system prompt and the buildSystemPrompt function."

### Agent Response:
Generated structured system prompt with:
- Role definition: StockSense AI, inventory advisor for SME
- Dynamic JSON injection at every API call with pre-computed fields
- Kazakhstan seasonal calendar with demand multipliers
- Behavioral rules: language matching, Tenge formatting, 200 word limit, anti-hallucination instruction
- Pre-computed fields: daysUntilStockout = stock/sold30*30, status = CRITICAL/LOW/OK, eoqRecommended, reorderPoint

### Bottleneck Identified and Resolved:
**Problem:** First version sent full message history including the initial AI greeting, causing Gemini to return 400 errors ("Invalid role in contents array").

**Architect direction:** "Gemini uses system_instruction as a separate top-level field - not as a message in the contents array. Filter contents to only include user and model roles. The initial greeting must not be sent as a conversation turn."

**Agent fix:** Rewrote callGemini() with explicit role mapping. User messages map to role "user", AI messages map to role "model". System prompt passed in system_instruction field. 400 errors resolved.

---

## Session 3 - UI Architecture and Firebase Integration

### Architect Prompt:
"Design the UI. Requirements: (1) clean white interface, extremely simple - must be understandable by someone who has never used software, (2) English only, no emojis, regular hyphens not em dashes, (3) KPI cards on dashboard with critical alerts in red immediately visible, (4) AI chat embedded in Dashboard, (5) Firebase Authentication with graceful fallback to demo mode if Firebase is misconfigured - app must not break, just skip auth and show empty inventory."

### Agent Response:
Designed CSS variable system with white/off-white surfaces, semantic colors (danger red, warning amber, success green, accent blue), DM Sans font family, 8/12px border radius system.

Built Firebase initialization as ES module with window._fbReady flag and fb-ready custom event. Implemented 3.5 second timeout: if authentication check is still pending, skip auth and render app with empty inventory.

### Architect Direction - Smart Minimum Stock:
"Remove the manual minimum stock threshold field entirely. SME owners do not know what EOQ means. Calculate it automatically: minimum = daily sales rate x (lead time + 7 day buffer). Show the calculated value in the UI as 'alert at X units' so users understand what the system is doing."

### Agent Response:
Implemented autoMinStock() function:
```
function autoMinStock(sold30, leadDays) {
  if (!sold30 || sold30 <= 0) return 5;
  const dailySales = sold30 / 30;
  return Math.max(3, Math.round(dailySales * (leadDays + 7)));
}
```
Removed minStock field from the add product form. Threshold now calculated live from sold30 and leadDays inputs. Preview shown in form: "System will alert when stock drops below X units (auto-calculated)". Table shows "alert at X" under each product's stock count.

### Bottleneck Identified and Resolved:
**Problem:** Chart.js instances not destroyed before component re-renders. Navigation away from Analytics page and back caused "Canvas already in use" error and blank chart.

**Architect direction:** "Store Chart.js instance in useRef. In the useEffect cleanup function (the return statement), call chartInstance.current.destroy(). Also call destroy at the start of the effect body before creating new instance."

**Agent fix:** Applied useRef + destroy pattern to all 3 chart components. Error resolved.

---

## Session 4 - Quota Error Handling and Analytics Redesign

### Architect Prompt:
"When the Gemini API quota is exceeded, the current app shows a raw error code in the chat. This is useless to an SME owner. Build a dedicated QuotaBanner component that appears when quota is exceeded, explains the problem in plain language, and gives three concrete options: wait 24 hours, enable billing, or create a new project for a fresh free tier quota. The banner must link directly to the relevant pages."

### Agent Response:
Built QuotaBanner component with yellow/amber styling. Detects quota error by checking for "quota" or "RESOURCE_EXHAUSTED" in the Gemini error message. Throws QUOTA_ERR sentinel string, caught in the calling component which sets quotaErr state and renders the banner. Three options with direct links to aistudio.google.com and console.cloud.google.com.

### Architect Prompt - Analytics Redesign:
"The analytics page has problems: (1) the doughnut chart for category values is unreadable - users cannot tell what they are looking at, (2) the health breakdown needs text descriptions explaining what each status means, not just a count, (3) the sales chart should be horizontal bar not vertical - product names are too long for vertical axis. Redesign the page to be self-explanatory."

### Agent Response:
Redesigned Analytics page:
- Category value chart replaced with horizontal progress bars showing category name, product count, T value, and percentage - no chart needed, everything is readable text
- Health breakdown cards now each have a label, number, color border, and one-line description (e.g. "Stock is critically low - order immediately")
- Sales ranking changed to horizontal bar chart (indexAxis: "y") so product names display cleanly
- Critical items section added: lists only CRITICAL items sorted by days-until-stockout, most urgent first

---

## Session 5 - Forecast Page Redesign and Final Polish

### Architect Prompt:
"The Forecast page is confusing. Users open it and have no idea what they are looking at. Add: (1) a clear explanation box at the top telling users how to read the page, (2) three summary cards showing stock right now, when to reorder (reorder point), and how much to order (EOQ), (3) a warning if any forecasted month will exceed current stock, (4) a month-by-month table with the seasonal reason written in plain English next to each month - not just a number."

### Agent Response:
Rebuilt Forecast page:
- Blue info banner at top: "The chart shows expected sales per month. The green line is your current stock. When bars rise above it, you may run out. Red = peak months, yellow = normal, blue = slow."
- Three summary cards: current stock with days remaining, reorder point with explanation, EOQ with cost in Tenge
- Alert banner when any forecasted month exceeds current stock
- Month table: month name, seasonal reason (e.g. "Nowruz peak (March 21-23)"), progress bar showing relative demand, units forecast, Peak/Normal/Low label
- Chart changed from line to bar with color coding (red bars for peak months, yellow for normal, blue for slow)

### Architect Final Direction:
"Remove Auto Parts from the category list. Remove the Gemini 2.0 Flash text label from the sidebar footer - keep only the green status dot. All text must use regular hyphens, never em dashes. No emojis anywhere in the app."

### Agent Response:
Applied all changes. Final verification checklist confirmed clean.

---

## Reflection on Agentic Architecture

### What worked well

Breaking the project into IT4IT phases before any code was written kept the architecture disciplined. The D2C requirement forced early thinking about quota errors and hallucination risk, which produced better design decisions than if those had been discovered during testing.

The autoMinStock insight - eliminating a manual input field entirely by computing it automatically - is the kind of UX decision that came from thinking about the actual SME user rather than the technical implementation. The agent would not have made this decision unprompted.

### Hardest bottlenecks

**Gemini API format** was the first major blocker. The agent defaulted to OpenAI format. Required explicit correction with the exact field name (system_instruction) and role value ("model" not "assistant").

**Quota error UX** required anticipating a failure mode before it occurred and building a specific component for it. Generic try/catch is not sufficient when the error message needs to teach a non-technical user what to do.

**Chart.js + React lifecycle** remains the most technically complex interaction. The agent required explicit direction about useRef, useEffect cleanup, and the order of operations (destroy before create). A vague "fix the chart bug" prompt produced incorrect solutions.

**Auto minimum stock** required rethinking what data the user actually needs to provide versus what can be derived. This is product design judgment, not implementation - the agent generates code to implement a decision but does not make the decision itself.

### Key takeaway

The architect role in agentic development is not about knowing syntax. It is about knowing what questions to ask before code is written, what failure modes to design for, and what the actual user needs versus what they would ask for if you let them specify every field themselves. The agent is fast and capable. The bottleneck is always the quality of the architectural brief.