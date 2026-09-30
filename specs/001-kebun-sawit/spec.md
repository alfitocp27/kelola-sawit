# Spec: Sistem Pengelolaan Kebun Sawit Keluarga

> Source of truth: `C:\Users/PC/.director/decision_brief_final.md` (DIRECTOR_APPROVED)

## User Scenarios & Testing

### User Story 1 - Record harvest on mobile (Priority: P1)

Ibu (primary user) opens the PWA on her phone, picks a plot, picks today's date, enters weight + price/kg (or reuses history) plus worker wage + extra cost, and taps the sticky Save button.

**Why this priority**: This is the core value — without recording harvest the system does nothing. Mobile-first because Ibu only has her phone.

**Independent Test**: Open the PWA on a phone, record one harvest entry, and verify the net result is auto-calculated correctly (weight × price/kg − wage − extra cost) without any manual math.

**Acceptance Scenarios**:
1. **Given** the harvest form is open on a mobile device, **When** the user enters weight 10 kg at Rp3,000/kg, wage Rp50,000, and extra cost Rp10,000, **Then** net result shows Rp240,000 (= 10×3000 − 50000 − 10000) and Save is enabled.
2. **Given** a previous harvest was saved for the selected plot, **When** the user starts a new entry, **Then** the prior weight and price appear as default values.

---

### User Story 2 - Maintain offline queue + persistence (Priority: P1)

The mobile form saves its state to local storage so accidental tab closes or offline periods never lose a half-typed entry.

**Why this priority**: Ibu has spotty connectivity and cannot afford data loss.

**Independent Test**: Start filling the harvest form on mobile, close the tab, reopen, and confirm the partially entered values are still present.

**Acceptance Scenarios**:
1. **Given** the user has typed weight but not yet tapped Save, **When** the browser tab is closed, **Then** reopening the PWA restores the entered weight.
2. **Given** the device is offline, **When** the user taps Save, **Then** the entry is added to a pending-sync queue.
3. **Given** the device comes back online, **When** the sync runs, **Then** all queued entries appear in the desktop report.

---

### User Story 3 - Record maintenance expenses (Priority: P2)

Ayah logs a fertilizer, spraying, or other maintenance cost against a plot.

**Why this priority**: Expenses are needed to compute true net result; deferred to P2 because manual wage entry already covers most P0 needs.

**Independent Test**: Add one expense entry and confirm it is counted in the plot's monthly expense total.

**Acceptance Scenarios**:
1. **Given** the maintenance form is open, **When** the user selects "pemupukan" and enters Rp25,000, **Then** the entry is saved and categorized.
2. **Given** one expense of Rp25,000 exists for a plot, **When** the report is viewed for that period, **Then** total expenses = Rp25,000.

---

### User Story 4 - View plot report on desktop (Priority: P1)

Ayah logs in on desktop, uses the sidebar to open the plot dashboard, and sees revenue, expenses, and net result for the selected period.

**Why this priority**: Reviewing aggregated results is the secondary user's main job.

**Independent Test**: After recording 2 harvest entries and 1 expense, open the desktop dashboard for the plot and confirm the three totals sum correctly.

**Acceptance Scenarios**:
1. **Given** 2 harvest entries (net Rp240,000 and Rp150,000) and 1 expense (Rp25,000) exist for April, **When** the April report is opened, **Then** revenue = sum of (weight×price), expenses = Rp25,000, net = (revenue − expenses).
2. **Given** harvest entries exist for 3 plots, **When** the user selects plot 2, **Then** only plot 2's data is shown.

---

### User Story 5 - Export data to CSV / JSON (Priority: P2)

Ayah exports monthly harvest + maintenance data to CSV for spreadsheet analysis, or exports the full database to portable JSON for backup.

**Why this priority**: Backup and portability are explicit Director requirements (zero-cost, DIY).

**Independent Test**: Click Export CSV, open the downloaded file in a spreadsheet, and confirm columns match harvest/expense fields.

**Acceptance Scenarios**:
1. **Given** harvest entries exist for a plot, **When** Export CSV is clicked, **Then** a file downloads with columns tanggal, berat_kg, harga_per_kg, upah, biaya_tambahan, hasil_bersih.
2. **Given** data exists across all tables, **When** Export JSON is clicked, **Then** a file downloads containing panen, biaya, kebun, and pekerja rows.

