# StockSense AI

AI-powered inventory management and demand forecasting tool for small and medium businesses.
Built with Google Gemini Flash and deployed on Firebase.

https://stocksense-ai-sme.web.app

---

## What it does

StockSense AI helps SME owners manage their stock without spreadsheets. It automatically alerts when items are running low, forecasts demand for the next 6 months based on seasonal patterns, and answers plain-language questions about inventory in English or Russian.

**Pages:**
- **Dashboard** - KPI overview, critical stock alerts, AI chat assistant
- **Inventory** - Add, edit, delete products. Auto-calculated reorder thresholds
- **Demand Forecast** - 6-month projection per product with seasonal Kazakhstan factors (Nowruz, summer, Black Friday, New Year)
- **Analytics** - Inventory health, sales ranking, category value breakdown, AI cost monitoring

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React 18 via CDN (no build step) |
| AI | Google Gemini Flash |
| Auth + Database | Firebase Authentication + Firestore |
| Hosting | Firebase Hosting |

---

## Project structure

```
stocksense-ai/
- index.html                          Main application
- firebase.json                       Firebase Hosting config
- .firebaserc                         Firebase project reference
- seed_demo_data.js                   Demo data script for presentations
- README.md                           This file
- docs/
  - IT4IT_Summary.md                  IT4IT framework mapping + reflection
- logs/
  - antigravity_session_log.md        Architect session logs
```