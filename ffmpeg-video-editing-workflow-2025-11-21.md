# FFmpeg Video Editing Workflow - Sojourner Interview Video

**Date:** November 21, 2025
**Purpose:** Edit 6 Sojourner interview video clips into one 60-90s vertical social media video
**Target Platforms:** Instagram Reels, TikTok, Facebook Reels
**Timeline:** Must complete by Nov 23 for posting

---

## Quick Start: Two Approaches

### **Approach 1: FAST WORKFLOW (Stream Copy) - 2-5 minutes total**
- **Speed:** ⚡⚡⚡ Lightning fast
- **Quality:** Original quality preserved
- **Limitations:** Can only cut at keyframes (may be 1-2 seconds off exact timestamp)
- **Transitions:** None (hard cuts only)
- **Best for:** Quick edits, tight deadlines, don't need fancy transitions

### **Approach 2: ADVANCED WORKFLOW (Re-encoding) - 15-40 minutes total**
- **Speed:** 🐌 Slower (depends on video length and PC specs)
- **Quality:** High quality (can control with CRF settings)
- **Limitations:** Takes longer to process
- **Transitions:** ✅ Crossfades, dissolves, custom transitions possible
- **Best for:** Polished final product, need precise timestamps, want smooth transitions

---

## APPROACH 1: FAST WORKFLOW (Stream Copy)

**Total Time:** 2-5 minutes for 6 clips

### Step 1: Extract Segments (No Re-encoding)

```bash
# Navigate to video folder
cd "C:/Users/figon/OneDrive/Kk6/Sojourner Center interview"

# Extract Segment 1 (Video 3 - Hook: "You've beat corporations")
ffmpeg -ss 00:00:15 -i "video3.mp4" -to 00:00:23 -c copy segment1.mp4

# Extract Segment 2 (Video 1 - Largest individual donor)
ffmpeg -ss 00:01:30 -i "video1.mp4" -to 00:01:45 -c copy segment2.mp4

# Extract Segment 3 (Video 2 - Track record)
ffmpeg -ss 00:02:10 -i "video2.mp4" -to 00:02:30 -c copy segment3.mp4

# Extract Segment 4 (Video 4 - Teens forgotten)
ffmpeg -ss 00:00:45 -i "video4.mp4" -to 00:01:00 -c copy segment4.mp4

# Extract Segment 5 (Video 5 - Gifting Hope dates)
ffmpeg -ss 00:01:20 -i "video5.mp4" -to 00:01:35 -c copy segment5.mp4

# Extract Segment 6 (Video 6 - Call to action)
ffmpeg -ss 00:00:30 -i "video6.mp4" -to 00:00:45 -c copy segment6.mp4
```

**Notes:**
- `-ss` = start time
- `-to` = end time (NOT duration!)
- `-c copy` = stream copy (no re-encoding, super fast)
- Cuts only at keyframes (may be 1-2s off exact timestamp)

### Step 2: Create Concatenation List

```bash
# Create text file listing all segments
cat > concat_list.txt << EOF
file 'segment1.mp4'
file 'segment2.mp4'
file 'segment3.mp4'
file 'segment4.mp4'
file 'segment5.mp4'
file 'segment6.mp4'
EOF
```

### Step 3: Concatenate All Segments

```bash
# Combine all segments into one video
ffmpeg -f concat -safe 0 -i concat_list.txt -c copy combined_raw.mp4
```

### Step 4: Optimize for Social Media (Final Re-encode)

```bash
# Re-encode for social media with vertical format
ffmpeg -i combined_raw.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" \
  -c:v libx264 -crf 23 -preset medium -b:v 4M \
  -c:a aac -b:a 192k \
  sojourner_video_final.mp4
```

**Settings Explained:**
- `scale=1080:1920` = Vertical format (9:16 for IG Reels/TikTok)
- `crf 23` = Quality (18=visually lossless, 23=good for social, 28=acceptable)
- `preset medium` = Speed/quality balance (ultrafast→veryslow)
- `b:v 4M` = Max video bitrate 4 Mbps
- `b:a 192k` = Audio bitrate 192 kbps

---

## APPROACH 2: ADVANCED WORKFLOW (Re-encoding with Transitions)

**Total Time:** 15-40 minutes for 6 clips

### Step 1: Extract Segments with Re-encoding

