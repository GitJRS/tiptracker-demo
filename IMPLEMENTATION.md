# Tip Tracker iOS – Implementation Guide

## 0. Purpose
Help service workers log tips quickly and recall them reliably. Core is a fast ledger of per-shift entries (cash + card) with accurate totals per day and period.

## 1. Architecture & Tech
- SwiftUI app; data via SwiftData/Core Data (local-first). Optional CSV export via `FileExporter`.
- State: `@State` for view-local, `@Query`/`@FetchRequest` or `@ObservableObject` store for entries, `@Environment(\.modelContext)` for mutations.
- Model: `TipEntry { id: UUID; date: Date; cash: Decimal; card: Decimal; note: String?; createdAt/updatedAt: Date }`. Computed `total = cash + card`.
- Validation: amounts ≥ 0; date not in distant future (configurable).
- Currency: store as `Decimal`; format with `FormatStyle.Currency(code: Locale.current.currency?.identifier ?? "USD")`.

## 2. Features & Expected Behaviors

### 2.1 Home / Today
1. Shows today’s total (cash, card, combined) computed from entries where `Calendar.isDateInToday(date)`.
2. Lists today’s entries with time, amounts, optional note. Sorted descending by time.
3. Primary call-to-action: “Log tip” opens New Entry sheet.
4. Empty state: friendly message + “Log tip” button; no placeholders for totals.
5. Pull-to-refresh not needed if using live queries; rely on data bindings.

### 2.2 New Entry (sheet/modal)
1. Fields: date (default today), time (default now), cash amount, card amount, optional note.
2. Input helpers: number pad with decimal; pre-fill last-used note/tag if desired.
3. Validation: amounts ≥ 0; reject NaN/inf; prevent save if both amounts are zero only if product says so (otherwise allow zero-card or zero-cash).
4. Save action:
   - Trim note; set `createdAt/updatedAt = now`.
   - Persist entry; dismiss sheet; updates propagate automatically.
5. Errors: show inline alert/banner; do not dismiss on failure.

### 2.3 History / Ledger
1. Default list: grouped by day, newest day first. Each day shows day total (cash, card, combined).
2. Expanding a day reveals entries with time + amounts (+ note).
3. Date filter: preset ranges (week, pay period, month) plus custom range.
4. Uses the same live store; no manual refresh. Sorting stable.

### 2.4 Entry Detail / Edit
1. From History or Today, tapping an entry opens detail/edit.
2. Fields identical to New Entry; pre-populated with current values.
3. Save updates fields and `updatedAt`; propagate back to lists.
4. Delete removes the entry after confirm; lists update immediately.

### 2.5 Summaries
1. Show quick totals for selected range: cash, card, combined.
2. Optionally display entry count and average per shift.
3. Export: CSV with headers (date, time, cash, card, total, note). Respect filters.

## 3. Data Model & Persistence
- SwiftData schema versioned; add migration step when changing model.
- Index on `date` for range queries; optional on `createdAt`.
- `TipEntry` conforms to `Identifiable` and `Hashable` if needed for lists.
- Sample CSV export row: `2024-03-12,18:00,120.50,80.00,200.50,"Friday patio"`.

## 4. UI/UX Details
- Formatting: use `Date.FormatStyle` with `.date(.abbreviated)` and `.time(.shortened)`.
- Accessibility: VoiceOver labels include cash, card, total, and date.
- Loading states: minimal—local data is immediate. Show progress only for exports.
- Errors: present non-blocking alerts; avoid silent failures.
- Theming: light/dark support; avoid hardcoded colors—use system semantic colors.

## 5. Testing Checklist
- Create entry: valid amounts, zero/zero policy, note trimming.
- Edit entry: amounts, date/time changes reflect in Today and History.
- Delete entry: removal updates totals and summaries.
- Filters: preset ranges produce expected totals; custom range boundaries inclusive.
- Export: CSV matches filtered dataset; numbers formatted with dot decimal.
- Timezone: Today and grouping respect current calendar/timezone.

## 6. Performance & Reliability
- Keep operations on main actor for UI; heavy exports on background task if large.
- Avoid redundant fetches; rely on live queries.
- Debounce filter changes if performing computed summaries over large sets.

## 7. Implementation Steps (order of work)
1. Define `TipEntry` model and persistence stack (SwiftData).
2. Build New Entry sheet and saving logic with validation.
3. Implement Today view with live list + totals.
4. Build History view with grouping and filters.
5. Add entry detail/edit/delete flow.
6. Add summaries and CSV export.
7. Add tests (unit for totals/filters, UI snapshot/behavior as needed).

## 8. Out of Scope (initial version)
- Cloud sync, multi-user accounts.
- Advanced analytics or dashboards.
- Push notifications.
- Tip-out splits or pooling logic.

