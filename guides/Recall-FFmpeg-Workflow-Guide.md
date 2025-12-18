# Recall Timestamp Export - FFmpeg Video Editing Workflow Guide

**Date:** November 21, 2025
**Purpose:** Guide for using Recall's timestamp exports to automate video editing with FFmpeg

---

## Overview

When you enable **"Export timestamps and captions"** in Recall, the app generates 3 files for each video/audio file you transcribe:

1. **`filename_transcription.txt`** - Human-readable transcript
2. **`filename_data.json`** - Full timestamp data (for automation)
3. **`filename.srt`** - Standard subtitle captions

This guide explains how to use these files for video editing workflows.

---

## File Breakdown

### 1. **`_transcription.txt`** - Quick Reference

**Purpose:** Human-readable transcript for reviewing content

**Example:**
```
Speaker A (00:00:00 - 00:00:34): Can a crew can a kickback? We also want to thank you all. The Cana crew has really showed up every year. Each year you bring more money. I think last year we reached 6,500. You've outdone yourself every year and you've actually beat other corporations around here.
```

**Use Case:**
- Quick review to find interesting quotes
- Identify which segments you want to extract
- Share with team members for content review

---

### 2. **`_data.json`** - The Power Tool

**Purpose:** Complete timestamp data for programmatic access and FFmpeg automation

**Structure:**
```json
{
  "source_file": "C:/path/to/video.mp4",
  "audio_duration": 36,
  "confidence": 0.95563835,
  "text": "Full transcript...",

  "utterances": [
    {
      "speaker": "A",
      "text": "Complete thought or sentence",
      "start_ms": 640,
      "end_ms": 34730,
      "start_time": "00:00:00.640",
      "end_time": "00:00:34.730",
      "confidence": 0.95563835
    }
  ],

  "words": [
    {
      "text": "beat",
      "start_ms": 20700,
      "end_ms": 21000,
      "start_time": "00:00:20.700",
      "end_time": "00:00:21.000",
      "confidence": 0.98,
      "speaker": "A"
    }
  ]
}
```

**Key Fields:**

| Field | Description | Use Case |
|-------|-------------|----------|
| `utterances` | Complete thoughts/sentences by speaker | Extract full quotes or segments |
| `words` | Individual word timestamps | Frame-accurate editing, karaoke-style effects |
| `start_time` | FFmpeg-ready format (`HH:MM:SS.mmm`) | Direct use in `-ss` parameter |
| `end_time` | FFmpeg-ready format | Direct use in `-to` parameter |
| `confidence` | AI confidence score (0.0-1.0) | Quality checking, filter low-confidence segments |
| `speaker` | Speaker label (A, B, C, etc.) | Filter by speaker, create speaker-specific clips |

**Use Cases:**
- **Automated clip extraction** based on keywords
- **Speaker isolation** (extract all Speaker A segments)
- **Word-level precision** for tight edits
- **Quality filtering** (skip low-confidence segments)

---

### 3. **`.srt`** - Standard Subtitles

**Purpose:** Industry-standard subtitle format for captions

**Format:**
```srt
1
00:00:00,640 --> 00:00:03,920
Can a crew can a kickback?

2
00:00:04,720 --> 00:00:08,240
Okay. All right. We also do
```

**Use Cases:**
- **Burn captions into video** with FFmpeg
- **Upload to YouTube/TikTok/Instagram** as separate caption file
- **Import into video editing software** (Premiere, Final Cut, DaVinci Resolve)

---

## FFmpeg Workflows

### **Workflow 1: Manual Extraction (Using Timestamps)**

**Step 1:** Review `_transcription.txt` to find quotes you want

**Step 2:** Look up timestamps in `_data.json`

**Example:** You want to extract "beat other corporations"

Search in JSON:
```json
{
  "text": "beat",
  "start_time": "00:00:20.700",
  "end_time": "00:00:21.000"
}
```

**Step 3:** Use FFmpeg to extract segment

```bash
# Extract single segment
ffmpeg -ss 00:00:20.700 -i "video.mp4" -to 00:00:24.000 -c copy segment1.mp4

# Or with re-encoding for frame accuracy
ffmpeg -ss 00:00:20.700 -i "video.mp4" -t 3.300 \
  -c:v libx264 -crf 23 -preset medium \
  -c:a aac -b:a 192k \
  segment1.mp4
```

---

### **Workflow 2: Claude-Assisted Automation**

**Step 1:** Tell Claude (or another AI assistant) what you want

**Example:**
> "Extract segments where they talk about 'beat corporations' and 'Cana crew showed up'"

