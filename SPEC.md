# Technical Specification: Masroofati (مصروفاتي)

**Version:** 0.1 (first trial version)
**Based on:** `PRD.md` (v1.0) plus the decisions in section 2
**Audience:** the product owner (non-technical) and whoever builds the app

> **How to read this document.** Technical terms are explained in plain words the first time they appear. Lines starting with **"In plain words"** summarize a section without jargon. Anything marked **[Proposed]** was chosen by the author, not by the owner, and can be changed. Anything marked **[Open]** still needs the owner's answer (collected in section 17).

---

## 1. Summary

Masroofati is a personal expense tracker in Arabic. The owner pastes the SMS text that a bank sends after a transaction. The app reads it, pulls out the amount, date, description and type, suggests a category, and saves it. A dashboard, a transaction log and a reports page show where the money goes.

This first version is a **trial**:

- It contains **sample data only**.
- It has a **cosmetic (fake) login page**, with no real authentication.
- Everything runs **inside the browser**. No server, no database, no external AI or other external service is used.

**In plain words:** one web page that works by itself on the phone or laptop, stores data inside the browser, and never sends anything out.

---

## 2. Decisions Log

### 2.1 Decisions made by the owner

| # | Decision | Effect |
|---|---|---|
| D1 | Login is cosmetic. The login page has a **name field and a password field**. The password is never stored or checked. | The entered name is shown in a bar at the top of every page. No role switcher. |
| D2 | A single role: the owner. | No permissions logic. |
| D3 | Sample data appears **automatically** on first open, with a **"Clear sample data"** button to start from zero. | See sections 8.2 and 12. |
| D4 | Dates are shown in **both Gregorian and Hijri**. | See section 9.4. (This is an addition to the PRD.) |
| D5 | Numbers use **English digits (0-9)**, not Arabic-Indic digits. | Applies to amounts, dates and charts. |
| D6 | Hosting: the owner delegated the choice. | See D8 below and section 5. |
| D7 | Category list: the owner gave no preference, so the proposed list is used as a placeholder. | See section 10. |

### 2.2 Decisions proposed by the author [Proposed]

| # | Decision | Reason |
|---|---|---|
| D8 | Publish the finished page as a **private link from the owner's existing Claude account**; no new account needed. Fallback: Cloudflare Pages (free). | Simplest for a non-technical owner. |
| D9 | Build as **one single HTML file** with plain JavaScript (no framework). | Fewest moving parts; works with the publishing method in D8. |
| D10 | Charts use the open-source **Chart.js** library, loaded from a trusted public address. | Free, well known, supports pie charts. |
| D11 | Amounts are stored as **whole halalas** (1 SAR = 100 halalas), not as decimals. | Avoids rounding errors such as 0.1 + 0.2. |

---

## 3. Differences From the PRD

| Area | PRD said | This spec |
|---|---|---|
| Login | No login | Cosmetic login page and name bar (D1) |
| Data | Real data | Sample data first, with a clear button (D3) |
| Dates | Not specified | Gregorian and Hijri (D4) |
| Digits | Not specified | English digits (D5) |
| Bank | Not specified | Rule-based reading of common SMS wording, since no bank formats were given (section 9) |

Everything else in the PRD stays as written.

---

## 4. Architecture

```
┌────────────────────────── Browser (phone or laptop) ──────────────────────────┐
│                                                                                │
│  index.html  (single file: HTML + CSS + JavaScript)                            │
│  ├─ Screens: Login · Dashboard · Analyze SMS · Transactions · Reports          │
│  ├─ SMS Parser  (pure JavaScript rules, no AI, no network)                     │
│  ├─ Category Engine (keyword rules)                                            │
│  ├─ Charts (Chart.js, loaded once from a trusted CDN)                          │
│  └─ Storage layer ───────────►  IndexedDB (inside this browser only)           │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
        No server · No database elsewhere · No user data leaves the browser
```

**In plain words:** the page does all the work itself. The only thing downloaded from the internet is the chart library (and optionally a font); after that, nothing is sent anywhere.

