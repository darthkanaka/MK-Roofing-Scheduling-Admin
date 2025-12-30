# Murakami Roofing - Availability Manager

A simple admin portal to manage inspector availability and special dates.

## Features

- 📅 **Calendar View** - See all available dates with slot status at a glance
- ⭐ **Special Dates** - Add extra availability outside Mon/Wed/Fri
- 🗑️ **Remove Dates** - Easy removal of special dates
- 🔄 **Real-time** - Connected directly to your Supabase database

## Deployment

### Option 1: Vercel (Recommended)

1. Push this repo to GitHub
2. Go to [vercel.com](https://vercel.com) and sign in with GitHub
3. Click "New Project" → Import your repo
4. Click "Deploy"
5. Done! You'll get a URL like `https://murakami-portal.vercel.app`

### Option 2: Netlify

1. Push this repo to GitHub
2. Go to [netlify.com](https://netlify.com) and sign in with GitHub
3. Click "Add new site" → "Import an existing project"
4. Select your repo
5. Click "Deploy"
6. Done! You'll get a URL like `https://murakami-portal.netlify.app`

### Option 3: GitHub Pages

1. Go to your repo Settings → Pages
2. Under "Source", select "Deploy from a branch"
3. Select `main` branch and `/ (root)` folder
4. Click Save
5. Your site will be at `https://yourusername.github.io/repo-name`

## n8n Workflow Update

After deploying, update your n8n workflow to pull special dates from Supabase instead of hardcoded values.

Add a new **Supabase node** before the "Aggregate by date" node:
- **Operation**: Get All Rows
- **Table**: `special_dates`

Then update the **"Aggregate by date"** Code node to use the fetched dates:

```javascript
const events = $input.all();

// Get special dates from Supabase
const specialDatesData = $('Get Special Dates').all();
const specialDates = specialDatesData.map(item => item.json.date);

// Generate all Mon/Wed/Fri dates for next 6 weeks
const daysOfWeek = [1, 3, 5];
const weeksAhead = 6;
const allDates = {};
const today = new Date();

for (let i = 0; i < weeksAhead * 7; i++) {
  const date = new Date(today);
  date.setDate(date.getDate() + i);
  const dateString = date.toISOString().split('T')[0];
  
  if (daysOfWeek.includes(date.getDay()) || specialDates.includes(dateString)) {
    allDates[dateString] = {
      date: dateString,
      quota_used: 0,
      available_slots: 3
    };
  }
}

// Now count actual appointments
events.forEach(event => {
  const dateString = event.json.start.dateTime.split('T')[0];
  
  if (allDates[dateString]) {
    allDates[dateString].quota_used++;
    allDates[dateString].available_slots = 3 - allDates[dateString].quota_used;
  }
});

// Convert to array
return Object.values(allDates).map(item => ({ json: item }));
```

## Configuration

The Supabase credentials are already configured in `index.html`. If you need to update them:

```javascript
const SUPABASE_URL = 'https://buasiiuvzxpbzrpqlnfy.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key-here';
```

## Security

The anon key is safe to use client-side. Row Level Security (RLS) is enabled on the tables to control access.