```bash
cd "C:/Users/figon/OneDrive/Kk6/Sojourner Center interview"

# Extract with precise timestamps (re-encodes for frame accuracy)
ffmpeg -ss 00:00:15 -i "video3.mp4" -t 8 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" \
  -c:v libx264 -crf 23 -preset medium \
  -c:a aac -b:a 192k \
  segment1.mp4

# Repeat for all 6 segments with their specific timestamps
```

**Notes:**
- `-t` = duration (instead of `-to` for end time)
- Re-encoding happens during extraction = precise frame accuracy
- All segments scaled to 1080x1920 vertical format

### Step 2: Add Crossfade Transitions

```bash
# Crossfade between segment 1 and segment 2 (1-second crossfade)
ffmpeg -i segment1.mp4 -i segment2.mp4 \
  -filter_complex "[0:v][1:v]xfade=transition=fade:duration=1:offset=7[vout];[0:a][1:a]acrossfade=d=1[aout]" \
  -map "[vout]" -map "[aout]" \
  merged_1_2.mp4

# Continue chaining crossfades for all segments
```

**Common Transition Types:**
- `fade` = Simple crossfade
- `wipeleft` = Wipe from left to right
- `slidedown` = Slide down effect
- `dissolve` = Dissolve transition

### Step 3: Final Social Media Optimization

```bash
# Final pass with Instagram/TikTok settings
ffmpeg -i final_merged.mp4 \
  -c:v libx264 -crf 23 -preset medium -b:v 4M -maxrate 5M -bufsize 8M \
  -c:a aac -b:a 192k -ar 48000 \
  -movflags +faststart \
  -pix_fmt yuv420p \
  sojourner_video_final.mp4
```

**Additional Settings:**
- `-maxrate 5M` = Prevents bitrate spikes
- `-bufsize 8M` = Buffer size for bitrate control
- `-movflags +faststart` = Optimizes for web streaming (loads faster)
- `-pix_fmt yuv420p` = Ensures compatibility with all devices

---

## TROUBLESHOOTING

### Issue: "Timestamps don't match exactly"
**Solution:** Use Approach 2 (re-encoding) for frame-accurate cuts, or accept 1-2s variance with Approach 1

### Issue: "Video orientation is wrong"
**Solution:** Add rotation filter before scaling:
```bash
-vf "transpose=1,scale=1080:1920:..."  # Rotate 90° clockwise
-vf "transpose=2,scale=1080:1920:..."  # Rotate 90° counter-clockwise
```

### Issue: "Audio out of sync"
**Solution:** Re-encode audio during extraction:
```bash
-c:a aac -b:a 192k  # Instead of -c:a copy
```

### Issue: "File size too large"
**Solution:** Increase CRF value (lower quality, smaller file):
```bash
-crf 28  # Instead of -crf 23 (saves ~40% file size)
```

### Issue: "Processing too slow"
**Solution:** Use faster preset (lower quality):
```bash
-preset fast      # Instead of -preset medium
-preset ultrafast # Emergency speed (quality suffers)
```

### Issue: "Concat fails with 'Unsafe file name'"
**Solution:** Use absolute paths in concat list:
```bash
file 'C:/Users/figon/OneDrive/Kk6/Sojourner Center interview/segment1.mp4'
```

---

## VERTICAL VIDEO OPTIMIZATION (9:16 for Instagram/TikTok)

### Force Vertical Format with Letterboxing

```bash
# If source video is horizontal, convert to vertical with black bars
ffmpeg -i input.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:black" \
  output_vertical.mp4
```

### Crop to Vertical (if source is square or horizontal)

```bash
# Crop center portion to 9:16
ffmpeg -i input.mp4 \
  -vf "crop=ih*9/16:ih" \
  output_vertical.mp4
```

---

## AUDIO HANDLING

### Normalize Audio Levels (Make all clips same volume)

```bash
# First pass: analyze audio
ffmpeg -i input.mp4 -af "volumedetect" -f null -

# Second pass: apply normalization
ffmpeg -i input.mp4 -af "volume=10dB" output.mp4  # Adjust 10dB to your needs
```

### Add Background Music (Ducking)

```bash
# Mix background music under voice (voice at full volume, music at 20%)
ffmpeg -i video.mp4 -i music.mp3 \
  -filter_complex "[1:a]volume=0.2[music];[0:a][music]amix=inputs=2:duration=shortest[aout]" \
  -map 0:v -map "[aout]" \
  output.mp4
```

### Remove Audio Completely

```bash
ffmpeg -i input.mp4 -an output_silent.mp4
```

---

## WORKFLOW AUTOMATION EXAMPLE

### Bash Script for Batch Processing

