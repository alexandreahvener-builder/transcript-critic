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

After completing the transcript analysis above, generate a video editing plan to produce an optimized version of the original video. The goal is to reduce duration as much as possible **without compromising the logical flow or the quality of the material** — there is no fixed time target.

The timestamps from the transcript are the authoritative source for all cut points. Never estimate times by eye.

### Step 1 — Define speech boundaries
From the transcript, identify:
- **`T_start`** = `start` of the first speech segment — cut everything before it (intro music/jingle).
- **`T_end`** = `end` of the last speech segment — cut everything after it (outro music/jingle).

### Step 2 — Plan content cuts (most important step)
Build a **list of segments to KEEP**, each with exact `start` and `end` times from the transcript.

**Selection criteria (what to keep):**
- The hook/opening that presents the topic.
- The logical backbone of the argument, in original order — no segment should leave the viewer disoriented.
- Key scripture verses or supporting references — one per argument block, not every repetition.
- The closing appeal or conclusion.

**What to cut first:**
- Repetitions of the same point already made.
- Long lists of citations on the same theme — reduce to one representative example.
- Secondary digressions that don't break the thread if removed.
- Teasers for future videos, if there is room to cut them.

**Hard rules:**
- Never cut a sentence in half. Every kept segment must start at a speech `start` and end at a speech `end`.
- The video must not end mid-thought — the last segment must be a complete idea.
- Every transition between kept segments must make sense (topic → topic) with no argument left hanging.

> Before executing any cuts, deliver a **decision table** listing kept segments (`start` → `end` timestamps and the corresponding speech excerpt) for review.

### Step 3 — Trim and concatenate
Cut each segment using the real timestamps and concatenate in original order.
- **Edge padding:** start each segment ~0.15–0.25 s **before** the speech `start` and end ~0.15–0.25 s **after** the speech `end`, landing in the silence between phrases.
- Apply small audio fades at joins to avoid clicks.
- Use re-encode (not `-c copy`) for frame-accurate cuts.

### Step 4 — Apply 1.1× speed
```bash
ffmpeg -i montagem.mp4 -filter_complex "[0:v]setpts=PTS/1.1[v];[0:a]atempo=1.1[a]" -map "[v]" -map "[a]" acelerado.mp4
```

### Step 5 — Export
- Container **MP4**, video **H.264**, audio **AAC**.
- Resolution ≤ **720p**; preserve original aspect ratio.
```bash
ffmpeg -i acelerado.mp4 -c:v libx264 -preset medium -crf 24 -c:a aac -b:a 128k -movflags +faststart final_editado.mp4
```

### Validation checklist
Before delivering, confirm all items:
- [ ] Video is **1.1× faster**.
- [ ] **No** intro or outro music/jingle.
- [ ] **No** sentence cut in half — every segment starts and ends on real speech timestamps.
- [ ] The video **does not end mid-thought** — closes on a complete idea.
- [ ] **Transitions make sense** — no jump that confuses the viewer.
- [ ] The **central message** is comprehensible from start to finish.
- [ ] Audio has no clicks at joins; audio/video sync preserved.
- [ ] MP4 format (H.264/AAC).

### Deliverables
1. `final_editado.mp4` — the edited video.
2. **Cut table** — kept segments with real timestamps and corresponding speech excerpt.
3. **Selection rationale** — why each block was kept or removed (1–2 sentences each).