### Navigation

One page with several "screens" switched by the part of the address after `#`:

| Screen | Address ending | Notes |
|---|---|---|
| Login | `#login` | Shown when no name is saved |
| Dashboard | `#dashboard` | Default after login |
| Analyze SMS | `#analyze` | |
| Transactions | `#transactions` | |
| Reports | `#reports` | |

---

## 5. Technology Stack and Hosting

| Layer | Choice | Notes |
|---|---|---|
| Language | HTML, CSS, JavaScript (no framework) | [Proposed] |
| Charts | Chart.js, **pinned to version 4.4.1**, loaded from `cdnjs.cloudflare.com` | [Proposed] Pinned means a fixed version, so it cannot change unexpectedly. |
| Storage | IndexedDB (built into the browser) | Chosen by the owner in the PRD |
| Fonts | An Arabic-friendly web font with a fallback list (for example Tajawal, then system Arabic fonts) | If the font fails to load, the page still works. |
| Dates | Built-in browser date formatting (`Intl.DateTimeFormat`) | No extra library, including for Hijri. |
| Languages and direction | Arabic, right-to-left (`dir="rtl"`, `lang="ar"`) | |

### Hosting [Proposed, D8]

- **Primary:** publish the finished HTML file as a private link from the owner's Claude account. The link is visible only to the owner until they choose to share it.
- **Fallback (free, needs a free account):** Cloudflare Pages. Its free plan advertises unlimited static bandwidth, with limits such as 500 builds a month, which is far more than this project needs.

### Rules the hosting imposes on the file

- One self-contained `.html` file, under 16 MB.
- External scripts only from approved hosts, each with an exact pinned version (cdnjs is preferred). External styles only from Google Fonts.
- No remote images and no network calls to other sites. Anything else is blocked silently, so the page must not depend on it.
- The page must follow the viewer's light or dark mode.
- Layout must be responsive and keep content clear of the phone's top and bottom system bars.
- The app does not use the Claude API, `window.storage`, or any other platform-specific feature.

---

## 6. Browser and Device Support

| Item | Requirement |
|---|---|
| Browsers | Current versions of Chrome, Edge, Safari (iPhone and Mac) and Firefox |
| Screens | Phone (about 360 px wide and up) and laptop |
| Internet | Needed only to open the page the first time and load the chart library |

---

## 7. Data Model

Storage is an IndexedDB database named `masroofati`, version 1.

### 7.1 Store `transactions`

| Field | Type | Description | Example |
|---|---|---|---|
| `id` | text | Unique id generated by the app | `"a1b2c3"` |
| `amountHalalas` | whole number | Amount in halalas | `12550` (= 125.50 SAR) |
| `date` | text `YYYY-MM-DD` | Gregorian date; the Hijri date is computed from it | `"2026-11-20"` |
| `description` | text | Merchant or short description | `"كافيه الحي"` |
| `category` | text | Category id from section 10 | `"food"` |
| `type` | text | `purchase`, `withdrawal`, `transfer`, or `deposit` | `"purchase"` |
| `source` | text | `sms`, `manual`, or `sample` | `"sample"` |
| `createdAt` | text (ISO date-time) | When the record was saved | |

Indexes: `date`, `category`, `type`, `source`.

> The original SMS text is **not** stored. Only the extracted fields are saved, which keeps sensitive details (card numbers, balances) out of storage.

### 7.2 Store `settings`

| Key | Value |
|---|---|
| `displayName` | The name typed on the login page |
| `sampleSeeded` | `true` once sample data was inserted, so it is never re-inserted after "Clear sample data" |

### 7.3 Persistence behavior

- On startup the app requests "persistent storage" from the browser (`navigator.storage.persist()`) as a best-effort measure. It may be refused; the app works either way.
- Data stays on **one device and one browser**. The phone and the laptop each have their own separate data.

---

## 8. Screens and Functional Behavior

### 8.1 Login (cosmetic)

