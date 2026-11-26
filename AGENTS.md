# AGENTS.md: Working Rules for Masroofati (مصروفاتي)

This file tells you (the coding agent) how to work on this project. Read it fully before making any change, and re-read it if you are unsure.

## 1. About the Project

Masroofati is a personal expense tracker in Arabic (right-to-left, Saudi riyal). The owner pastes a bank SMS, the app extracts the amount, date, description and type locally, suggests a category, and shows a dashboard, a transaction log and reports. Data is stored in the browser (IndexedDB). There is no server and no external AI or other external service.

Source of truth, in order:

1. `SPEC.md`: the technical specification. It decides what to build and how.
2. `PRD.md`: the product requirements (Arabic). It decides why and for whom.
3. The design concepts canvas (three UI options). The owner has **not** chosen one yet. Do not build the UI until the owner picks a direction (see 6.2).

If these files disagree, `SPEC.md` wins for the trial version. If something is unclear, ask the owner. Do not guess.

## 2. Who You Are Working With

The owner is **not technical** and prefers **Arabic**. Everything you say to them must be understandable without programming knowledge.

## 3. Language Rules

- **Think, plan, write code, comments, commit messages, file names and technical documents in English.**
- **Every message to the owner must be in Arabic only.** This includes questions, summaries, options, instructions, error explanations and progress updates.
- Do not paste raw code, stack traces, or terminal output into your Arabic replies. If something technical must be shown, explain it in plain Arabic first, and keep any snippet to the minimum.
- Keep Arabic replies short, warm and direct. Use plain words. If a technical term cannot be avoided, explain it in one sentence the first time.
- Keep user-facing app text in Arabic (labels, messages, buttons), as `SPEC.md` requires. Use English digits (0-9).

## 4. How to Communicate With the Owner

### 4.1 After every change, explain it simply

At the end of each change, send a short Arabic summary with:

1. **What changed**, in one or two plain sentences (what the owner will notice, not which files you edited).
2. **Why**, in one sentence.
3. **How to see it**, if there is something to look at.
4. **How to undo it**, in one sentence (see section 7).

Example of the right tone: "أضفت زر «مسح البيانات التجريبية» في لوحة التحكم. عند الضغط عليه تُحذف البيانات التجريبية فقط وتبقى عملياتك. يمكنني التراجع عن هذا التغيير متى شئت."

### 4.2 Present options in non-technical language

When there is a choice to make:

- Offer **2 to 3 options at most**, each in one or two sentences, describing **what the owner will see or experience**, plus the trade-off (cost, effort, risk) in everyday words.
- **Say which one you recommend and why**, in one sentence.
- Do not use jargon (framework, API, bundle, schema, cache). If it cannot be avoided, give a plain-language comparison.
- Ask **one question at a time**. Wait for the answer before moving on.
- Never make the owner pick between technical details they cannot judge. Decide those yourself, record the decision in the project notes, and tell the owner in plain terms.

### 4.3 Step-by-step instructions for anything outside the app

Whenever the owner must do something outside the app (creating an account, uploading a file, changing a setting, opening a link, using a website or phone setting), give **numbered steps, one action per step**, in Arabic.

- Say exactly what to click or tap and what they should see afterwards.
- Do one small group of steps at a time, then ask the owner to confirm before continuing.
- Say what to do if something looks different, and what **not** to share (passwords, codes, personal details).
- If the owner is stuck, offer to rewrite the steps differently. Never assume they know a technical term.

## 5. Hard Rules (never break these)

1. **Sample data only.** The app must contain only fictional, generic sample data. Never use, request, store, or log real personal or financial information: no real names, phone numbers, card numbers, account numbers, IBANs, balances, addresses, or real bank messages.
2. **Never ask the owner for personal data.** If a test needs a bank message, write an invented one. If the owner offers a real message, ask them to remove all sensitive details first and do not store the original.
3. **One change at a time.** Make a single, focused change, finish it, explain it, save it (section 7), and only then ask what is next. Do not bundle unrelated work.
4. **No features outside `SPEC.md` without asking.** If you think of something useful that is not in `SPEC.md`, do **not** build it. Ask the owner in plain Arabic, and wait for approval. Approved additions must be written into `SPEC.md` (see 6.3) before you build them. Ideas that are not approved go in the "Ideas for later" section of `SPEC.md`.
5. **No external services.** No server, no external database, no AI service, no analytics, and no network requests that carry user data. Respect the hosting limits in `SPEC.md` (single self-contained HTML file; external scripts only from approved hosts, each pinned to an exact version at least two weeks old).
6. **No secrets.** Never write passwords, tokens, or keys into any file. The login page is cosmetic: it must never store or validate the password field.
7. **Do not delete the owner's work.** Never remove files, data, or history without asking first.

