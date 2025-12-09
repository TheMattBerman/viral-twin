# Viral Twin 2.0

**Reverse-engineer viral TikToks and generate branded alternatives using AI.**

Viral Twin analyzes a TikTok video, extracts what makes it work (pacing, hooks, camera angles, motion), then generates a new video with your brand's look—preserving the viral psychology while creating something original.

Built with n8n, FAL AI, Scrapecreators and Google's Gemini.

---

## What It Does

1. **Analyzes** a viral TikTok (visual breakdown, psychological hooks, pacing, segment-by-segment motion)
2. **Creates a Character Bible** (consistent subject appearance, environment, lighting across all frames)
3. **Generates images** for each 10-second segment using AI (nano-banana-pro)
4. **Animates each image** into video clips with motion matching the original (Kling 2.6 Pro)
5. **Stitches clips** into a final video (FAL FFmpeg)
6. **Uploads to Google Drive** and logs everything to a spreadsheet

The output: A 10-60 second vertical video that captures the _feel_ of the original without copying it.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           VIRAL TWIN 2.0 SYSTEM                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌───────────┐ │
│  │    MANUAL    │───▶│   ANALYZER   │───▶│  SYNTHESIZER │───▶│  STITCHER │ │
│  │    INGEST    │    │              │    │              │    │           │ │
│  │   (9 nodes)  │    │  (16 nodes)  │    │  (19 nodes)  │    │ (22 nodes)│ │
│  └──────────────┘    └──────────────┘    └──────────────┘    └─────┬─────┘ │
│         │                                                          │       │
│         │                                                          ▼       │
│         │                                                    ┌───────────┐ │
│         │                                                    │   LOGGER  │ │
│         │                                                    │ (4 nodes) │ │
│         └────────────────────────────────────────────────────┴───────────┘ │
│                                                                             │
│  Total: 5 workflows, 70 nodes                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Workflow Breakdown

| Workflow          | Purpose                      | Key Operations                                                                                      |
| ----------------- | ---------------------------- | --------------------------------------------------------------------------------------------------- |
| **MANUAL INGEST** | Entry point via n8n Form     | Accepts TikTok URL, archetype selection, brand color                                                |
| **ANALYZER**      | Deconstructs the viral video | Gemini 2.5 Flash vision analysis, segment extraction, character bible generation                    |
| **SYNTHESIZER**   | Generates new content        | Image generation (nano-banana-pro), video generation (Kling 2.6 Pro), sequential segment processing |
| **STITCHER**      | Assembles final video        | FFmpeg merge, Google Drive upload, shareable link generation                                        |
| **LOGGER**        | Records everything           | Google Sheets logging with full metadata                                                            |

---

## Requirements

### APIs & Services