- Fields: **Name** and **Password**.
- The password is **never read, stored or validated**. Any value, including empty, is accepted.
- Name is required (non-empty after trimming spaces). If empty, show an Arabic message asking for a name.
- A visible note on the page: "نسخة تجريبية: تسجيل الدخول شكلي ولا يحمي البيانات" (Trial version: login is cosmetic and does not protect data).
- On submit: save the name in `settings.displayName`, go to `#dashboard`.
- If a name is already saved, skip this page and go straight to the dashboard.

### 8.2 Top bar (all screens except login)

- App name, the **saved name**, and the navigation links (Dashboard, Analyze SMS, Transactions, Reports).
- On a phone, the links must stay usable (for example a scrollable row).

### 8.3 Dashboard

Shows, based on **all saved transactions** (no date filter in this version):

1. **Stat cards:** total expenses (SAR), number of transactions, average daily expense, and the top category.
2. **Pie chart: expenses by category** (share of the total amount).
3. **Pie chart: expenses by transaction type** (purchase, withdrawal, transfer).
4. **Top categories list:** the highest-spending categories with amounts, in descending order.
5. **Spike days list:** days where spending rose (definition in section 11.3).
6. **Sample-data banner:** shown while any record has `source = sample`: "أنت تشاهد بيانات تجريبية" with a **"مسح البيانات التجريبية"** button.
   - The button asks for confirmation, then deletes only records whose `source` is `sample`.
   - Records the owner added remain.
   - After clearing, the dashboard shows an empty state with a link to Analyze SMS.

Empty state (no data): friendly message instead of empty charts.

### 8.4 Analyze SMS

1. A large text box: "الصق رسالة البنك هنا".
2. Button **"تحليل"**.
3. The parser (section 9) returns: amount, date, description, type, and suggested category.
4. Results show in **editable fields**. Nothing is saved until the owner presses **"حفظ"**.
5. If the owner changes the suggested category, the **category-mismatch alert** applies (section 8.6).
6. Messages:
   - Success: "تم التحليل. راجع البيانات ثم احفظ."
   - Partial (some fields missing): fields left empty and highlighted, with a message saying which ones need to be filled in.
   - Failure (no amount found): "تعذّر قراءة الرسالة. يمكنك إدخال العملية يدويًا." and the same form opens empty for manual entry.
7. One message at a time (assumption, section 17).

### 8.5 Transactions (log)

- A list or table, newest first, showing: date (Gregorian and Hijri), description, amount (SAR), category, and type.
- On a phone it appears as stacked cards rather than a wide table.
- Actions: **add manually**, **edit**, **delete** (delete asks for confirmation).
- Editing the category triggers the mismatch alert (8.6).

### 8.6 Category-mismatch alert

Rule: whenever the owner saves a transaction whose chosen category **differs from the category the app would suggest** from the description, show a dialog:

> "التصنيف الذي اخترته (X) يختلف عن التصنيف المقترح (Y). ماذا تريد؟"
> Buttons: **"أبقِ اختياري"** · **"استخدم المقترح"**

- "Keep mine": saves the owner's category.
- "Use suggested": saves the suggested category.
- The dialog is not shown when both match, or when the description is empty (nothing to compare).
- The app does not learn from the choice in this version.

### 8.7 Reports and Statistics

- **Summary figures:** total expenses, number of transactions, average per day, average per transaction, and the largest single expense.
- **Category table:** each category's total and its percentage of all expenses.
- **Daily totals table:** one row per day with that day's total expenses, newest first.
- Deposits are shown as a separate "total deposits" figure and are not part of expenses (section 11.1).

---

## 9. SMS Parsing Specification

> **In plain words:** the app looks for familiar words and number patterns in the message, like "SAR 45.50" or "لدى". It does not "understand" language like an AI; it follows written rules. Because no bank formats were provided, the rules cover common wording, and new patterns can be added as unreadable messages are found.

### 9.1 Steps

