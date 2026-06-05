# MASTER_CONTEXT.md — KMP Pattimura
> Last updated: June 2026 | Read this file first before touching any code.

---

## 1. WHAT IS THIS PROJECT

**KMP Pattimura** is a web-based reporting platform for **Koperasi Desa Merah Putih** across the **Kepulauan Maluku** region in Indonesia. It replaces the previous WhatsApp-based reporting system.

- Cooperative branches (koperasi) submit periodic financial reports
- A supervisor (pengawas) monitors all reports from a single dashboard
- AI auto-summarizes all incoming reports for the supervisor
- Fully mobile-friendly, works on HP (smartphone) from any island

**Live URL:** `silapor-pattimura-[xxx].vercel.app` (check Vercel dashboard)  
**GitHub Repo:** `github.com/Abihmdi/Si-Lpaor-Pattimura`  
**Main File:** `index.html` (single-file app, everything in one HTML file)

---

## 2. TECH STACK

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML + CSS + JavaScript (single file) |
| Database | Supabase (PostgreSQL) |
| File Storage | Supabase Storage |
| Hosting | Vercel (auto-deploy from GitHub) |
| AI | Anthropic Claude API (claude-sonnet-4-20250514) |
| Fonts | Google Fonts: Syne + DM Sans |

**No build tools. No npm. No framework. Pure HTML file.**

---

## 3. CREDENTIALS & CONFIG

```javascript
// In index.html
const SUPABASE_URL = 'https://vuincywepfqudgrkgecs.supabase.co';
const SUPABASE_KEY = 'sb_publishable_EEF-pJtnLlYFBkbJcSaAkw_NmobIvFc';

// Pengawas login password (hardcoded)
password: 'pengawas123'

// Koperasi passwords (stored in DB)
// KMP-Ambon: kmpambon, KMP-Seram: kmpseram, etc.
```

**Anthropic API:** Called directly from browser via `https://api.anthropic.com/v1/messages` — no API key needed in code (handled by Claude.ai artifact system).

---

## 4. DATABASE SCHEMA

### Table: `koperasi`
```sql
id          serial primary key
nama        text   -- e.g. 'KMP-Ambon'
kecamatan   text   -- e.g. 'Kota Ambon'
pulau       text   -- e.g. 'Kepulauan Ambon'
password    text   -- plain text password
```

### Table: `laporan`
```sql
id                  serial primary key
koperasi_id         int
koperasi_nama       text
pulau               text
periode             text   -- e.g. 'Mei 2025'
jenis               text   -- 'Bulanan' / 'Triwulan' / 'Tahunan'

-- Neraca
total_aset          bigint default 0
total_kewajiban     bigint default 0
piutang_anggota     bigint default 0
pinjaman_disalurkan bigint default 0
simpanan_pokok      bigint default 0
simpanan_wajib      bigint default 0
simpanan_sukarela   bigint default 0

-- PHU
pemasukan           bigint default 0
pengeluaran         bigint default 0
bunga_pinjaman      bigint default 0
shu                 bigint default 0

-- Operasional
anggota             int default 0
catatan             text
kendala             text
rencana_tindak      text

-- File
file_url            text   -- public URL from Supabase Storage
file_name           text   -- original filename

waktu_kirim         timestamptz default now()
```

### Supabase Storage
- Bucket: `laporan-files` (PUBLIC)
- File naming: `{koperasi_id}_{timestamp}_{filename}`
- Access: public read, authenticated write

### RLS Status
```sql
-- Both tables have RLS DISABLED (run manually if issues):
alter table koperasi disable row level security;
alter table laporan disable row level security;
```

---

## 5. KOPERASI DATA (7 branches)

