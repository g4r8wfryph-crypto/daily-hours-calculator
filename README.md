# Daily Hours Calculator

A single-page timesheet calculator for non-exempt days that include field travel. Enter your transfer times in 24-hour format and it returns the total worked time in `HH:MM`, a decimal-hours figure for payroll entry, and a live projection of when you'll cross 8:00 and 8:30 worked.

No build step, no dependencies, no network calls, no data leaves the page.

---

## Why

Standard time clocks assume you clock in, maybe take lunch, and clock out. They don't handle a day that goes *work → drive → work → drive → work*, where travel is only partly compensated. This does, and it shows the arithmetic rather than just the answer.

## The rules it implements

| Rule | Behaviour |
|---|---|
| Travel grace | The first **0:30** of each travel leg is unpaid. `0:15` pays `0:00`; `0:50` pays `0:20`. Applied **per leg**, not per day. |
| Lunch | Optional and unpaid. Deducted from whichever work segment it overlaps. |
| Unused segments | Untick a segment, or leave a travel leg's transfer times blank, and it drops out — the work on either side runs straight through it. |
| Midnight | Any time earlier than the one above it is read as the next day, so overnight shifts total correctly. |
| Open travel | A leg with a start time but no end time counts as zero in the projections until both times are entered. |

## The day model

The sheet is a chronological run of six boundary times with five segments between them:

```
Clock in            ─┐
                     ├─ Pre-travel work
Travel 1 starts     ─┤
                     ├─ Travel 1          (first 0:30 unpaid)
Work resumes        ─┤
                     ├─ Midday work       (lunch deducted here, if taken)
Travel 2 starts     ─┤
                     ├─ Travel 2          (first 0:30 unpaid)
Non-exempt resumes  ─┤
                     ├─ Post-travel work
Clock out           ─┘
```

Internally the day is measured between the times you actually filled in, not segment by segment. A disabled or blank segment is removed from its stretch and the surrounding segments merge, which is why partial days and no-travel days total correctly.

One case is deliberately not guessed: a travel leg with only *one* of its two transfer times entered, sitting next to active work. There's no way to know where the drive ended, so that stretch is left uncounted and the readout names the leg and asks you to complete it or untick it.

## Projected clock-out

As you type, the panel shows the wall-clock time at which 8:00 and 8:30 of worked time would be reached, assuming the day carries on as it currently stands. A future lunch you've already entered is pushed out of the count. Once a target is passed, the row switches tone and reports the time you crossed it. Targets more than 24 hours out show a dash; overnight targets are marked `+1d`.

## Usage

**Standalone (`daily-hours-calculator.html`)**

Download and open it in any browser. It runs from `file://` — no server, no internet connection required.

**React component (`daily-hours-calculator.jsx`)**

A single default-export component whose only import is React. Drop it into an existing app:

```jsx
import HoursCalculator from "./daily-hours-calculator.jsx";

export default function App() {
  return <HoursCalculator />;
}
```

Styles are scoped under a `.tc` root class and injected by the component, so there's no stylesheet to wire up and nothing to collide with your global CSS.

### Entering times

Type `0745` or `7:45` — both normalize to `07:45` when the field loses focus. Bare hours work too: `9` becomes `09:00`. Invalid entries are outlined rather than silently discarded. 24-hour format throughout; there is no AM/PM anywhere in the tool.

### Output

- **Total worked** in `HH:MM`, with decimal hours beneath it
- **Breakdown** line by line, with unpaid travel and lunch shown as explicit deductions
- **Day bar** — a proportional strip of the whole day, hatched where time wasn't counted
- **Copy summary** — plain-text summary of the day for pasting into a timesheet

## Files

```
daily-hours-calculator.html   standalone build — open and go
daily-hours-calculator.jsx    React component build
README.md
```

Both builds share identical calculation logic and identical output; pick whichever fits where you're working.

## Configuration

The values worth changing live at the top of the model section in either file:

```js
const TRAVEL_GRACE = 30;      // unpaid minutes at the start of each travel leg
const TARGETS = [480, 510];   // projection targets, in minutes (8:00 and 8:30)
```

If your policy grants the 0:30 once per day rather than once per leg, that's a change to how the grace is summed, not a constant — open an issue and it's a small patch.

## Notes

State is held in memory only. Nothing is written to storage, so refreshing the page clears the day.

This is a personal calculator, not a payroll system of record. Verify the totals against your employer's policy before submitting them — travel compensation rules vary by employer, state, and classification.

## License

Copyright © 2026 Zaid Hamilton.

Daily Hours Calculator is licensed under Creative Commons Attribution-NonCommercial 4.0 International (**CC BY-NC 4.0**).

You're free to share and adapt this for non-commercial purposes with attribution. See `LICENSE`, or https://creativecommons.org/licenses/by-nc/4.0/ for the full terms.