1. **Normalize the text.**
   - Convert Arabic-Indic digits (٠١٢٣…) to English digits.
   - Remove diacritics and the tatweel (ـ) character.
   - Collapse repeated spaces and line breaks.
2. **Detect the type** from keywords:

   | Type | Example keywords |
   |---|---|
   | `purchase` | شراء، مشتريات، عملية شراء، POS، Purchase |
   | `withdrawal` | سحب، سحب نقدي، صراف، ATM، Withdrawal |
   | `transfer` | تحويل، حوالة، سداد، Transfer, Sent |
   | `deposit` | إيداع، إضافة، استلام، راتب، Deposit, Credit |

   If several match, the first matching in the order above is used; if none match, the type is left empty for the owner to choose.
3. **Extract the amount.**
   - Look for a number next to a currency word: `SAR`, `SR`, `ر.س`, `ريال`, or after a word like `مبلغ` / `Amount`.
   - Accept decimals (`45.50`) and thousand separators (`8,000.00`).
   - **Ignore** numbers near `الرصيد` / `Balance` / `Avail` and card or account numbers (for example after `بطاقة`, `card ending`, `**`).
   - If a currency other than SAR is detected (for example `USD`), do not guess: leave the amount empty and warn that only SAR is supported.
4. **Extract the date.** Accepted patterns: `DD/MM/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`, `DD-MM-YY`. If no date is found, use **today's date** and show "لم يُعثر على تاريخ، استُخدم تاريخ اليوم".
5. **Extract the description** from text after markers such as `لدى`, `من`, `إلى`, `at`, `to`, `from`. Trim card numbers and trailing dates. If nothing is found, leave empty.
6. **Suggest a category** from the description using the keyword rules in section 10.
7. **Return** the fields for review (section 8.4).

### 9.2 Safety rules

- Parsing never makes a network request.
- Parsing never throws an error to the screen; failure produces the friendly messages in 8.4.
- The pasted text is not stored.

### 9.3 Test messages

> These messages are **invented examples** for testing, since no real messages were supplied. Replace or extend them with real (redacted) messages from the owner's bank.

| # | Message | Expected result |
|---|---|---|
| 1 | `شراء\nمبلغ: 45.50 SAR\nلدى: كافيه الحي\nالتاريخ: 2026-11-20` | purchase · 45.50 · 2026-11-20 · "كافيه الحي" · food |
| 2 | `Purchase of SAR 120.00 at PANDA RETAIL on 20/11/2026 using card ending 1234` | purchase · 120.00 · 2026-11-20 · "PANDA RETAIL" · groceries |
| 3 | `سحب نقدي بمبلغ 500.00 ر.س من صراف آلي بتاريخ 21/11/2026` | withdrawal · 500.00 · 2026-11-21 · other |
| 4 | `تم تحويل مبلغ 300.00 ر.س إلى حساب أحمد بتاريخ 22/11/2026` | transfer · 300.00 · 2026-11-22 · transfers |
| 5 | `إيداع بمبلغ 8,000.00 ر.س في حسابك بتاريخ 25/11/2026` | deposit · 8000.00 · 2026-11-25 |
| 6 | `شراء بمبلغ ٧٥٫٢٥ ر.س لدى صيدلية النهدي` (Arabic-Indic digits) | purchase · 75.25 · today's date (with notice) · health |
| 7 | `Purchase USD 20.00 at ONLINE STORE` | amount left empty, SAR-only warning |
| 8 | `مرحبا، هذه رسالة عادية` | failure message, manual entry offered |

### 9.4 Date display (Gregorian and Hijri)

- Dates are stored as Gregorian (`YYYY-MM-DD`).
- Each date is displayed as both, one after the other, using the browser's built-in formatting with Latin digits (locale options `ca-gregory` and `ca-islamic-umalqura`, with `nu-latn`).
- **Known limitation:** the browser's Hijri calendar can differ by one day from the official announced date. The Hijri date is for convenience only.
- Hijri dates written inside SMS messages are not read in this version.

---

## 10. Categories [Proposed, placeholder from D7]

