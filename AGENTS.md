# AGENTS.md

Instructions for any AI assistant reading, editing, or extending this repository. This file is about the *content* of this knowledge base — facts, sourcing, and conventions — not about git workflow, which is out of scope here.

## What this repository is

A fact-checked technical reference for FPV multirotor ESC (Electronic Speed Controller) firmware, tuning, and troubleshooting, covering BLHeli_32, Bluejay, AM32, and ESCape32. It was rebuilt after an earlier version was found to contain fabricated video citations and unsourced numbers presented as fact. Every convention below exists because it was violated once and caught.

## Never assume — verify, every time, no exceptions

This is the single most important rule in this file. This is a knowledgebase, not a writing exercise — a plausible-sounding number is not the same thing as a correct one, and "it's probably the same on both firmwares" is exactly the kind of sentence that has produced a real, published error in this repository more than once.

- Never carry a confirmed fact from one firmware, one video, one class of hardware, or one context to another without independently checking it applies there too. A concrete example of exactly this mistake happening in this repository: AM32's By-RPM PWM range (24-48kHz) is directly sourced from the AM32 creator's own words. That number was then copied onto BLHeli_32's Variable/By-RPM mode in several places without checking — but BLHeli_32's own Variable PWM explicitly extends to 128kHz, and BLHeli_32's By-RPM range has no confirmed source at all. The fix was not to soften the wording; it was to check the actual source for each firmware separately and state only what each one actually supports.
- If a query would need one more lookup to confirm — a video re-checked, a wiki page re-read, a UI dump re-scanned — do that lookup before writing the claim. Do not write the plausible version first and caveat it later.
- "Probably," "likely the same as," "should also apply to," and "consistent with" are phrases that belong after verification, not instead of it. If verification wasn't actually done, say so explicitly rather than writing a confident-sounding sentence.
- A false claim in a knowledgebase is worse than an admitted gap. "Not independently confirmed" is an acceptable thing to write. A wrong number stated as fact is not.

## Sourcing standard — non-negotiable

- Every concrete technical claim (a parameter range, a recommended value, a mechanism explanation, an attributed quote) must trace to one of: a real downloaded video transcript, an official firmware wiki or repository, a specific GitHub issue/PR, or an independently verifiable external reference (manufacturer docs, cross-checked community articles).
- If no such source exists, label the claim explicitly as an unverified community report or unverified community range — never state it as settled fact just because it "sounds right" or matches general background knowledge.
- Verify a source's actual content before citing it. Do not infer what a video or article probably says from its title, from general knowledge of the topic, or from what a plausible-sounding citation would imply. Read/watch/fetch it, or don't cite it.
- When a real citation is found to be wrong (wrong video, wrong timestamp, wrong number, misattributed creator), correct it — don't just soften the wording around a bad citation.
- Update `SOURCES.md` whenever a new source is added or an existing citation changes, so it stays an accurate index of what's cited where.
- Check a video's actual publish date (`yt-dlp --print upload_date`, not a guess) before treating it as current — firmware and community consensus both move over 2-4 years, and a newer source disagreeing with an older one is a real signal worth weighing, not noise to average away. `SOURCES.md` tracks verified publish dates for this reason.

## Firmware-specific accuracy

BLHeli_32, Bluejay, AM32, and ESCape32 are architecturally different firmwares, not interchangeable variants of one system. Confirmed differences already found the hard way:
- AM32 has no Demag Compensation setting at all (verified directly against the AM32 web configurator UI) — it uses Motor KV and Motor poles instead. Do not describe AM32 as having "Demag Timing" or any Demag control.
- BLHeli_32's Demag Compensation has four levels (Off/Low/Medium/High), not three.
- Bluejay's Demag Compensation has three levels (Off/Low/High).
- PWM-frequency recommendations (fixed 24kHz vs. 48kHz vs. Variable/By-RPM) differ by firmware even when tested by the same person on similar airframes — a finding sourced for one firmware does not automatically transfer to another. Verify per firmware before generalizing.
- Any firmware can technically run on any airframe size. Where a doc lists "common" firmware/size pairings, that reflects typical usage, not a capability restriction — don't imply exclusivity.

Before stating a setting exists on a given firmware, or a range/default applies to it, check that firmware's own actual configurator or wiki — don't assume parity across firmwares just because a parameter name sounds generic (e.g. "timing," "startup power").

## Voice and style

- State facts directly with their real, named source. Avoid vague collective attributions like "the creators' own approach" when it's unclear whose approach is meant (a video's presenter, a firmware's actual developer, and a community consensus are three different things — name which one).
- Do not narrate this document's own revision history inline ("an earlier draft said X, this was corrected to Y"). State the current, correct fact. Revision history belongs in commit messages, not in the reference content itself.
- Plain internal cross-references ("see elsewhere in this document," "see §3 above") are fine — that's normal technical-document navigation, not self-narration.
- Second-person instructional phrasing ("set X to your motor's actual KV") is normal and expected for a tuning guide — this is not the same thing as self-referential narration and should not be avoided.
- Keep the tone matter-of-fact and technical. No conversational filler, no hedging language that doesn't reflect genuine uncertainty.

## Repository structure conventions

- Raw research material (downloaded video transcripts, wiki page copies, per-video extraction notes) lives under `research/` and is tracked — it's the evidence trail for citations elsewhere in this repository. See `research/README.md` for what's in each subfolder. The one exception is `research/humand-pasted/` (user-provided screenshots and Discord messages), which is gitignored and never tracked — it isn't this repository's content to redistribute.
- Video transcripts under `research/transcripts/` are verbatim third-party content (auto-generated captions of other people's videos). They're included under the removal policy in `TAKEDOWN.md` — if a rights holder objects, remove the file(s) and update the citing doc to note the video was used as a source without including the verbatim transcript.
- `SOURCES.md` is the canonical list of every real source backing a claim in this repository, organized by category, including sources found during research that aren't yet cited in-text (flagged there for review rather than silently worked into prose).
- Each firmware/topic gets its own file; cross-link rather than duplicate. If the same procedure or data table would need to appear in two files, put it in one and link to it from the other instead.

## When you find something wrong

Fix it directly and correct the sourcing, rather than softening the language around an error. If a claim can't be verified one way or the other, say so explicitly rather than picking a side. If two real sources genuinely disagree, present both with their actual scope (what was tested, by whom, under what conditions) rather than forcing a single answer.