```bash
#!/bin/bash
# batch_extract_segments.sh

# Define timestamps (format: video_file start_time duration)
segments=(
  "video3.mp4 00:00:15 8"
  "video1.mp4 00:01:30 15"
  "video2.mp4 00:02:10 20"
  "video4.mp4 00:00:45 15"
  "video5.mp4 00:01:20 15"
  "video6.mp4 00:00:30 15"
)

# Extract all segments
counter=1
for seg in "${segments[@]}"; do
  read -r video start duration <<< "$seg"
  echo "Extracting segment $counter from $video..."
  ffmpeg -ss "$start" -i "$video" -t "$duration" -c copy "segment${counter}.mp4"
  ((counter++))
done

echo "All segments extracted!"
```

**Run:**
```bash
chmod +x batch_extract_segments.sh
./batch_extract_segments.sh
```

---

## QUALITY VS SPEED TRADE-OFFS

| Preset | Speed | Quality | Use Case |
|--------|-------|---------|----------|
| **ultrafast** | ⚡⚡⚡⚡⚡ | ⭐ | Preview, rough cut |
| **fast** | ⚡⚡⚡⚡ | ⭐⭐ | Quick social posts |
| **medium** | ⚡⚡⚡ | ⭐⭐⭐ | **RECOMMENDED for social media** |
| **slow** | ⚡⚡ | ⭐⭐⭐⭐ | High-quality final export |
| **veryslow** | ⚡ | ⭐⭐⭐⭐⭐ | Archival, professional use |

| CRF Value | Quality | File Size | Use Case |
|-----------|---------|-----------|----------|
| **18** | Visually lossless | Large | Archival, master copy |
| **23** | High quality | Medium | **RECOMMENDED for social media** |
| **28** | Acceptable | Small | Low-bandwidth, file size critical |

---

## TIME ESTIMATES (Based on Sojourner Video Project)

### Approach 1 (Stream Copy):
- **Extract 6 segments:** 30 seconds
- **Create concat list:** 10 seconds
- **Concatenate:** 20 seconds
- **Final social media encode:** 1-3 minutes (depends on total length)
- **TOTAL: 2-5 minutes**

### Approach 2 (Re-encoding):
- **Extract 6 segments with re-encode:** 5-15 minutes (depends on PC)
- **Add crossfade transitions:** 5-10 minutes
- **Final optimization:** 5-10 minutes
- **TOTAL: 15-40 minutes**

**Recommendation for Nov 23 deadline:** Use **Approach 1** (fast workflow) to meet deadline. Can always create polished version later with Approach 2 if needed.

---

## RECOMMENDED WORKFLOW FOR SOJOURNER VIDEO

**For Nov 23 Posting Deadline:**

1. **Use Approach 1** (stream copy for speed)
2. **Get transcript timestamps from Gilbert**
3. **Extract 6 key segments** (~8-15s each)
4. **Concatenate with hard cuts** (no transitions)
5. **Final encode to 1080x1920** with CRF 23
6. **Total time: ~5 minutes**

**Post-Deadline Polish (Optional):**
- Re-edit with Approach 2
- Add 1-second crossfade transitions
- Add background music (low volume)
- Add text overlays (can do in FFmpeg with drawtext filter)

---

## USEFUL REFERENCES

### Get Video Info
```bash
ffprobe -i video.mp4
```

### Extract Single Frame (for thumbnail)
```bash
ffmpeg -ss 00:00:05 -i video.mp4 -frames:v 1 thumbnail.jpg
```

### Speed Up/Slow Down Video
```bash
# 2x speed
ffmpeg -i input.mp4 -filter:v "setpts=0.5*PTS" output_2x.mp4

# 0.5x speed (slow motion)
ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" output_slow.mp4
```

### Add Text Overlay
```bash
ffmpeg -i input.mp4 \
  -vf "drawtext=text='KannaKickback 6':fontfile=/path/to/font.ttf:fontsize=72:fontcolor=white:x=(w-text_w)/2:y=50" \
  output.mp4
```

---

## NOTES FOR GILBERT

- **Timestamps:** Get exact timestamps from video transcripts
- **Format:** All source videos should be MP4 (H.264 codec)
- **Orientation:** Check if videos are vertical/horizontal before processing
- **Audio:** Normalize audio if clips have different volume levels
- **Deadline:** Use Approach 1 for Nov 23 posting
- **Testing:** Test final video on Instagram before posting (upload as draft)

---

**Last Updated:** November 21, 2025
**Next Steps:** Extract timestamps from transcripts, run Approach 1 workflow, post Nov 23