| Id | Arabic label | Example keywords (used for the automatic suggestion) |
|---|---|---|
| `food` | مطاعم وقهوة | مطعم، كافيه، قهوة، برجر، بيتزا، coffee, restaurant, cafe |
| `groceries` | بقالة وتسوق | بقالة، سوبرماركت، بنده، الدانوب، التميمي، مول، panda, market, mall |
| `transport` | مواصلات ووقود | وقود، بنزين، محطة، أوبر، كريم، uber, careem, fuel |
| `bills` | فواتير وخدمات | كهرباء، ماء، اتصالات، stc، موبايلي، زين، فاتورة، bill |
| `health` | صحة | صيدلية، مستشفى، عيادة، النهدي، pharmacy, clinic |
| `entertainment` | ترفيه | سينما، نتفلكس، ألعاب، اشتراك، cinema, netflix |
| `transfers` | تحويلات | Assigned automatically when type is `transfer` |
| `other` | أخرى | Default when nothing matches |

- Matching is case-insensitive and ignores diacritics.
- The keyword lists live in one clearly labeled place in the code, so they can be expanded easily.
- The owner can still choose any category manually.
- Adding or renaming categories in the app is **not** part of this version.

---

## 11. Calculation Definitions

### 11.1 What counts as an expense [Proposed]

- **Expenses:** `purchase`, `withdrawal`, and `transfer`.
- **Not expenses:** `deposit`. Deposits appear only as "total deposits" on the Reports page.

### 11.2 Statistics

| Figure | Definition |
|---|---|
| Total expenses | Sum of all expense amounts |
| Average per day | Total expenses ÷ number of days that have at least one expense |
| Average per transaction | Total expenses ÷ number of expense transactions |
| Category share | Category total ÷ total expenses (shown as a percentage, one decimal place) |
| Top category | Category with the largest total |

### 11.3 Spike days [Proposed]

A **spike day** is a day whose total expenses are more than **1.5 times the average daily expense** (average over days that have expenses). The dashboard lists up to **5** spike days, highest first, each with its amount and dates (Gregorian and Hijri).

### 11.4 Money format

- Shown as `1,250.50 ر.س`: English digits, thousand separators, two decimals.
- Internally stored in halalas as whole numbers (D11).

---

## 12. Sample Data (D3)

- On the **very first open** (when `sampleSeeded` is not set), the app inserts about **30 transactions** with `source = sample`, then sets `sampleSeeded = true`.
- Dates are generated **relative to the first-open day** (roughly the previous 30 days), so the dashboard always looks current. [Proposed]
- Content: a mix of all categories; at least two deposits; and one or two heavier-spending days so the spike list is not empty.
- Sample descriptions are fictional and generic (no real people, cards or accounts).
- "Clear sample data" removes only `source = sample` records (section 8.3). The sample set is never re-inserted afterwards.

---

## 13. Privacy and Security

| Topic | Requirement |
|---|---|
| Data location | IndexedDB in the owner's browser only |
| Network | No requests carrying user data. SMS text, amounts and names are never sent anywhere. |
| AI services | None. Parsing is local rules. |
| Login | **Cosmetic only.** It provides no protection. Anyone using the same device and browser can open the data. The login page states this. |
| Stored SMS | The original message text is not saved (section 7.1). |
| Shared devices | Clearing browser data deletes everything. Nothing is recoverable. This is expected in this trial. |
| Third-party code | Only Chart.js (pinned version) and optionally a web font. |

---

## 14. Non-Functional Requirements

| Area | Requirement |
|---|---|
| Language | All visible text is Arabic; the layout is right-to-left. |
| Speed | Parsing a message in under 100 ms. The dashboard renders in under 1 second with 1,000 transactions. |
| Touch use | Buttons and fields are comfortable to tap on a phone (about 44 px minimum height). |
| Readability | Sufficient color contrast in both light and dark modes. Color is never the only way to convey meaning. |
| Errors | Every failure (storage unavailable, parse failure, empty input) shows a clear Arabic message, never a blank screen. |
| Storage unavailable | If IndexedDB cannot be opened (for example some private-browsing modes), show: "التخزين غير متاح في هذا المتصفح. جرّب نافذة عادية." |