| Nama | Kecamatan | Pulau | Password |
|------|-----------|-------|----------|
| KMP-Ambon | Kota Ambon | Kepulauan Ambon | kmpambon |
| KMP-Seram | Maluku Tengah | Kepulauan Seram | kmpseram |
| KMP-Buru | Kabupaten Buru | Kepulauan Buru | kmpburu |
| KMP-Kei | Maluku Tenggara | Kepulauan Kei | kmpkei |
| KMP-Tanimbar | Kepulauan Tanimbar | Kepulauan Tanimbar | kmptanimbar |
| KMP-Aru | Kepulauan Aru | Kepulauan Aru | kmparu |
| KMP-Babar | Maluku Barat Daya | Kepulauan Babar | kmpbabar |

---

## 6. APP ARCHITECTURE

Single-page app with 3 screens toggled via CSS class `.active`:

```
screen-login     → Role select + password
screen-koperasi  → Koperasi portal (submit reports)
screen-pengawas  → Supervisor dashboard
```

### Koperasi Tabs
1. **📤 Kirim Laporan** — Full form (Identitas, Neraca, PHU, Catatan, Lampiran)
2. **📋 Riwayat** — Past submitted reports

### Pengawas Tabs
1. **📊 Dashboard** — Status per kepulauan (sudah/belum lapor)
2. **📈 Rekap** — Aggregated totals, bar charts, ranking (filterable by period & island)
3. **📋 Laporan** — All incoming reports list
4. **🔍 Cari** — Keyword search across all report fields
5. **✨ AI** — AI auto-summary of all reports via Claude API

---

## 7. KEY FUNCTIONS IN CODE

```javascript
dbGet(table, query)        // GET from Supabase REST API
dbInsert(table, data)      // POST to Supabase REST API
uploadFile(file, kopId)    // Upload to Supabase Storage
doLogin()                  // Auth logic for both roles
kirimLaporan()             // Submit report + optional file upload
loadDashboard()            // Load pengawas dashboard stats
loadRekap()                // Load aggregated rekap with charts
loadSemuaLaporan()         // Load all reports list
doSearch()                 // Client-side keyword search
buatRangkuman()            // Call Claude API for AI summary
showDetailLaporan(l)       // Show modal with full report detail
showDetailKop(kopId)       // Show modal with koperasi summary
```

---

## 8. WHAT'S WORKING ✅

- Login for both koperasi and pengawas
- Full report form (Neraca + PHU + Catatan) per Permenkop UKM No.2/2024
- File upload (PDF/Excel) to Supabase Storage
- Pengawas can view uploaded files via link in report detail modal
- Dashboard with status per kepulauan
- Rekap total: pemasukan, pengeluaran, SHU, aset, simpanan, anggota
- Bar charts per kepulauan (pemasukan + SHU)
- Ranking koperasi by pemasukan
- Filter by periode and kepulauan
- Search by keyword (koperasi, periode, pulau, kendala, catatan)
- AI auto-summary via Claude API
- Report history per koperasi
- Mobile-friendly responsive design
- Deployed on Vercel, connected to Supabase

---

## 9. KNOWN BUGS & ISSUES 🐛

### File Upload Not Showing for Pengawas
- **Status:** Partially broken
- **Root cause:** Supabase Storage CORS or RLS issue — files may upload but URL returns error when pengawas tries to open
- **Fix attempted:** Set bucket to PUBLIC, added SQL policy, set `public=true` via SQL
- **Next step:** Verify CORS settings in Supabase Storage > Settings, add allowed origins `*` or the Vercel domain

### Authentication is NOT secure
- Passwords stored as plain text in DB
- Pengawas password hardcoded as `pengawas123`
- No JWT, no session management — just JS variable `currentUser`
- **This is a prototype — needs proper auth before production**

### No Real-time Updates
- Pengawas must manually refresh/re-click tabs to see new reports
- No WebSocket or polling implemented

---

## 10. NOT YET IMPLEMENTED ❌

