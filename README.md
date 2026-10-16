# Attune

Attune is a dependency-free web app built with plain HTML, CSS and JavaScript. It watches how a person uses the page (mouse, touch, keyboard or voice) and adapts the interface to match. It also audits any pasted HTML for common WCAG problems, offers cognitive-support tools, and measures whether accessibility fixes actually make a task easier.

Most accessibility labs are static demos with a hard-coded broken page. This one reacts to the user, checks real markup, and produces numbers.

## What it does

- Listens for mouse, touch, keyboard and voice input and adapts target size, focus ring, theme and motion.
- Records every change in a "Why did it change?" log, so each adaptation is explained and can be undone.
- Runs a timed task on a broken form (version A) and a fixed form (version B) and compares the results.
- Parses pasted HTML in a detached document, reports WCAG issues and offers a fix with a before and after view.
- Provides reading aids and a readability score for people who find dense text hard to follow.
- Tracks which components work with which input methods in a modality matrix.

## Quick start

No build step and no install.

```
git clone https://github.com/areebanaveed2004/attune.git
cd attune
```

Then either open `attune.html` in Chrome or Edge, or serve the folder locally:

```
python -m http.server 8000
# visit http://localhost:8000/attune.html
```

Allow microphone access if you want to use voice commands. If voice recognition does not start when the file is opened directly, use the local server option.

## How to use it

1. Open the **Adaptive engine** tab. Move the mouse, tap the screen or press Tab, and watch the "Input detected" label and the log.
2. Click the small squares in the practice area, and miss them on purpose a few times, to see the targets grow.
3. Open **Before and after**, choose version A or B, and complete the form with mouse, touch or keyboard only.
4. Open **HTML auditor**, press Run audit, then apply fixes one at a time or all at once.
5. Open **Cognitive support** to check readability, switch on the reading ruler or focus mode, and score the auditor HTML for distraction.
6. Open **Modality matrix** and press cells to record which input methods work for each component.

## Adaptive signals

The engine starts with text at 100%, targets at 32 px and a 2 px focus ring.

| Signal | Trigger | Adaptation |
|---|---|---|
| Touch input | `pointerdown` from a touch pointer | Targets grow to 48 px |
| Missed clicks | 3 or more clicks that miss every control in the practice area within 10 seconds | Targets grow to 44 px |
| Erratic pointer | 10 or more sideways direction reversals within 1.5 seconds | Targets grow to 48 px |
| Keyboard navigation | Tab key pressed | Focus ring grows to 4 px |
| Reduced motion | System setting `prefers-reduced-motion` | Calm mode turns animation and transitions off |
| More contrast | System setting `prefers-contrast: more` | High-contrast theme |
| Dark colour scheme | System setting `prefers-color-scheme: dark` | The browser applies the dark theme, and the log notes it |
| Voice | A recognised voice command | See the voice table below |

Mouse and voice input are also detected and shown in the "Input detected" label. The "Adapt automatically" checkbox turns automatic changes off, and "Reset to defaults" restores every setting. Manual controls (theme, text size, spacing, voice) still work when automatic adaptation is off.

### Voice commands

Click Start voice commands and say one of these:

| Say | Result |
|---|---|
| bigger text | Text size up by 10% (maximum 200%) |
| smaller text | Text size down by 10% (minimum 80%) |
| dark mode | Dark theme |
| light mode | Light theme |
| high contrast | High-contrast theme |
| calm | Calm mode, with no animation |

The theme selector and text size buttons are included as a fallback for browsers without speech recognition.

## Before and after experiment

The task is to enter a name, an email address and a date, then send the form.

| Version | What is different |
|---|---|
| A: original | Placeholder-only fields, a tiny "go" control built from a `div` that keyboards cannot reach, and errors shown only as a red border |
| B: fixed | Visible labels, correct input types, a 44 px or larger Send button and error messages in text |

| Metric | Meaning | Why it matters |
|---|---|---|
| Time (s) | From the first action in the form to a valid submit | Overall effort |
| Tab presses | Number of Tab key presses during the run | Keyboard path length |
| Missed clicks | Clicks inside the form that hit no control | Target size and layout problems |
| Errors | Invalid submissions | Clarity of labels and error messages |
| Finished | Whether the run ended in a valid submit or with "I can't finish" | Whether the task was possible at all |
| Input | The last input method detected when the run ended | Context for the result |