| Service            | Purpose                                                      | Get It                                           |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| **FAL AI**         | Image gen (nano-banana-pro), Video gen (Kling), FFmpeg merge | [fal.ai](https://fal.ai)                         |
| **OpenRouter**     | Gemini 2.5 Flash for prompt generation                       | [openrouter.ai](https://openrouter.ai)           |
| **ScrapeCreators** | TikTok video/metadata extraction                             | [scrapecreators.com](https://scrapecreators.com) |
| **Google Drive**   | Final video storage                                          | Google Cloud Console                             |
| **Google Sheets**  | Logging                                                      | Google Cloud Console                             |

### n8n Credentials Required

- `FAL API` - HTTP Header Auth with your FAL key
- `OpenRouter API` - HTTP Header Auth with your OpenRouter key
- `ScrapeCreators API` - HTTP Header Auth
- `Google Drive OAuth2` - OAuth2 credentials
- `Google Sheets OAuth2` - OAuth2 credentials

### Environment Variables

```bash
# Workflow IDs (set after importing)
ANALYZER_WORKFLOW_ID=<workflow-id>
SYNTHESIZER_WORKFLOW_ID=<workflow-id>
STITCHER_WORKFLOW_ID=<workflow-id>
LOGGER_WORKFLOW_ID=<workflow-id>

# Google integrations
VIRAL_TWIN_SHEET_ID=<google-sheet-id>
VIRAL_TWIN_DRIVE_FOLDER_ID=<google-drive-folder-id>
```

---

## Setup

### 1. Import Workflows

Import in this order (dependencies matter):

1. `viral-twin-logger.json`
2. `viral-twin-stitcher.json`
3. `viral-twin-synthesizer.json`
4. `viral-twin-analyzer.json`
5. `viral-twin-manual-ingest.json`

### 2. Set Environment Variables

After importing, grab each workflow's ID from the URL and set the environment variables in n8n Settings → Variables.

### 3. Create Google Sheet

Create a sheet named `Viral_Twin_Log` with these columns:

```
video_id | timestamp | original_url | author | description | virality_reason |
audio_energy | pacing | tone | status | final_video_url | drive_url |
drive_download_url | total_duration | clip_count | segment_1_url | segment_2_url |
segment_3_url | segment_4_url | segment_5_url | segment_6_url | archetype |
brand_color | character_bible_json | all_clips_json | stitched_at
```

### 4. Create Google Drive Folder

Create a folder for outputs and grab the folder ID from the URL.

### 5. Configure Credentials

Map all credentials in each workflow to your n8n credential names.

### 6. Activate & Test

1. Activate MANUAL INGEST workflow
2. Access the form: `https://your-n8n-instance/form/viral-twin`
3. Submit a TikTok URL and wait

---

## Usage

### Form Interface

Access at: `https://your-n8n-instance/form/viral-twin`

**Fields:**

- **TikTok URL** - Full URL to the video you want to "twin"
- **Archetype** - Brand personality to apply:
  - `operator` - Calm competence, minimal flash (default)
  - `creator` - Artistic, expressive energy
  - `educator` - Clear, authoritative presence
  - `entertainer` - High energy, dynamic
  - `custom` - Define your own
- **Brand Color** - Hex color for subtle brand integration (default: `#10B981`)

### Processing Time

| Video Length | Segments | Approximate Time |
| ------------ | -------- | ---------------- |
| 10 seconds   | 1        | 8-12 minutes     |
| 20 seconds   | 2        | 16-22 minutes    |
| 30 seconds   | 3        | 25-35 minutes    |
| 60 seconds   | 6        | 50-70 minutes    |

Most time is spent waiting for Kling video generation (~6 minutes per segment).

---

## Cost Breakdown

**Per 30-second video (3 segments):**

| Service          | Operation               | Cost       |
| ---------------- | ----------------------- | ---------- |
| Gemini 2.5 Flash | Video analysis          | ~$0.01     |
| Gemini 2.5 Flash | 3× prompt generation    | ~$0.01     |
| nano-banana-pro  | 3× image generation     | ~$0.15     |
| Kling 2.6 Pro    | 3× 10s video generation | ~$2.10     |
| FAL FFmpeg       | Video merge             | ~$0.01     |
| **Total**        |                         | **~$2.28** |

At scale: ~$2.30 per 30-second video, ~$4.50 per 60-second video.

---

## How It Works (Technical)

### Analysis Phase

Gemini 2.5 Flash receives the video as base64 and outputs:

```json
{
  "vibe_check": {
    "audio_energy": "upbeat, trending sound",
    "pacing": "fast cuts, 2-second average shot length",
    "viral_hooks": ["pattern interrupt at 0:03", "payoff at end"],
    "tone": "comedic with surprise element"
  },
  "character_bible": {
    "subject_type": "person",
    "physical_description": "...",
    "environment": {
      "setting_type": "modern kitchen",
      "key_elements": "marble counter, pendant lights",
      "lighting": "warm natural from left"
    },
    "consistency_anchors": ["blue shirt", "wedding ring", "specific counter"]
  },
  "segments": [
    {
      "index": 0,
      "time_range": "0:00 - 0:10",
      "description": "Subject enters frame, picks up object",
      "camera": "medium shot, slight low angle",
      "motion_description": "walk-in from left, reach and grab motion",
      "key_frame_description": "subject mid-reach toward counter"
    }
  ]
}
```

### Synthesis Phase

For each segment:

1. **Generate image prompt** - Gemini creates detailed prompt using character bible + segment data
2. **Generate seed image** - nano-banana-pro creates 9:16 starting frame
3. **Generate video** - Kling 2.6 Pro animates the image using motion_description
4. **Poll for completion** - Wait for async FAL jobs to finish
5. **Loop to next segment**

Key architectural decision: **Sequential processing via Loop Over Items**. n8n's Wait nodes don't preserve multiple items in parallel, so segments process one at a time.

### Stitching Phase

1. Collect all segment video URLs
2. Submit to FAL FFmpeg for concatenation
3. Download merged video
4. Upload to Google Drive
5. Generate shareable link
6. Log everything to Google Sheets

---

## Customization

### Adding New Archetypes

Edit the MANUAL INGEST workflow's archetype options:

```javascript
{
  name: "Provocateur",
  description: "Challenges assumptions, creates productive tension",
  visual_cues: "direct eye contact, confident posture, dramatic lighting",
  energy: "intense but controlled"
}
```

### Adjusting Video Quality

In SYNTHESIZER, modify the Kling submission:

```javascript
{
  duration: '10',        // 5 or 10 seconds
  cfg_scale: 0.5,        // 0-1, lower = more creative
  aspect_ratio: '9:16'   // determined by input image
}
```

### Changing Image Model

Swap nano-banana-pro for another FAL model in Generate Seed Image node. Options:

- `fal-ai/flux-pro` - Different aesthetic
- `fal-ai/flux-realism` - More photorealistic
- Keep aspect_ratio: '9:16' for vertical video

---

## Troubleshooting

### "Can't determine which item to use"

After Merge nodes, use `$json` or `.first()` instead of `.item` to reference data.

### Videos are too static

Check that `motion_description` from Analyzer is being passed to Synthesizer prompts. The motion should match the original video's energy.

### Scene drift between segments

The character_bible.environment must be copied exactly into each segment's image prompt. Check the consistency_anchors are being used.

### Memory errors on large videos

n8n may run out of memory with 60-second videos. Increase Docker memory limits or process shorter videos.

### Polling timeouts

Kling Pro takes ~6 minutes per 10-second clip. Initial wait is 120 seconds, retry wait is 90 seconds. Adjust if needed.

---

## Limitations

- **Audio generated**: Kling 2.6 creates original audio for the video, but it does not match the original TikTok audio.
- **Character consistency**: Relies on prompt engineering, not LoRAs. Same "person" may vary slightly between segments. A future update my use Nano Banana pro for better character consistency.
- **Processing time**: Sequential generation means ~8-12 minutes per 10-second segment.
- **Max 60 seconds**: Capped at 6 segments to control cost and time.
- **API dependencies**: Requires FAL, OpenRouter, and Google APIs to all be operational.

---

## License

MIT License - Use it, modify it, build on it.

---

## Credits

Built by [Big Players](https://bigplayers.cp) - Stealing billion-dollar growth strategies so you don't have to.
Follow on [X](https://x.com/themattberman.com) 

**Stack:**

- [n8n](https://n8n.io) - Workflow automation
- [Scrapcreators](https://scrapecreators.com) - Workflow automation
- [FAL AI](https://fal.ai) - Image/video generation infrastructure
- [Kling](https://klingai.com) - Video generation model
- [Google Gemini](https://deepmind.google/technologies/gemini/) - Vision analysis and prompt generation

---

## Support

Questions? Issues?

- Open a GitHub issue
- Or reach out via the [Big Players newsletter](https://bigplayers.beehiiv.com)