1. **Export PDF** — Print/download laporan as formatted PDF document
2. **WhatsApp Notifications** — Alert pengawas when new report arrives (Fonnte/Zenziva)
3. **Reminder System** — Auto-remind koperasi that haven't submitted
4. **Pengawas Feedback** — Supervisor can comment/respond to reports
5. **Proper Authentication** — JWT tokens, secure sessions
6. **Multi-period Comparison** — Compare performance across months
7. **Data Export to Excel** — Export rekap as spreadsheet
8. **Admin Panel** — Add/edit/delete koperasi accounts

---

## 11. DESIGN SYSTEM

### Colors
```css
--red: #D42B2B          /* Primary brand */
--red-dark: #A81F1F     /* Hover states */
--red-light: #FFF0F0    /* Backgrounds */
--black: #0F0F0F        /* Login screen bg */
--green: #1A9E5C        /* Success states */
--orange: #E07B2A       /* Warning/pending */
--blue: #2563EB         /* Info/links */
```

### Typography
- **Syne** (800 weight) → headings, logo, buttons
- **DM Sans** (400/500/600) → body, labels, UI text

### UI Principles
- Dark login screen, light app screens
- Card-based layout with 16px border-radius
- Bottom sheet modals (slide up animation)
- Pill badges for status indicators
- Mobile-first, max-width 720px content area
- Sticky topbar with user chip

---

## 12. DEPLOYMENT PROCESS

1. Edit `index.html` locally
2. Push/upload to GitHub repo `Abihmdi/Si-Lpaor-Pattimura`
3. File must be named exactly `index.html` at root level
4. Vercel auto-deploys on every commit to `main` branch
5. No build step needed — static HTML

### Reset Data (if needed)
```sql
delete from laporan;  -- clears all reports, keeps koperasi
```

---

## 13. IMMEDIATE NEXT PRIORITIES

In order of importance:

1. **Fix file upload CORS** — Check Supabase Storage > Settings > CORS, add `*` or Vercel URL
2. **Export PDF** — Use `window.print()` with print CSS, or jsPDF library via CDN
3. **WhatsApp notif** — Integrate Fonnte API after laporan submitted
4. **Reminder** — Button in pengawas to send WA reminder to koperasi belum lapor
5. **Proper auth** — At minimum, hash passwords in DB

---

## 14. HOW TO ADD A NEW FEATURE

Since everything is one HTML file:

1. Add HTML in the correct screen section
2. Add CSS in the `<style>` block
3. Add JS function in the `<script>` block
4. If new DB columns needed → run `ALTER TABLE` SQL in Supabase SQL Editor
5. Upload updated `index.html` to GitHub → auto-deploys

---

## 15. AI CONTEXT SUMMARY

> For the next AI model — read this section first.

You are continuing development of **KMP Pattimura**, a single-file (`index.html`) web app for cooperative branch reporting in Maluku, Indonesia. The app is deployed on Vercel and uses Supabase as backend.

**The entire app is one HTML file.** No framework, no npm, no build tools. CSS and JS are inline in the file.

**The owner (Abi) is a non-technical indie developer** — explain things simply, give copy-paste ready SQL and code, avoid jargon. He communicates in casual Indonesian.

**Current blocker:** File upload works but pengawas cannot open uploaded files — likely a Supabase Storage CORS issue. The bucket `laporan-files` is set to PUBLIC but files still don't open. Check CORS settings in Supabase Storage configuration.

**When making changes:** Always output the complete updated `index.html` file — Abi uploads it directly to GitHub. Never give partial diffs or code snippets only.

**Supabase credentials are already hardcoded** in the file — do not ask for them again:
- URL: `https://vuincywepfqudgrkgecs.supabase.co`
- Key: `sb_publishable_EEF-pJtnLlYFBkbJcSaAkw_NmobIvFc`

**Tone:** Casual, direct, Indonesian language preferred. Abi gets frustrated with long explanations — be concise and action-oriented.

---

*End of MASTER_CONTEXT.md*