---

## 15. Known Risks and Limitations

1. **Data can disappear.** Browsers can clear storage, and Safari may delete data for sites not visited for about a week. There is no backup in this version (backup and export are listed in section 18).
2. **No sync.** The phone and the laptop do not share data.
3. **Unknown bank wording.** Because no bank formats were given, some real messages may fail to parse at first. The fix is to add the real message wording to the rules.
4. **Hijri date accuracy.** It may differ by a day from the official date.
5. **Cosmetic login.** It protects nothing.

---

## 16. Acceptance Criteria

Mapped from the PRD, with the trial-version changes included.

| # | Criterion | How to verify |
|---|---|---|
| A1 | Messages 1 to 5 in section 9.3 parse with the expected fields | Paste each message and compare |
| A2 | Amounts are extracted correctly, including decimals, thousand separators and Arabic-Indic digits | Messages 5 and 6 |
| A3 | Balance and card numbers are never mistaken for the amount | Add a balance line to a message, confirm the amount is unchanged |
| A4 | Categories are suggested automatically for known merchants | Messages 1, 2 and 4 |
| A5 | Changing a category to one different from the suggestion shows the mismatch alert with both options working | Edit a transaction and test both buttons |
| A6 | Unreadable text shows the friendly failure message and offers manual entry | Message 8 |
| A7 | Data survives a page refresh and closing and reopening the browser | Add a transaction, refresh, reopen |
| A8 | The login page accepts any name and password, shows the name in the top bar, and the password is not stored anywhere | Log in with random values, inspect storage |
| A9 | Sample data (about 30 records) appears on first open, and "Clear sample data" removes only sample records and leaves the owner's own | Add one record, clear, confirm it remains |
| A10 | Dashboard shows total expenses, top categories, spike days, and both pie charts | Review with sample data |
| A11 | Each date shows Gregorian and Hijri; all numbers use English digits | Visual check on every screen |
| A12 | Works on phone and laptop, in Arabic right-to-left, without sideways page scrolling | Open on both |
| A13 | The page sends no user data out | Check the browser's network tab while using every feature; only the chart library (and font) may be fetched |
| A14 | Totals match: the sum of the category table equals total expenses | Compare Dashboard and Reports |

---

## 17. Open Items [Open]

Please confirm or change these. Defaults apply if you do not answer.

| # | Item | Default if unanswered |
|---|---|---|
| O1 | Real bank SMS examples (with card numbers and names hidden) | Use the invented test messages |
| O2 | Is one SMS pasted at a time enough? | Yes, one at a time |
| O3 | Should transfers count as expenses? | Yes (section 11.1) |
| O4 | Is the spike-day rule (1.5 times the daily average) acceptable? | Yes |
| O5 | How does the owner change the displayed name or return to the login page? | Not possible in this version; the name stays until browser data is cleared |
| O6 | Final category list (section 10) | Use the placeholder list |
| O7 | Should the dashboard cover all data or only the current month? | All data |

---

## 18. Out of Scope and Ideas for Later

### Out of scope (this version)

- Real login, accounts, multiple users, roles, or permissions.
- Any server, external database, or sync between devices.
- External AI services.
- Direct bank connections or automatic reading of SMS from the phone.
- Currencies other than SAR; languages other than Arabic.

### Ideas for later (not part of this version)

- Export to Excel/CSV, and a backup file with restore (reduces the data-loss risk).
- Paste several messages at once.
- Date and category filters on the log, dashboard and reports.
- Monthly budgets per category with warnings.
- The app learning from the owner's manual category corrections.
- Custom categories (add, rename, delete).
- A way to sign out or change the displayed name.
- Reading Hijri dates inside SMS messages.
- Installing the site on the phone like an app and working offline.
- Permanent hosting on a custom domain.