## 6. Doing the Work

### 6.1 Before you start a change

- Re-read the relevant part of `SPEC.md`.
- State (to yourself, in English) the smallest change that satisfies the request.
- If the request is ambiguous or conflicts with `SPEC.md`, ask the owner one clear question in Arabic. Never fill gaps with assumptions about product behavior.

### 6.2 Design

The owner has seen three UI concepts and has not chosen one. Before building any screen, ask which direction they prefer (or what to combine). After they choose, record the decision in `SPEC.md` and follow that design. The login and reports screens still need to be designed for the chosen direction.

### 6.3 Keeping the documents in sync

- If a change alters behavior, update `SPEC.md` in the same change.
- Keep a short "Decisions" list in the project notes (what was decided, when, and why).
- Open questions from `SPEC.md` section 17 must be answered by the owner, not by you.

### 6.4 Engineering best practices

- Follow `SPEC.md` architecture: one HTML file, plain JavaScript, IndexedDB, Chart.js pinned to the version stated in `SPEC.md`.
- Keep code simple, readable and commented in English. Prefer small functions. Keep the keyword lists and parsing rules in clearly labeled places so they are easy to extend.
- Store money as whole halalas. Never use floating-point decimals for amounts.
- Handle errors without blank screens: show clear Arabic messages (empty input, parse failure, storage unavailable).
- Parsing must work fully offline and must never send the pasted text anywhere.
- Support right-to-left layout, phone and laptop screens, light and dark mode, keyboard focus, and tap targets of about 44 px.
- Do not add libraries, fonts or scripts beyond what `SPEC.md` allows.
- Test your change before saying it is done: run the relevant test messages from `SPEC.md` section 9.3, check the acceptance criteria in section 16 that your change touches, and check that a page refresh keeps the data. Tell the owner in plain Arabic what you checked and what you could not check.
- If something fails or you are unsure, say so honestly. Never claim something works without testing it.

## 7. Saving Versions (so the owner can go back)

Use **Git** so that every change is saved as a version the owner can return to.

1. If the project is not a Git repository yet, set it up and tell the owner in plain Arabic what this means ("نسخة محفوظة لكل تغيير يمكنك الرجوع إليها").
2. **Commit after every completed change** (one change, one commit). Never leave finished work uncommitted.
3. Write clear commit messages in English in the imperative mood, with a short title (about 50 characters) and a one-line reason if needed. Example: `Add clear-sample-data button to dashboard`.
4. **Tag milestones** (for example `v0.1-first-trial`) when a meaningful stage is complete, and tell the owner the milestone name in Arabic.
5. **Never rewrite or delete history** (no force push, no history rewriting) and never commit secrets or real personal data.
6. **Undoing:** when the owner asks to go back, use the safe method that keeps history (a new "revert" commit) rather than erasing anything. Explain in Arabic what will happen before you do it, and confirm afterwards.
7. Keep a simple list of versions for the owner (what each one contains, in plain Arabic) when asked.
8. If the owner wants an online backup of the versions (for example on GitHub), guide them step by step as in section 4.3, and never ask them to share passwords.
9. If Git is not available in your environment, tell the owner in Arabic and keep dated copies of the project files as a fallback, until Git can be used.

## 8. Quick Checklist Before Replying to the Owner

- [ ] Is the reply in Arabic only, short and free of jargon?
- [ ] Is it **one** change, finished and saved as a version?
- [ ] Did I explain what changed, why, and how to undo it?
- [ ] Is it inside `SPEC.md`? If not, did I ask first?
- [ ] Is all data fictional, and did I avoid asking for personal data?
- [ ] If the owner must act outside the app, are the steps numbered and simple?
- [ ] Did I test it and say honestly what I checked?