---

### Edge Cases

- What happens when the user opens the PWA with no plots created? → Show an empty state with a "Tambah Kebun" prompt.
- How does the system handle an offline Save with no prior connectivity? → Store in localStorage pending queue; flush on first reconnect.
- What happens when the selected plot has no entries in the chosen period? → Report shows zeros and an empty message.
- How does the system handle a failed export (no data)? → Export button is disabled or shows a toast "Tidak ada data untuk diekspor".

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow recording a harvest entry with weight (kg), price per kg, worker wage, and extra cost, and auto-calculate net result = (weight × price/kg) − wage − extra cost.
- **FR-002**: System MUST persist the harvest form state in browser local storage so data survives accidental tab close.
- **FR-003**: System MUST display harvest history and offer prior weight + price as defaults for the next entry.
- **FR-004**: System MUST preserve a history of sale price per kg for each harvest entry (P0/MVP).
- **FR-005**: System MUST preserve a history of harvested weight (kg) per harvest/setoran entry (P0/MVP).
- **FR-006**: System MUST allow recording maintenance expenses categorized as fertilizer (pemupuan), spraying (semprot), or other (lainnya).
- **FR-007**: System MUST display a summary report per plot showing total revenue, total expenses, and net result for the selected period.
- **FR-008**: System MUST support a single shared family account protected by a PIN/passcode, with session cookies marked Secure and HttpOnly.
- **FR-009**: System MUST rate-limit login attempts (5 failed attempts → cooling period).
- **FR-010**: System MUST export per-plot harvest and maintenance data to CSV.
- **FR-011**: System MUST export the full database to a portable JSON file.
- **FR-012**: System MUST queue offline form submissions and sync them to the backend on connectivity return.
- **FR-013**: System MUST support mobile form factors with 48px touch targets and decimal numeric keypad on numeric inputs.
- **FR-014**: System MUST support desktop layouts with sidebar navigation.
- **FR-015**: System MUST display Δ price badge (±RpX/kg vs prior entries) and Δ weight badge (±X kg) in the desktop dashboard (P1).
- **FR-016**: System MUST display a line chart of price and weight trends across harvest entries in the desktop dashboard (P1).
- **FR-017**: System MUST show a simple explanation of revenue change caused by price, weight, or both when displaying Δ badges (P1).

### Key Entities

- **app_user** — one family account; attributes: id, family_id, auth_pin_hash, created_at.
- **kebun** (plot) — attributes: id, family_id, nama, luas_m2, created_at.
- **panen** (harvest) — attributes: id, kebun_id, tanggal, berat_kg, harga_per_kg, upah, biaya_tambahan, hasil_bersih, catatan, created_at.
- **biaya** (expense) — attributes: id, kebun_id, tanggal, jenis (pemupukan/semprot/lainnya), deskripsi, nominal, created_at.
- **pekerja** (worker) — attributes: id, nama, family_id.
- **panen_pekerja** — junction table linking panen → pekerja (many-to-many).

## Success Criteria

### Measurable Outcomes

- **SC-001**: A family can record a harvest entry on mobile in under 30 seconds without calculation errors.
- **SC-002**: 100% of recorded harvest value + expenses reconcile to the same net-result figure the family calculates by hand.
- **SC-003**: Family can view a plot's month-to-date net result on desktop within 10 seconds.
- **SC-004**: Monthly CSV export for a plot with ≤ 365 entries completes in under 5 seconds.
- **SC-005**: Zero hosting cost is incurred for a family with 3 plots over a 12-month period.

## Assumptions

- Family uses 1 shared account for MVP; multi-user role system deferred to P1.
- Workers are selected from a manually-maintained per-family worker list.
- Price/weight "history" means reusing the most recent recorded values for the selected plot as defaults.
- Monthly export period = current calendar month (user-adjustable start/end).
- P0 stores raw records only; Δ-badge and trend charts arrive in P1.

## Clarifications

No open clarifications. All decisions sourced from the DIRECTOR_APPROVED decision brief.
