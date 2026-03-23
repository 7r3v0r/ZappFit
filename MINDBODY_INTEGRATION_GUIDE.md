# Mindbody Integration Guide for Zapp Fitness Website

## Overview

This guide covers how to connect your Zapp Fitness website to Mindbody so that your class schedule, bookings, and memberships sync automatically. There are two approaches — the simple widget embed (recommended to start) and the full API integration.

---

## Option 1: HealCode Widget (Quick Start)

HealCode is Mindbody's official embeddable widget system. It drops a live schedule, class sign-up, or pricing widget directly into your HTML with a single script tag. This is the fastest path to a working integration.

### Steps

**1. Log into your Mindbody business account**
Go to https://clients.mindbodyonline.com and sign in with your Zapp Fitness owner credentials.

**2. Navigate to the HealCode section**
From the Mindbody dashboard: **Home > Widgets > HealCode Widgets**. If you don't see this option, contact Mindbody support to enable it for your account (it's included on most plan tiers).

**3. Create a Schedule Widget**
Click "Create Widget" and select **Schedule Widget**. Configure it:
- Choose which class types to display (HIIT Burn, Strength, Conditioning, Total Body)
- Set the date range (typically "upcoming 7 days")
- Pick a color theme — choose the dark theme or custom CSS option so it blends with the site
- Enable "Book Now" buttons so visitors can sign up directly

**4. Copy the embed code**
HealCode will generate a snippet that looks like this:

```html
<healcode-widget
  data-type="schedules"
  data-widget-partner="object"
  data-widget-id="YOUR_WIDGET_ID"
  data-widget-version="1">
</healcode-widget>
<script src="https://widgets.healcode.com/javascripts/healcode.js" type="text/javascript"></script>
```

**5. Paste it into schedule.html**
Open `schedule.html` and find the `<div id="mindbody-widget">` container. Replace the placeholder loading animation with the HealCode snippet. The surrounding CSS will handle the dark theme container.

**6. Style overrides (optional)**
HealCode widgets accept custom CSS. Add overrides after the widget script to match the Zapp palette:

```css
.hc-widget { background: #111 !important; color: #f0f0f0 !important; font-family: 'Inter', sans-serif !important; }
.hc-widget a { color: #67da32 !important; }
.hc-widget .hc-button { background: #67da32 !important; color: #0a0a0a !important; }
```

---

## Option 2: Mindbody API (Full Control)

For deeper integration — pulling class data into your own custom UI, syncing memberships, or building a fully branded booking flow — use the Mindbody Public API.

### Prerequisites

- A Mindbody business account (Zapp Fitness)
- API access enabled (request via https://developers.mindbody.io)
- Your **Site ID** (found in Mindbody dashboard under Settings > Account)
- An **API Key** (generated from the Mindbody Developer Portal)

### Key API Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /public/v6/class/classes` | Fetch upcoming class schedule |
| `GET /public/v6/class/classschedules` | Get recurring schedule templates |
| `GET /public/v6/class/classdescriptions` | Get class type details |
| `POST /public/v6/class/addclienttoenrollment` | Book a client into a class |
| `GET /public/v6/staff/staff` | Fetch trainer profiles and bios |
| `GET /public/v6/sale/services` | Get pricing/membership options |

### Authentication Flow

All requests require two headers:

```
Api-Key: YOUR_API_KEY
SiteId: YOUR_SITE_ID
```

For client-facing actions (booking, purchasing), you also need a **user token**:

```
POST /public/v6/usertoken/issue
Body: { "Username": "client@email.com", "Password": "..." }
```

The returned token goes in the `Authorization` header for subsequent requests.

### Example: Fetching the Weekly Schedule

```javascript
const SITE_ID = 'YOUR_SITE_ID';
const API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.mindbodyonline.com/public/v6';

async function fetchSchedule() {
  const startDate = new Date().toISOString().split('T')[0];
  const endDate = new Date(Date.now() + 7 * 86400000).toISOString().split('T')[0];

  const response = await fetch(
    `${BASE_URL}/class/classes?StartDateTime=${startDate}&EndDateTime=${endDate}`,
    {
      headers: {
        'Api-Key': API_KEY,
        'SiteId': SITE_ID,
        'Content-Type': 'application/json'
      }
    }
  );

  const data = await response.json();

  // data.Classes is an array of class objects with:
  // - ClassDescription.Name (e.g. "HIIT Burn")
  // - StartDateTime / EndDateTime
  // - Staff.Name (coach name)
  // - Location.Name
  // - MaxCapacity / TotalBooked (availability)
  // - IsCanceled

  return data.Classes;
}
```

### Example: Rendering Classes into the Schedule Grid

```javascript
function renderSchedule(classes) {
  const grid = document.getElementById('schedule-grid');

  classes.forEach(cls => {
    const day = new Date(cls.StartDateTime).toLocaleDateString('en-US', { weekday: 'short' }).toUpperCase();
    const time = new Date(cls.StartDateTime).toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' });
    const spots = cls.MaxCapacity - cls.TotalBooked;

    // Find the matching cell in your grid and populate it
    const cell = grid.querySelector(`[data-day="${day}"][data-time="${time}"]`);
    if (cell) {
      cell.innerHTML = `
        <div class="class-name">${cls.ClassDescription.Name}</div>
        <div class="class-coach">${cls.Staff.Name}</div>
        <div class="class-spots">${spots} spots</div>
      `;
      cell.classList.add('has-class');
    }
  });
}
```

---

## Recommended Approach

**Start with HealCode widgets** (Option 1). You can have a working, bookable schedule on your site in under 30 minutes with zero backend code. The widget handles authentication, payments, and class management through Mindbody's existing system.

**Graduate to the API** (Option 2) only if you want to build a fully custom booking UI that matches the industrial techno aesthetic pixel-for-pixel, or if you need to pull trainer data, membership info, or attendance stats into your own system.

---

## Useful Links

- Mindbody Developer Portal: https://developers.mindbody.io
- HealCode Widget Builder: https://healcode.com (accessed through your Mindbody dashboard)
- API Reference: https://developers.mindbody.io/reference
- Mindbody Support: https://support.mindbodyonline.com

---

## Files Modified

| File | What to change |
|------|---------------|
| `schedule.html` | Replace the `#mindbody-widget` placeholder with your HealCode snippet or custom API code |
| `training.html` | Optionally pull real trainer bios via `GET /public/v6/staff/staff` |
| `pricing.html` | Optionally pull live pricing via `GET /public/v6/sale/services` |
