# 💍 EngagePlanner

A real-time, collaborative engagement activity planning web app — built as a single `index.html` file with a Gen-Z blue/white aesthetic. No build tools, no npm, no server required.

---

## ✨ Features

| Section | What it does |
|---|---|
| **Dashboard** | Live stats — total activities, upcoming events, budget summary, countdown timer |
| **Full Overview** | One-page scrollable view of every section with inline add buttons |
| **Activities** | Create, edit, and delete engagement activities with title, date, time, location, notes, and quarter tag |
| **By Quarter** | View activities grouped by Q1–Q4 of the current year |
| **Locations** | Track venues with address, capacity, and notes |
| **Car Seat Plan** | Assign people to car seats per vehicle (driver + passenger seats) |
| **Bed Plan** | Assign guests to rooms and beds for overnight stays |
| **Engagement Games** | Log planned games with descriptions and materials |
| **Organizing Committee** | Track committee members with role and contact info |
| **Grocery List** | Add grocery items with quantity and category |
| **Meal Plan** | Plan breakfast, lunch, dinner, and snacks per day |
| **Budget Tracker** | Set a total budget, track spending with a progress bar |
| **Expenses** | Log expenses by category (Staycation DP/Full, Food, Gas, Grocery, Other) |

---

## 🚀 Getting Started

### Prerequisites
- A modern browser (Chrome, Edge, Firefox, Safari)
- A Firebase project with **Cloud Firestore** enabled

### Run locally
Just open `index.html` directly in your browser — no web server needed.

```bash
# Option 1: double-click index.html in File Explorer
# Option 2: open via terminal
start index.html        # Windows
open index.html         # macOS
xdg-open index.html     # Linux
```

### Real-time sync (multi-device)
The app uses **Firebase Firestore** for real-time collaboration. All data is stored in a single Firestore document and every connected client stays in sync automatically.

---

## 🔥 Firebase Setup

### 1. Create a Firestore database
1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Select your project → **Build → Firestore Database**
3. Click **Create database** → choose **Start in production mode** → select a region → **Done**

### 2. Set Firestore security rules
Go to **Firestore Database → Rules** tab and publish:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

> ⚠️ This allows public access — suitable for a private team tool. Add authentication rules later if needed.

### 3. Firebase config (already hardcoded)
The Firebase project config is already embedded in `index.html`. To switch to a different Firebase project, find and update the `FB_FIRESTORE_CONFIG` object in the `<script>` block:

```js
const FB_FIRESTORE_CONFIG = {
  apiKey:            "your-api-key",
  authDomain:        "your-project.firebaseapp.com",
  projectId:         "your-project-id",
  storageBucket:     "your-project.firebasestorage.app",
  messagingSenderId: "your-sender-id",
  appId:             "your-app-id"
};
```

---

## 🗂️ Project Structure

```
Engagement Planning/
└── index.html      ← entire app (HTML + CSS + JS, ~3,600 lines)
└── README.md       ← this file
```

Everything lives in one self-contained file — no dependencies, no node_modules, no build step.

---

## 🏗️ Architecture

```
index.html
├── <head>
│   ├── Firebase App SDK (compat v10)
│   ├── Firebase Firestore SDK (compat v10)
│   └── <style> — all CSS (Inter + Space Grotesk fonts, CSS variables, layout)
│
├── <body>
│   ├── .sidebar — navigation, date/quarter display, countdown, sync status
│   ├── .main
│   │   ├── .topbar — page title + context action button
│   │   └── .pages — one .page div per section (Dashboard, Activities, Locations, etc.)
│   └── Modals — activity, location, car, bed, game, committee, grocery, meal, expense, set-date, budget
│
└── <script>
    ├── Firebase / Firestore init + onSnapshot listener
    ├── State management (loadState / saveState — localStorage + Firestore)
    ├── Date / Quarter helpers
    ├── Navigation (renderPage, sidebar active state)
    ├── Per-section render functions (renderActivities, renderLocations, etc.)
    ├── Modal open/close/save handlers
    ├── Budget & Expense logic
    ├── Countdown timer
    └── INIT block (boot from localStorage → connect Firestore)
```

---

## 💾 Data Model

All data is stored as a **single flat Firestore document**:

```
Collection: planner
Document:   engagement_planner
```

```js
{
  engagement: { name: string, date: string, time: string },
  budget:     { total: number, label: string },
  activities: [{ id, title, date, time, quarter, location, notes, color }],
  locations:  [{ id, name, address, capacity, notes }],
  cars:       [{ id, name, seats: [{ label, person }] }],
  beds:       [{ id, room, bed, person, notes }],
  games:      [{ id, name, description, materials }],
  committees: [{ id, name, role, contact }],
  grocery:    [{ id, item, qty, category, done }],
  meals:      [{ id, day, type, description }],
  expenses:   [{ id, category, description, date, amount }]
}
```

A mirror copy is also kept in `localStorage` (`engage_planner_v1`) for instant offline boot.

---

## 🎨 Design System

| Token | Value |
|---|---|
| Background | `#f0f4ff` |
| Surface | `#ffffff` |
| Accent (blue) | `#2563eb` |
| Text | `#1a1f36` |
| Muted | `#6b7280` |
| Success | `#059669` |
| Danger | `#dc2626` |
| Font (UI) | Inter |
| Font (headings) | Space Grotesk |
| Border radius | `12px` (cards), `8px` (inputs) |

---

## 🔄 Real-time Sync Flow

```
Open page
   │
   ├─ loadState() → read localStorage (instant paint)
   ├─ initDate() + updateCounts() + renderDashboard()
   │
   └─ initFirebase()
         │
         └─ _fbDocRef.onSnapshot(...)
               ├─ Document exists → merge into state → re-render
               └─ Document missing → seed with local state → save to Firestore

User action (add/edit/delete)
   │
   └─ saveState()
         ├─ localStorage.setItem(...)   ← offline cache
         └─ _fbDocRef.set(state)        ← Firestore write
               └─ onSnapshot fires on all clients → UI updates everywhere
```

---

## 📋 Changelog

### v1.0.0 — Initial release
- Full engagement planning UI (all sections listed above)
- Gen-Z blue/white aesthetic (Inter + Space Grotesk)
- Firebase Firestore real-time sync (hardcoded config)
- localStorage offline fallback
- Live countdown timer (days / hours / mins / secs)
- Budget tracker with progress bar
- Full Overview page
- Car Seat visual planner
- Bed assignment planner
- Organizing committee tracker
- Grocery list with done/undone toggle
- Meal planner (breakfast, lunch, dinner, snacks)
- Expense table (Staycation DP/Full, Food, Gas, Grocery auto-pull, Other)
- Quarter-based activity grouping (Q1–Q4)

---

## 📄 License

MIT — free to use, modify, and share.
