# Transcript Summary Prompt Template

Summarize this transcript entitled: [TITLE]

- Analyze this transcript based solely on its content. Do not incorporate prior knowledge about the channel, speaker, or their typical positions.
- If domain-specific terminology requires explanation, define it based on context within the transcript.

CONTENT INGESTION REQUIREMENTS:
- Before writing any summary, read the ENTIRE transcript using the `Read` tool.
- If the transcript is large, use the `offset` and `limit` parameters of the `Read` tool to fetch remaining content in chunks.
- Continue fetching until you have read every line. Do not begin summarizing until the full text has been ingested.

TRANSCRIPT FORMAT:
- The transcript is provided in WebVTT (.vtt) format, which includes timestamps for each segment.
- Use these timestamps to anchor your analysis to specific moments in the video.
- When citing timestamps, use `[HH:MM:SS]` format (second resolution, no sub-seconds). For ranges, use `[HH:MM:SS--HH:MM:SS]` with double hyphens (`--`) as the separator, not an en-dash.
- When a concept, claim, or fallacy spans multiple consecutive segments, cite the full range covering the relevant passage.
- In markdown headings (`#`, `##`, `###`), timestamps appear without backticks. In all other text (bullet points, prose), wrap each timestamp in backticks for readability.

**OUTPUT FORMAT:**
Maintain neutral, descriptive tone; do not endorse or criticize the speaker's views.
Provide a structured summary with the following sections:

## Overview
A 3-5 sentence high-level summary of the video's thesis and scope.

## Source Material
- **Source:** [SOURCE]

Formatting rules for this section:
- If the source is a URL, render it as a markdown link: `[URL](URL)` (the link text and href are identical).
- If the source is a local file path, render it as inline code: `` `filename` ``.
- Always use exactly one bullet point with the bold label **Source:** followed by the value.
- Do not add any additional commentary, description, or metadata to this section.

## Key Terms and Concepts
A list of important terms, concepts, or frameworks introduced by the speaker, each with a definition or explanation based on what the speaker said or what can be directly inferred from context. Include the timestamp of the first mention. Use the format:
- **Term** `[HH:MM:SS]`: Definition

## Detailed Summary
A comprehensive section-by-section summary of the content, using subheadings to organize major topics or shifts in the discussion. Include a timestamp range for each subheading indicating the span of that section. Aim for thoroughness over brevity. Use the format:
### Section Title [HH:MM:SS--HH:MM:SS]

## Scripture References
(Include this section only if the content is theological)
List any Bible verses or passages mentioned, with brief context on how the speaker used them. Include all timestamps where each verse is referenced (not just the first mention), as speakers often introduce a verse in one context and revisit it later. Use the format:
- **Verse** `[HH:MM:SS]`, `[HH:MM:SS]`, ...: Context of how the speaker used it.

## Evidentiary Notes
Flag notable claims and categorize the type of support the speaker provides. Include a timestamp or timestamp range indicating where the claim and its supporting evidence appear. A claim and its evidence may span multiple segments; cite the full range. Use the format:
- **Category** `[HH:MM:SS--HH:MM:SS]`: Description of the claim and evidence.

Categories:
- **Anecdotal**: Personal experience or observation ("In my experience...")
- **Appeal to authority**: References another person's opinion without a direct source ("DHH believes...")
- **Logical argument**: Reasoned case without external data
- **Cited source**: Specific study, article, data point, or verifiable reference

## Logical Fallacies
Identify any logical fallacies present in the speaker's reasoning. If none are detected, state "No significant logical fallacies detected."
**IMPORTANT:** Only flag fallacies that significantly weaken the argument; rhetorical flourishes are not necessarily fallacies.

For each fallacy found, include a timestamp or range covering the relevant passage (premise through conclusion). A fallacy may involve a premise stated at one point and a conclusion drawn later; cite the full span. Use the format:
- **Fallacy type** `[HH:MM:SS--HH:MM:SS]`: Brief description of where/how it appears in the argument and why it weakens the reasoning.