Timing starts at the first pointer press, key press or focus in the form. The comparison chart shows the average time of finished runs for each version, and the results can be downloaded as JSON.

## HTML auditor

The pasted HTML is parsed with `DOMParser` in a detached document. It is never inserted into the page and never run.

| WCAG | Check | Fix |
|---|---|---|
| 1.1.1 | Image has no `alt` | Adds placeholder alt text that you must replace with real wording |
| 3.3.2 | Form field has no label | Adds an `aria-label` from the placeholder or name |
| 4.1.2 | Button or link has no accessible name | Adds a placeholder `aria-label` |
| 2.1.1 | Clickable element is not keyboard reachable | Adds `role="button"`, `tabindex="0"` and an Enter and Space handler |
| 2.4.3 | Positive `tabindex` | Sets it to 0 |
| 1.4.3 | Inline text colour contrast below 4.5:1 | Switches to black or white, whichever contrasts better |
| 1.3.1 | Heading level skipped | Reported only |
| 4.1.1 | Duplicate `id` | Reported only |

Each issue shows the original element and the fixed element. "Apply this fix" changes the text area and re-runs the audit, and "Apply all fixes" does the same for every automatic fix.

## Cognitive support

| Tool | What it does |
|---|---|
| Readability | Flesch reading ease (70 or above easy, 50 to 69 fair, below 50 hard), average sentence length and the number of sentences over 25 words |
| Reading ruler | A highlighted band that follows the pointer |
| Focus mode | Dims every paragraph except the one under the pointer or keyboard focus |
| Wider spacing | Increases letter spacing across the page |
| Distraction score | Scores the auditor HTML from 0 to 10 for autoplay (3 points each), marquee or blink tags (3), CSS animation (2), animated GIFs (2) and video or audio (1) |

## Modality matrix

Seven components (button, link, text field, dropdown, slider, modal dialog, drag and drop) are rated against five input methods (mouse, touch, keyboard, voice, single switch). Each cell cycles through Untested, Pass, Partial and Fail, and shows a symbol and a word as well as a colour. Columns for mouse, touch, keyboard and voice are marked "seen" once that input is detected. Single switch is never detected automatically.

## How it works

- **Settings:** a single settings object drives CSS custom properties (`--fs`, `--tap`, `--fw`, `--ls`) and a `data-th` theme attribute on the root element. Every automatic or manual change goes through one function that also writes the log entry and announces it in a polite live region.
- **Signals:** `pointerdown`, `pointermove` and `keydown` listeners feed the detection rules, and `matchMedia` reads system preferences on load.
- **Experiment:** one form is rendered at a time. Listeners on the form count Tab presses and missed clicks, and the run is saved when the form is submitted or abandoned.
- **Auditor:** each check queries the parsed document, and every fix is a function that can run on a clone to preview the result before it is applied to the real element.
- **Voice:** the Web Speech API runs in continuous mode and looks for command keywords in each transcript.

## Browser support

- Chrome and Edge are the intended targets, including voice.
- Other browsers should run everything except voice recognition. This has not been tested.

## Project structure

```
attune/
├── attune.html   # page, styles and script in one file
└── README.md
```

## Limitations and next steps

- Experiment results and matrix ratings are kept in memory and are lost on reload, so download the results first.
- One person completing a task is a method demonstration, not a statistical study. Several participants are needed for real conclusions.
- The adaptation thresholds (3 missed clicks, 10 reversals, and so on) are fixed values chosen by hand, not tuned on data.
- The auditor reads inline styles only and does not resolve styles from stylesheets. Its checks are a fast first pass and do not replace testing with a screen reader, a keyboard and real users.
- Voice recognition needs a browser with the Web Speech API and, in Chrome, an internet connection. Matching is keyword-based, so commands need to be spoken clearly.
- Possible extensions: an axe-core cross-check in the auditor, a session-timeout test for WCAG 2.2.1, a satisfaction rating after each experiment run, and remembering settings between visits.

## Author

Areeba Naveed
