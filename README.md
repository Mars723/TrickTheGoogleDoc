TrickTheGoogleDoc

Reverse-engineering how Google Docs generates its version history to understand what version gaps can—and can’t—prove.

⚠️ Responsible use
This repository is for technical research, education, and reproducibility only. Do not use these observations to mislead instructors or evade academic-integrity policies.

⸻

Table of Contents
	•	Inspiration
	•	Experimental Design & Procedure (with script)
	•	Results (numeric summary)
	•	Findings & Interpretation
	•	Reproducibility & Environment
	•	Responsible Use & Caveats
	•	License

⸻

Inspiration

Some courses inspect Google Docs Version history to infer writing behavior (e.g., “pasted in one go,” “single draft,” “AI-assisted”). Out of engineering curiosity and for reproducibility, this project measures how minor versions are created under different typing cadences and input types, and highlights why version gaps alone are a weak signal.

⸻

Experimental Design & Procedure (with script)

Independent variables
	•	i: time interval (seconds) between two keystrokes
	•	t: total experiment duration
	•	Input type: letters only / letters + spaces + punctuation / occasional newlines

Measurement
	•	After each run, open File → Version history → See version history and record the number of newly created minor versions v. When visible, also note approximate characters per version.

### Automation script (AppleScript, example)

```applescript
set letters to {"a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"}

-- Baseline string to drive line breaks / punctuation rhythm
set baseline to "mbmaqslkteojnqclzyoziwbfcwazbmahtqxwnctnx"
set x to (length of baseline)

set totalKeystrokes to 1000
set typedLetters to 0
set jitter to (random number from 80 to 150) / 100.0 -- 0.80–1.50s jitter (optional)

tell application "System Events"
  delay 3 -- time to focus the browser and the doc
  repeat totalKeystrokes times
    -- (1) type a random letter
    set i to (random number from 1 to (count of letters))
    keystroke (item i of letters)
    set typedLetters to typedLetters + 1

    -- (2) line breaks at x and 2x+5
    if (typedLetters = x) or (typedLetters = (2 * x + 5)) then
      keystroke return
    else
      -- (3) punctuation every (x+3) characters
      if (typedLetters mod (x + 3) = 0) then
        set punctChoice to (random number from 1 to 2)
        if punctChoice = 1 then
          keystroke ","
        else
          keystroke "."
        end if
      end if
    end if

    -- (4) keystroke interval; set this to 0.5 / 1 / 5 / 10 / 20, etc.
    delay 1
  end repeat
end tell

For each run, adjust delay to the target i (e.g., 1s, 5s, 9–10s, 20s) and keep other conditions fixed.``` 

⸻

Results (numeric summary)

Exp	i (s)	Input	t (min)	Minor versions v	Notes (short)
1	1	letters	12	1	merged
2	0.5–1.5	letters/space/punct	15	1	jitter didn’t split
3	5	same	25	1	merged
4	20	same	—	≈1 per input	frequent splits
5	11	same	—	≈1 per input	near a threshold
6	~10	same	—	many	often 1–2 chars/version; some 5
7	9.9	same	—	7	per-version chars varied (e.g., 2–25)
8	9.5	same	—	2	~60 chars; ~10 chars
9	9	same	—	1	all merged


⸻

Findings & Interpretation
	•	Batching + periodic snapshots. Version creation behaves like a combination of save batching and periodic snapshots, not a simple fixed-second cutoff.
	•	Short intervals merge edits. With short keystroke intervals (a few seconds), large amounts of typing often collapse into a single minor version.
	•	Longer intervals split more. With longer intervals (≈10–20s), minor versions split more frequently, sometimes close to “one per input.”
	•	Non-deterministic boundaries. The number of characters per version varies widely even at the same i, suggesting client/network latency and server scheduling jitter shift the batch boundaries.
	•	Empirical neighborhood: around ~10–11 seconds we often observed easier splitting; below ~9 seconds we often saw merges. These are observations, not guarantees.

Educational takeaway: Version gaps are a weak signal. They should not be used in isolation to assert copy-paste, single-draft writing, or AI assistance.

⸻

Reproducibility & Environment
	•	Date / TZ: 2025-10-06, America/Indiana/Indianapolis
	•	Doc: blank Google Doc, default autosave, no add-ons
	•	Browser: Chrome (desktop)
	•	Network: residential broadband (ordinary variance)
	•	Method: automated keystrokes; manual counting in Version history

How to reproduce
	1.	Create a fresh Google Doc; disable unrelated extensions and collaborators.
	2.	Set the AppleScript delay to the target i.
	3.	Run for the planned duration t.
	4.	Open Version history and record new minor versions v.
	5.	Repeat for other i / input types and compare.

⸻

Responsible Use & Caveats
	•	Non-determinism. Version creation depends on network jitter, browser state, concurrency, and background tasks; identical settings may yield different splits.
	•	Scope. This project examines Docs’ version history behavior only. Related topics (e.g., comment anchors across revisions) are known to be unstable and are outside this study’s scope.
	•	Ethics. Do not use these observations to mislead instructors or mask misconduct. Treat version history as an imperfect, context-dependent artifact.

⸻

License

MIT (with the Responsible-Use notice above). Contributions with anonymized data points are welcome.
