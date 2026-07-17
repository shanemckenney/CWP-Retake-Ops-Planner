# Retake Ops Planner

A self-serve CompTIA exam retake study planner. A student enters their score and what they missed — the tool builds a personalized, day-by-day study plan up to their retake date. No backend, no login, no data ever leaves the student's own device.

Built for the Cyber Warrior Program (CWP) at MyComputerCareer, where 30+ students across 5 certifications need personalized retake plans throughout a 19-week cohort — this replaces hand-building one for every student, every exam, every time.

**[Try it live →](#)** *(update this link once deployed to GitHub Pages)*

---

## For Students

### How it works

1. **Enter your info.** Your name, the exam, your score, and your planned retake date.
2. **Tell it what you missed.** Paste the objective numbers from your score report (like `2.1`, `3.4`), or check them off manually if you don't have the numbers handy.
3. **Get your plan.** A day-by-day countdown to your retake, with a warm-up recall question each day, a full practice exam a couple days out, and a light review day right before you test — not a cram session the night before.

### A few things worth knowing

- **Your plan saves on your own device only.** Nobody else — not another student, not your instructor — can see it. It's not sent anywhere; there's no server involved at all.
- **Come back day by day.** Each day unlocks on its actual calendar date. If you miss a day, it doesn't disappear — it rolls forward so you can catch up.
- **It auto-deletes.** Once your retake date passes, your data clears itself automatically. You can also hit "Clear my data" anytime.
- **If you were close** (within 50 points of passing) and testing soon, the plan automatically narrows to your top 5 highest-impact gaps instead of trying to cover everything — the rest show up in an "If You Have Extra Time" list, just not scheduled into a day.
- **Calendar reminders are optional.** Each day has "Add to Google Calendar" / "Add to Outlook" buttons, and there's a one-click download at the top for Apple/other calendar apps. This app doesn't and can't send you notifications on its own — hand it to your calendar app once, and let that do the reminding.
- **Private/incognito browsing** will stop your progress from saving between visits — you'll see a banner if that's the case. Use a normal browser window if you want it to remember you tomorrow.

That's it. No account, no install, no cost.

---

## For Instructors, Contributors, or Anyone Reusing This

### What this actually is

A single self-contained `.html` file. No build step, no npm, no framework, no server. Open it in a browser and it works. The only external dependency is Google Fonts, loaded from a CDN.

- **Storage:** `localStorage`, namespaced per exam, versioned, auto-expiring past the retake date. Nothing is transmitted anywhere — see `SECURITY.md` for the full write-up and the tier decision behind that.
- **No login, no account, no backend** — by design, not as a limitation. This keeps the tool FERPA-clean by architecture rather than by policy.

### Deploying it

Host `retake-planner.html` anywhere that serves static files — GitHub Pages, Netlify, a plain web server, doesn't matter. Hosting it at a real URL (rather than opening it as a downloaded local file) is actually recommended: browsers are inconsistent about `localStorage` on `file://` origins, and downloaded `.html` files can get flagged by antivirus/SmartScreen tools for containing embedded scripts. A hosted URL avoids both problems.

### Adding a new exam

All exam content lives in one JavaScript object, `EXAM_DATA`, near the top of the `<script>` block. Each exam needs:

```js
"exam-id": {
  name: "Exam Display Name",
  domains: [
    {id:"1.0", name:"Domain Name", weight: 20} // weight = % of exam, from CompTIA's official objectives doc
  ],
  objectives: [
    {
      id:"1.1",
      domain:"1.0",
      title:"Objective title, from CompTIA's official wording",
      roi:true,        // is this objective disproportionately high-value to study?
      confuse:true,     // is this commonly confused with something else?
      kind:"recall",    // "recall" (compare/explain/summarize) or "scenario" (given-a-scenario/troubleshoot/configure)
      tip:"Instructor's SME note on what trips students up here.",
      checklist:["Specific term or concept 1","Specific term 2", ...], // the actual named things to study
      recall:[
        // "recall" kind → multiple choice
        {type:"mc", q:"Question text?", options:["A","B","C","D"], answer:2},
        // "scenario" kind → short-answer with reveal
        {type:"short", q:"Scenario question?", a:"Answer text."}
      ]
    }
  ]
}
```

Then add the exam's passing score to `PASSING_SCORES`, and enable its `<option>` in the exam `<select>` dropdown (remove the `disabled` attribute).

**The content is the real work.** The engine (matching, scheduling, taper, triage) is exam-agnostic and just consumes whatever's in `EXAM_DATA`. Writing accurate `checklist`/`tip`/`recall` content per objective — grounded in the exam's official blueprint, not guessed — is what actually takes time. Currently: A+ Core 1 (220-1201) is fully built out with a reviewed 3-question recall bank per objective; CySA+ (CS0-003) has checklists/tips but no recall bank yet.

### How the plan actually gets built

- **`planTaper(daysUntil)`** decides the day structure: study days, then a full practice exam ~2 days before the retake, then a light review day right before it. Short countdowns drop the practice exam rather than cramming it in next to the retake.
- **Triage mode** auto-detects a near-miss by comparing the student's entered score against `PASSING_SCORES` — no extra question asked. If the score gap is ≤50 points *and* the retake is ≤3 days out, the plan caps at the top 5 highest-priority objectives instead of the full miss list.
- **The 5-per-day cap is universal, not just for triage** — regardless of score gap, no single day will ever get more than 5 objectives crammed into it. Overflow goes into the same "If You Have Extra Time" appendix.
- **Objective priority** = domain weight (from the official blueprint) + bonuses for `confuse`/`roi` flags. Bundled days interleave domains so a day doesn't accidentally become all-one-topic.
- All of the thresholds above (`50` points, `3` days, `5` per day) are plain constants near the top of the script — change them there if your program's numbers differ.

### Known limitations / things to know before extending this

- Objective matching is regex-based (`\d\.\d` pattern), not NLP. Unrecognized input falls back to a manual checklist rather than guessing wrong.
- No PDF score-report parsing — deliberately manual, to avoid file-upload handling as new attack surface.
- No cross-attempt memory — each retake plan is a clean slate, by design, to keep the privacy story simple.
- A hidden instructor testing utility exists in the source for verifying multi-day behavior without waiting real days or changing your system clock. Not documented here on purpose — ask directly if you need it.

### Files in this repo

- `retake-planner.html` — the whole application.
- `SECURITY.md` — the SSDLC security pass: attack surface, data handling, accepted tradeoffs, tier decision.

---

Built by Shane McKenney for the MyComputerCareer Cyber Warrior Program, with Claude.
