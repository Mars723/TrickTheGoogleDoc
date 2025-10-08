# TrickTheGoogleDoc

Reverse-engineering how Google Docs generates its version history to understand what version gaps can—and can’t—prove.

> ⚠️ **Responsible use**  
> This repository is for technical research, education, and reproducibility only. Do **not** use these observations to mislead instructors or evade academic-integrity policies.

-----

## Table of Contents

- [Inspiration](#inspiration)
- [Experimental Design & Procedure (with script)](#experimental-design--procedure-with-script)
- [Summary](#summary)
- [Applications](#applications)
- [Notes & Cautions](#notes--cautions)

-----

## Inspiration

This project started from a real-world frustration: professors checking Google Docs version history to spot copy-pasting or AI use. If the versions show big gaps with lots of content added at once, it looks suspicious—like you dumped in a block of text. But what if you could make fewer sub-versions, stretching out the intervals? That way, even if you did paste something in, the history looks more like natural, gradual writing. You could confidently explain to your prof that it’s all your own work. The goal here is to poke at Google’s hidden rules and find ways to make the version history work in your favor.

-----

## Experimental Design & Procedure (with script)

### Independent variables

- `i`: time interval (seconds) between two keystrokes  
- `t`: total experiment duration  
- input type: letters only / letters + spaces + punctuation / occasional newlines

### Measurement

After each run, open **File → Version history → See version history** and record the number of newly created **minor versions** `v`. When visible, also note approximate characters per version.

### Automation script (AppleScript, example)

```applescript
-- Automation script (AppleScript, example)

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
```

> For each run, adjust `delay` to the target `i` (e.g., **1s**, **5s**, **~10s**, **20s**) and keep other conditions fixed.

---

## Results (numeric summary)

| Exp | `i` (s) | Input                 | `t` (min) | Minor versions `v` | Notes (short)                     |
|---:|--------:|-----------------------|----------:|-------------------:|-----------------------------------|
| 1  | 1       | letters               | 12        | 1                  | merged                            |
| 2  | 0.5–1.5 | letters/space/punct   | 15        | 1                  | jitter didn’t split               |
| 3  | 5       | same                  | 25        | 1                  | merged                            |
| 4  | 20      | same                  | —         | ≈1 per input       | frequent splits                   |
| 5  | 11      | same                  | —         | ≈1 per input       | near a threshold                  |
| 6  | ~10     | same                  | —         | many               | often 1–2 chars/vers.; some 5     |
| 7  | 9.9     | same                  | —         | 7                  | per-version chars vary (2–25)     |
| 8  | 9.5     | same                  | —         | 2                  | ~60 chars; ~10 chars              |
| 9  | 9       | same                  | —         | 1                  | all merged                        |

> The table contains only short phrases; detailed interpretation is below.

---

## Summary

From the experiments, without network lag, Google Docs seems to create a new sub-version every ~11 seconds. But lag messes this up—saving a single input can take ~2s, shifting when versions split. The sweet spot: Keep inputs within every 9 seconds to “refresh” the timer and prevent new versions. This merges changes into fewer, larger versions, making the history look like steady work instead of big pastes.

My guess on the weird letter counts (like in Exp 7): Lag causes Docs to snapshot before a save finishes, bundling uneven groups. Closer to the 11s threshold, more variability.

-----

## Applications

The big win: Fewer sub-versions mean bigger time gaps between them, so you write a great amout of content with appproperiate time shown, instead of with 1 or 2 minutes, showing as a suspicious lump. Tell your prof it’s all gradual—because the history backs it up.

Bonus trick: If you paste text and the formatting screams "paste" (weird spacing, etc.), don’t freak. Just start editing right away, but make sure to input something every 9 seconds or less. This keeps the paste from triggering a new version immediately. You can fix the format over time, and as long as you stay under 9s per input, everything merges into the current version. By the time you stop, it looks organic.

Beside, The average typing speed for a university student composing an essay (thinking while writing) is about 19 words per minute (WPM), versus 33 WPM for copying text. This varies by factors like experience and complexity, but 10-20 WPM is typical for thoughtful writing. So try to avoid writing >60 words in two minutes.

**Source**: Karat et al. (1999) study on text entry patterns, cited in Wikipedia’s “Words per minute” article: https://en.wikipedia.org/wiki/Words_per_minute.

Overall, this helps you understand the mechanics behind Google Docs’ version generator better.

-----

## Notes & Cautions

Use AI for homework at your own risk—and use it wisely. Don’t copy-paste blindly. I don’t recommend relying on AI, and I oppose any attempt to evade academic-integrity policies.
