# Kabun Farm Intelligence

An offline-first Progressive Web App for managing and logging daily activity on a small commercial vegetable farm in Malaysia. Built to work in the field with poor connectivity — actions are queued locally and synced to Google Sheets when back online.

---

## Features

**Beds & Crops**
- Track multiple numbered beds with active crops and day counts
- Watering alerts — bed cards flag beds not watered in 3+ days
- Tap a bed to see crop details, past crop history, and log actions
- Add new beds directly from the app
- Rename a bed (e.g. "Kangkong Row") or retire one (hidden from home, kept in records)

**Activity Logging**
- Log irrigation, pest control, harvest, and sowing per bed or whole farm
- Harvest marks crops as done and removes them from the active bed
- Optional harvest weight (kg) for yield tracking
- Sowing adds a new crop batch to the bed immediately
- Optional inputs/notes with formula picker (see Formulas)
- Optional cost per activity
- Pull-to-refresh: swipe down to sync latest data

**Sales**
- Log crop sales separately from harvest
- Fields: crop, quantity, unit (kg / ikat / bag), price per unit
- Total revenue auto-calculated
- No bed required — farmer thinks in crop + quantity, not per bed

**Formulas**
- Spray and fertigation recipes managed directly in the app
- Add, edit, and delete formulas from your phone — no Sheets access needed
- Single sprayer volume input (default 16L) recalculates all recipe doses instantly
- Formula picker in the log form — tap a recipe to auto-fill ingredients and amounts into inputs/notes

**Financial Summary**
- Revenue, cost, and net shown on the Activity tab
- Toggle between this week and this month

**Activity Log**
- All logs and sales merged into a single feed, grouped by date
- Filter by bed or activity type
- Most recent entries shown first

**Offline & Sync**
- All actions saved to a local queue first
- Syncs to Google Sheets via Apps Script on POST when online
- Cache-first Service Worker — app loads instantly with no signal
- Update banner appears when a new version deploys

---

## Stack

- Vanilla JS + CSS — no framework
- Service Worker (cache-first, offline queue)
- localStorage for data cache and offline queue
- Google Apps Script as backend (doGet / doPost)
- Google Sheets as database

---

## Google Sheets Structure

Create a Google Spreadsheet with these sheets and exact camelCase headers:

| Sheet | Headers |
|---|---|
| Beds | bedNumber, location, status, name |
| Batches | id, bedNumber, cropName, location, plantingDate, status, harvestDate |
| Logs | id, date, bedNumber, activityCategory, cropName, inputsUsed, costRM, revenueRM, weight |
| Formulas | id, name, category, description, recipe |
| Sales | id, date, crop, quantity, unit, pricePerUnit, totalRevenue |

**Formula recipe format:** pipe-separated `name:amount:unit` per ingredient
```
EM4:10:ml|Antracol:2:g
```
Amounts are per litre — the app multiplies by sprayer volume.

> **Note:** The `id` column in the Formulas sheet is required for add/edit/delete to work. Populate existing rows with unique values (e.g. `f1`, `f2`, ...).

---

## Deploying Your Own

1. Fork this repo
2. Set up the Google Spreadsheet with the sheets and headers above
3. Copy the Apps Script code into a new Apps Script project bound to the spreadsheet
4. Deploy the Apps Script as a web app (execute as me, anyone can access)
5. Copy the deployment URL into `app.js`:
   ```js
   const GOOGLE_SCRIPT_URL = "your_apps_script_url_here";
   ```
6. Enable GitHub Pages on the `master` branch
7. Open the app on your phone and add it to your home screen

---

## Apps Script Actions

**doGet**
- `?action=getBeds` — beds with active crops, crop history, and last activity per bed
- `?action=getLogs` — all logs sorted newest first
- `?action=getFormulas` — all formulas
- `?action=getSales` — all sales sorted newest first

**doPost**
- `addLog`, `addBed`, `updateBed`, `deleteBed`
- `addBatch`, `updateBatch`, `deleteLog`
- `addSale`, `deleteSale`
- `addFormula`, `updateFormula`, `deleteFormula`

---

## Context

Built for a single commercial vegetable farm in Malaysia. Units (ikat, bag) and workflows (end-of-day sales logging, spray formula calculator) are specific to this farm's operation. Harvest and sales are intentionally separate — the farmer harvests physically, then sells later, sometimes split across multiple buyers.