**Step 2:** Claude reads `_data.json` and generates FFmpeg commands

**Step 3:** Claude runs the commands automatically

**Output:** Individual segment files ready to concatenate

---

### **Workflow 3: Batch Extract All Utterances**

**Use Case:** Create individual clips for every speaker segment

**Python Script Example:**
```python
import json
import subprocess

# Load data
with open('video_data.json') as f:
    data = json.load(f)

# Extract each utterance
for i, utterance in enumerate(data['utterances']):
    start = utterance['start_time']
    end = utterance['end_time']
    speaker = utterance['speaker']

    cmd = [
        'ffmpeg', '-ss', start, '-i', data['source_file'],
        '-to', end, '-c', 'copy', f'utterance_{i:03d}_speaker_{speaker}.mp4'
    ]

    subprocess.run(cmd)
```

**Result:** 1 video file per utterance, named by speaker

---

### **Workflow 4: Burn Captions into Video**

**Use Case:** Add permanent captions to social media videos

**Command:**
```bash
ffmpeg -i video.mp4 -vf subtitles=video.srt output_with_captions.mp4
```

**Advanced (styled captions):**
```bash
ffmpeg -i video.mp4 \
  -vf "subtitles=video.srt:force_style='FontName=Arial,FontSize=24,PrimaryColour=&HFFFFFF&,OutlineColour=&H000000&,Outline=2'" \
  output_styled.mp4
```

---

### **Workflow 5: Create Social Media Reel**

**Example:** Extract 3 best quotes and combine into 60-second reel

**Step 1:** Identify timestamps from `_data.json`

```json
Quote 1: "00:00:20.700" to "00:00:24.000"
Quote 2: "00:01:15.200" to "00:01:19.500"
Quote 3: "00:02:05.100" to "00:02:10.800"
```

**Step 2:** Extract segments
```bash
ffmpeg -ss 00:00:20.700 -i video.mp4 -to 00:00:24.000 -c copy segment1.mp4
ffmpeg -ss 00:01:15.200 -i video.mp4 -to 00:01:19.500 -c copy segment2.mp4
ffmpeg -ss 00:02:05.100 -i video.mp4 -to 00:02:10.800 -c copy segment3.mp4
```

**Step 3:** Create concat list
```bash
cat > concat_list.txt << EOF
file 'segment1.mp4'
file 'segment2.mp4'
file 'segment3.mp4'
EOF
```

**Step 4:** Combine and format for Instagram/TikTok (vertical)
```bash
ffmpeg -f concat -safe 0 -i concat_list.txt -c copy combined_raw.mp4

ffmpeg -i combined_raw.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2,subtitles=video.srt" \
  -c:v libx264 -crf 23 -preset medium -b:v 4M \
  -c:a aac -b:a 192k \
  instagram_reel.mp4
```

---

## Advanced Use Cases

### **Filter by Speaker**

**Extract only Speaker A segments:**

```python
import json

with open('video_data.json') as f:
    data = json.load(f)

speaker_a_segments = [u for u in data['utterances'] if u['speaker'] == 'A']

for i, seg in enumerate(speaker_a_segments):
    print(f"Segment {i}: {seg['start_time']} - {seg['end_time']}")
    print(f"  Text: {seg['text'][:100]}...")
```

---

### **Search for Keywords**

**Find all segments mentioning "donation" or "support":**

```python
import json

with open('video_data.json') as f:
    data = json.load(f)

keywords = ['donation', 'support', 'contribute']

for utterance in data['utterances']:
    if any(kw in utterance['text'].lower() for kw in keywords):
        print(f"Found at {utterance['start_time']}: {utterance['text']}")
```

---

### **Confidence Filtering**

**Extract only high-confidence segments (>0.9):**

```python
high_confidence = [u for u in data['utterances'] if u['confidence'] > 0.9]
```

---

## Timestamp Format Reference

**Recall exports timestamps in TWO formats:**

| Format | Example | Use Case |
|--------|---------|----------|
| Milliseconds (`_ms`) | `20700` | Calculations, frame-accurate editing |
| FFmpeg Time (`start_time`) | `00:00:20.700` | Direct use in FFmpeg `-ss` / `-to` |

**Conversion (if needed):**
```python
# Milliseconds to FFmpeg format
def ms_to_ffmpeg(ms):
    seconds = ms / 1000
    h = int(seconds // 3600)
    m = int((seconds % 3600) // 60)
    s = int(seconds % 60)
    ms_part = int(ms % 1000)
    return f"{h:02d}:{m:02d}:{s:02d}.{ms_part:03d}"

# FFmpeg format to milliseconds
def ffmpeg_to_ms(time_str):
    h, m, s = time_str.split(':')
    s, ms = s.split('.')
    total_ms = int(h) * 3600000 + int(m) * 60000 + int(s) * 1000 + int(ms)
    return total_ms
```