Common fallacies to watch for:
- Ad Hominem (attacking the person rather than the argument)
- Straw Man (misrepresenting an opposing view to make it easier to attack)
- False Dichotomy (presenting only two options when more exist)
- Appeal to Emotion (relying on feelings rather than logic)
- Whataboutism/Tu Quoque (deflecting criticism by pointing to others' behavior)
- Slippery Slope (assuming one thing will inevitably lead to extreme consequences)
- Hasty Generalization (drawing broad conclusions from limited examples)
- False Authority (citing unqualified sources as experts)

## Questions and Underdeveloped Areas
List any points that seemed ambiguous, underdeveloped, or warrant further exploration. Include a timestamp or range indicating where the ambiguity or gap occurs. Explicitly state when interpretations are uncertain.

---

## Video Editing

After completing the transcript analysis above, produce an edited version of the original video with **no cuts to the content** — only the intro and outro are removed. The video is then accelerated by 10%.

### Step 1 — Find T_intro with two-pass frame extraction

The goal is to find the **exact timestamp** of the first frame where the presenter's face or body is visible — even if still in a crossfade or transition. Do **not** use 1fps alone; it is too coarse and will miss sub-second transitions.

**Pass 1 — Coarse scan (1fps, first 60s):**
```bash
ffmpeg -i input.mp4 -vf "fps=1" -frames:v 60 /tmp/frames/frame_%03d.jpg
```
Read the frames and identify approximately which second the transition from intro to presenter begins. Call this `T_approx`.

**Pass 2 — Fine scan (10fps, 6-second window around T_approx):**
```bash
ffmpeg -ss <T_approx - 3> -i input.mp4 -t 6 -vf "fps=10" /tmp/frames2/f%03d.jpg
```
Read these frames and find the **last frame where any intro animation element is still visible** (title cards, logo overlays, text overlays). The very next frame — where only the presenter against the clean background is visible — is `T_intro`. Each frame represents 0.1s.

**CRITICAL:** Do NOT set T_intro to the first frame the presenter *appears* (they may appear through a crossfade while intro elements are still visible), and do NOT set it to a "settled" frame 1–2 seconds after they first appear. The correct T_intro is the **earliest frame where the intro animation has completely vanished** — typically within 0.1–0.3s of the presenter first appearing.

Record the absolute timestamp as **`T_intro`** (in seconds, one decimal place).

**Audio-sync check:** Visually inspect the mouth at the T_intro frame. If the mouth is already open or mid-word at T_intro, the presenter began speaking during the intro fade. In that case, check the VTT for the first non-music/silence segment timestamp and pull T_intro back to that value (accepting a brief animated overlay at the very start of the edit). Document whether you adjusted for this.

**Why this matters:** Presenters in this series frequently begin speaking while the intro title card is still fading out. A T_intro set 0.5–1s too late will clip the opening word. Confirmed in Aula 5: presenter said "amigo minha amiga" but T_intro was set 0.8s late, clipping to "migo minha amiga".

### Step 2 — Find T_outro

**CRITICAL WARNING:** VTT timestamps for `[FOREIGN]` or unlabeled segments are **unreliable** — the auto-generated end time often cuts 1–3 seconds before the presenter actually finishes speaking. Never trust the VTT end timestamp alone for T_outro.

**Three-pass approach:**

**Pass 1 — Get VTT candidate:** From the VTT, find the end timestamp of the last segment that is not `[MUSIC]` or silence. Call this `T_vtt`. This is only a starting point.

**Pass 2 — Locate the fade/outro transition:** Extract frames from `T_vtt - 2s` to `T_vtt + 15s` at 2fps:
```bash
ffmpeg -ss <T_vtt - 2> -i input.mp4 -t 17 -vf "fps=2" /tmp/outro_check/f%03d.jpg
```
Find the **first frame where the presenter disappears or the outro animation/credits begin**. Call this `T_fade`. This is your hard upper boundary.

**Pass 3 — Fine scan around the fade:** Extract frames from `T_fade - 4s` to `T_fade + 2s` at 10fps:
```bash
ffmpeg -ss <T_fade - 4> -i input.mp4 -t 6 -vf "fps=10" /tmp/outro_fine/f%03d.jpg
```
Find the **last frame where the presenter's mouth is open or moving**. After that, the presenter closes their mouth and may stand still briefly before the fade begins.

Set **`T_outro = T_fade - 0.1s`**.

**Why T_fade - 0.1s (not the mouth-close frame + buffer):** Audio in Portuguese often has trailing nasal resonance (e.g. "ão", "em") that continues 0.2–0.5s after the visible mouth closes. The only safe guarantee that the last word is complete is to cut just before the fade animation starts — because the presenter has always fully finished speaking before the fade. A +0.3s buffer from the last visible mouth-open frame is not sufficient; it has caused repeated cut-word issues in practice.

### Step 3 — Trim intro and outro

Cut from `T_intro` to `T_outro` using re-encode for frame accuracy:
```bash
ffmpeg -y -ss <T_intro> -i input.mp4 -t <T_outro - T_intro> \
  -c:v libx264 -preset fast -crf 18 -c:a aac -b:a 192k trimmed.mp4
```

### Step 4 — Apply 1.1× speed
```bash
ffmpeg -y -i trimmed.mp4 \
  -filter_complex "[0:v]setpts=PTS/1.1[v];[0:a]atempo=1.1[a]" \
  -map "[v]" -map "[a]" \
  -c:v libx264 -preset medium -crf 24 -c:a aac -b:a 128k -movflags +faststart \
  final_editado.mp4
```

Delete the intermediate `trimmed.mp4` after this step.

### Step 5 — Enforce 12 MB file size limit

The output file **must not exceed 12 MB**. After Step 4, check the file size. If it exceeds 12 MB, re-encode with a calculated target bitrate:

```bash
# Get duration in seconds
DURATION=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 final_editado.mp4)

# Target: 11.5 MB = 11.5 * 8 * 1024 * 1024 bits
# Total bitrate (bps) = target_bits / duration
# Video bitrate = total_bitrate - 64000 (audio)
# Example for a 350s video: (11.5 * 8 * 1024 * 1024) / 350 - 64000 ≈ 211000 bps = 211k
```

Compute and hardcode the bitrate, then re-encode in place:
```bash
ffmpeg -y -i final_editado.mp4 \
  -c:v libx264 -preset medium -b:v <VIDEO_KBPS>k \
  -c:a aac -b:a 64k -movflags +faststart \
  final_editado_sized.mp4
mv final_editado_sized.mp4 final_editado.mp4
```

Use `-b:a 64k` (not 128k) when compressing for size — 64k AAC is fully acceptable for speech.

**Why 11.5 MB target (not 12):** Single-pass CBR can overshoot by 2–5%. Using 11.5 MB as the internal target ensures the output stays safely under 12 MB even with typical overshoot.

### Step 6 — Name the output file

Name the output file after the lesson, not generically. Use the format:
```
final_editado_<slug>.mp4
```
Where `<slug>` is a short identifier derived from the video title (e.g., `aula1`, `aula2_imperio`). Save it to the **same directory as the source video**.

### Validation checklist
Before delivering, confirm all items:
- [ ] Video starts on the **earliest clean presenter frame** — first frame where no intro animation element (title card, logo, text overlay) is visible. T_intro is NOT the first frame the presenter appears through a crossfade, and NOT a "settled" frame seconds later.
- [ ] Mouth inspected at T_intro frame: if mouth is already open/mid-word, T_intro was pulled back to audio start so no opening word is clipped.
- [ ] Video ends at the **last spoken word** — T_outro set to T_fade - 0.1s, verified by locating the fade transition visually, not just VTT timestamp.
- [ ] **No cuts inside the content** — the body of the video is intact.
- [ ] Video is **1.1× faster** (both video and audio).
- [ ] Audio/video sync preserved throughout.
- [ ] MP4 format (H.264/AAC).
- [ ] Intermediate `trimmed.mp4` deleted.
- [ ] Output filename follows the `final_editado_<slug>.mp4` convention.
- [ ] **File size ≤ 12 MB** — if over, re-encoded with calculated bitrate targeting 11.5 MB (Step 5).

### Deliverables
1. `final_editado_<slug>.mp4` — the edited video, in the same folder as the source, **≤ 12 MB**.
2. **`T_intro`** and **`T_outro`** timestamps used, with a one-line justification for each.
3. Note whether T_intro was adjusted for audio-sync and by how much.
4. Final file size.
