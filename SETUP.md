# DRIVE.TRACK — Setup Guide

## What you're deploying
- `index.html` — Driver page (phone)
- `manager.html` — Manager dashboard (laptop)

---

## Step 1 — Create Supabase (5 minutes)

1. Go to **https://supabase.com** → Sign up free
2. Click **New Project** → name it `drivetrack`
3. Wait ~1 minute for it to set up
4. Go to **SQL Editor** (left sidebar) → paste this and click Run:

```sql
-- Trips table
create table trips (
  id uuid default gen_random_uuid() primary key,
  driver_name text not null,
  started_at timestamptz not null,
  ended_at timestamptz,
  total_km numeric(8,2) default 0,
  max_speed_kmh integer default 0,
  avg_speed_kmh numeric(6,1) default 0,
  status text default 'active',
  created_at timestamptz default now()
);

-- GPS points table
create table gps_points (
  id uuid default gen_random_uuid() primary key,
  trip_id uuid references trips(id) on delete cascade,
  lat double precision not null,
  lng double precision not null,
  speed_kmh integer,
  accuracy numeric(6,1),
  recorded_at timestamptz not null
);

-- Enable real-time
alter publication supabase_realtime add table trips;
alter publication supabase_realtime add table gps_points;

-- Allow public access (anonymous read/write for this app)
alter table trips enable row level security;
alter table gps_points enable row level security;

create policy "allow all trips" on trips for all using (true) with check (true);
create policy "allow all gps_points" on gps_points for all using (true) with check (true);
```

5. Go to **Project Settings → API**
6. Copy your **Project URL** and **anon public key**

---

## Step 2 — Add your keys to the code

Open both `index.html` and `manager.html`.

Find this section in each file:
```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace with your actual values:
```js
const SUPABASE_URL = 'https://gczldnidvytkppbbwvyz.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imdjemxkbmlkdnl0a3BwYmJ3dnl6Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODExMjI4ODUsImV4cCI6MjA5NjY5ODg4NX0.xe5feNnhX-CEIasF1ms2J_wH88_Le2pO49sz1q1joOY';
```

---

## Step 3 — Deploy to Vercel (3 minutes)

1. Go to **https://github.com** → Create a new repository called `drivetrack`
2. Upload your `public/` folder (both HTML files)
3. Go to **https://vercel.com** → Sign up → New Project
4. Import your GitHub repo
5. Set **Root Directory** to `public`
6. Click Deploy

Your URLs will be:
- **Driver:** `https://drivetrack.vercel.app/index.html`
- **Manager:** `https://drivetrack.vercel.app/manager.html`

---

## Step 4 — Give to drivers

Send each driver their link (or a bookmark):
```
https://drivetrack.vercel.app
```

They open it → type their name → press Start.

You watch them live at:
```
https://drivetrack.vercel.app/manager.html
```

---

## How it works daily

| Who | What they do |
|-----|-------------|
| Driver | Opens link on phone, types name, taps Start |
| Driver | Drives all day — GPS saves every ~15 seconds |
| Driver | Taps Stop at end of day — sees their summary |
| Manager | Opens manager.html on laptop anytime |
| Manager | Sees all active drivers as colored lines on map |
| Manager | Click a driver card → map zooms to their route |
| Manager | Click "Trip History" → see all past trips + KM |

---

## Map colors
Each driver gets a different color route automatically.
- 🟢 Green dot = where they started
- 🔴 Red dot = where they are now (or ended)
- Colored line = the exact roads taken