---

## Quick Reference Commands

### **Extract Segment**
```bash
ffmpeg -ss START -i INPUT.mp4 -to END -c copy OUTPUT.mp4
```

### **Extract with Re-encoding (Frame Accurate)**
```bash
ffmpeg -ss START -i INPUT.mp4 -t DURATION -c:v libx264 -crf 23 OUTPUT.mp4
```

### **Burn Captions**
```bash
ffmpeg -i INPUT.mp4 -vf subtitles=FILE.srt OUTPUT.mp4
```

### **Vertical Format (Instagram/TikTok)**
```bash
ffmpeg -i INPUT.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" \
  -c:v libx264 -crf 23 OUTPUT.mp4
```

### **Concatenate Multiple Clips**
```bash
# Create list file
echo "file 'clip1.mp4'" > list.txt
echo "file 'clip2.mp4'" >> list.txt

# Concatenate
ffmpeg -f concat -safe 0 -i list.txt -c copy final.mp4
```

---

## Troubleshooting

### **"Timestamps don't match exactly"**
- **Cause:** Stream copy (`-c copy`) cuts at keyframes only
- **Solution:** Use re-encoding for frame accuracy (slower but precise)

### **"Caption timing is off"**
- **Cause:** SRT file may have been edited or video was trimmed
- **Solution:** Re-export SRT from original transcript

### **"Can't find a specific quote"**
- **Cause:** Transcription may have spelling variations
- **Solution:** Search for partial phrases or use fuzzy matching

---

## Integration with Other Tools

### **With Claude Code**
- Upload `_data.json` to Claude
- Ask: "Extract segments about [topic]"
- Claude generates and runs FFmpeg commands

### **With Python Scripts**
- Parse `_data.json` with `json.load()`
- Filter, search, and automate extraction
- Generate batch FFmpeg commands

### **With Video Editors**
- Import `.srt` for captions
- Use timestamps from JSON for precision cuts
- Combine manual editing with automated extraction

---

## Best Practices

1. **Always keep originals** - Don't delete source videos or JSON files
2. **Use meaningful filenames** - Name segments by content, not just numbers
3. **Verify timestamps** - Spot-check first few extractions before batch processing
4. **Test caption sync** - Preview SRT with video before final export
5. **Archive data files** - Keep `_data.json` for future re-editing

---

## Example Real-World Workflow

**Scenario:** Create a 90-second highlight reel from 6 interview videos

**Steps:**

1. **Transcribe all videos** with "Export timestamps" enabled in Recall
2. **Review transcripts** to identify best quotes
3. **Note timestamps** from `_data.json` files
4. **Extract segments** with FFmpeg:
   ```bash
   ffmpeg -ss 00:00:20.700 -i video1.mp4 -to 00:00:24.000 -c copy clip1.mp4
   ffmpeg -ss 00:01:15.200 -i video2.mp4 -to 00:01:19.500 -c copy clip2.mp4
   # ... repeat for all clips
   ```
5. **Combine clips**:
   ```bash
   # Create concat list
   ls clip*.mp4 | sed 's/^/file /' > clips.txt

   # Concatenate
   ffmpeg -f concat -safe 0 -i clips.txt -c copy combined.mp4
   ```
6. **Format for social media**:
   ```bash
   ffmpeg -i combined.mp4 \
     -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" \
     -c:v libx264 -crf 23 -preset medium \
     final_reel.mp4
   ```
7. **Add captions** (optional):
   ```bash
   ffmpeg -i final_reel.mp4 -vf subtitles=combined.srt final_with_captions.mp4
   ```

**Time saved:** Manual review (30 min) + Automated extraction (2 min) vs. Manual editing (4+ hours)

---

## Summary

**The Recall timestamp export gives you:**

✅ **Human-readable transcripts** for quick review
✅ **Structured JSON data** for automation
✅ **Standard SRT captions** for video platforms
✅ **FFmpeg-ready timestamps** (no conversion needed)
✅ **Word-level precision** for frame-accurate editing
✅ **Speaker identification** for multi-speaker content

**Use this to:**
- Extract highlight clips in seconds instead of hours
- Automate repetitive video editing tasks
- Create social media content from long-form videos
- Build searchable video archives
- Generate captions for accessibility

---

**Questions?** Refer to your FFmpeg research docs or ask Claude Code for help with specific extraction tasks!

**Last Updated:** November 21, 2025
