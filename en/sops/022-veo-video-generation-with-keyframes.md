# SOP: Keyframe-Based Video Generation with Veo 3.1

> Generate smooth transition videos using Image-to-Video with first/last frame interpolation

**Document ID**: 022
**Date**: 2025-01-01
**Tags**: `Veo` `Video Generation` `AI Media` `Replicate`

---

## Overview

This document describes how to use Google Veo 3.1 to generate smooth transition videos from keyframe images. The core feature is **first/last frame interpolation** — by specifying the starting and ending frames, the AI automatically generates animated transitions between them.

---

## 1. Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Keyframe Video Generation Workflow                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │ Prepare  │ → │ Configure │ → │ Generate │ → │ Post-    │          │
│  │ Images   │    │ Params   │    │ Videos   │    │ Process  │          │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘          │
│       │                │               │               │                │
│       ↓                ↓               ↓               ↓                │
│  scene_001.png   video-config   Replicate API    Merge/Export          │
│  scene_002.png      .json       (Veo 3.1)                              │
│       ...                                                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Core Concepts

| Concept | Description |
|---------|-------------|
| **First Frame (image)** | Starting frame of the video |
| **Last Frame (last_frame)** | Ending frame for interpolation |
| **Interpolation** | Veo generates smooth animation between frames |
| **Scene** | An independent video segment, typically 4-8 seconds |

### Time Estimates

| Step | Duration | Notes |
|------|----------|-------|
| Prepare Images | Varies | Using Midjourney/DALL-E |
| Configure Params | 5 min | Write JSON config |
| Generate Videos | 2-5 min/scene | Replicate API call |
| Post-Processing | 10-30 min | Edit, merge, add music |

---

## 2. Prerequisites

### 2.1 Environment Requirements

```bash
# Node.js version
node -v  # >= 18.0.0

# Install dependencies
pnpm install replicate
```

### 2.2 API Configuration

Create `config.json`:

```json
{
  "REPLICATE_API_TOKEN": "r8_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

### 2.3 Directory Structure

```
your-project/
├── storyboard/
│   └── frames/           # Keyframe images
│       ├── scene_001.png
│       ├── scene_002.png
│       └── ...
├── video/                # Generated videos
│   ├── scene_001.mp4
│   └── ...
└── video-config.json     # Video configuration
```

### 2.4 Checklist

- [ ] Replicate API Token configured
- [ ] Keyframe images ready (16:9, 1280x720 or higher)
- [ ] Images named correctly: `scene_001.png`, `scene_002.png`...
- [ ] Understood scene content and transition requirements

---

## 3. Prepare Keyframe Images

### 3.1 Image Specifications

| Parameter | Recommended | Notes |
|-----------|-------------|-------|
| Aspect Ratio | 16:9 or 9:16 | Match target video |
| Resolution | 1280x720 or 1920x1080 | Higher is better |
| Format | PNG / JPG | PNG preserves more detail |
| Style | Consistent | Ensure uniform style across scenes |

### 3.2 Naming Convention

```
scene_{number}.png      # Regular scene first frame
scene_{number}_end.png  # Explicit end frame (optional)

Examples:
scene_001.png         # Scene 1 first frame
scene_002.png         # Scene 2 first frame (also Scene 1 last frame)
scene_003.png         # Scene 3 first frame (also Scene 2 last frame)
```

### 3.3 First/Last Frame Logic

The script automatically handles frame relationships:

```
Scene 1: First = scene_001.png → Last = scene_002.png
Scene 2: First = scene_002.png → Last = scene_003.png
Scene 3: First = scene_003.png → Last = scene_004.png
...
Scene N: First = scene_N.png   → Last = none (final scene)
```

**Important**: When using `--only` to filter scenes, include adjacent scenes for proper last frame!

```bash
# ❌ Wrong: Single scene, no last frame
--only=006

# ✅ Correct: Include adjacent scene
--only=006,007
```

---

## 4. Configure Video Parameters

### 4.1 Configuration File Structure

Create `video-config.json`:

```json
{
  "project_id": "promo-video",
  "project_info": {
    "title": "Promotional Video",
    "description": "Generate smooth transition videos from keyframes",
    "style_token": "premium futuristic, cinematic lighting, smooth camera movement"
  },
  "technical_recommendations": {
    "model": "google/veo-3.1-fast",
    "resolution": "1080p",
    "aspect_ratio": "16:9"
  },
  "scenes": [
    {
      "id": "001",
      "prompt": "Scene description, camera movement, atmosphere...",
      "duration": 4
    },
    {
      "id": "002",
      "prompt": "Scene description...",
      "duration": 6
    }
  ]
}
```

### 4.2 Scene Configuration Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Scene number, matches image naming |
| `prompt` | string | Video animation description |
| `duration` | number | Video length (4/6/8 seconds) |
| `reference_images` | array | R2V mode reference images (optional) |

### 4.3 Prompt Writing Tips

**Good Prompt Structure**:
```
[Subject Action] + [Camera Movement] + [Atmosphere/Lighting] + [Transition]

Example:
"AI core stabilizes at center. Energy flow slows down gradually.
Purple gradients deepen. Strong sense of security and trust.
Camera holds steady. Minimal motion, maximum calm."
```

**Key Elements**:
- Actions: `pulses`, `flows`, `converges`, `stabilizes`
- Camera: `camera slow push-in`, `orbits slightly`, `holds steady`
- Atmosphere: `cinematic`, `elegant`, `smooth`, `calm`
- Transitions: `transition to`, `fade`, `merge into`

---

## 5. Execute Video Generation

### 5.1 Basic Command

```bash
# Set output directory and run
CUSTOM_OUTPUT_DIR="/path/to/project" \
node generate-veo-video.js \
  --config=video-config.json
```

### 5.2 Command Line Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| `--config=` | Config file path (required) | `--config=./video-config.json` |
| `--only=` | Generate specific scenes only | `--only=006,007` |
| `--force` | Overwrite existing videos | `--force` |
| `--mock` | Mock mode | `--mock` |
| `--hailuo` | Use Hailuo model | `--hailuo` |
| `--model=` | Specify model ID | `--model=google/veo-3.1` |

### 5.3 Common Commands

**Generate all scenes**:
```bash
CUSTOM_OUTPUT_DIR="/path/to/project" \
node generate-veo-video.js --config=video-config.json
```

**Regenerate specific scenes (with first/last frame)**:
```bash
CUSTOM_OUTPUT_DIR="/path/to/project" \
node generate-veo-video.js \
  --config=video-config.json \
  --only=006,007 \
  --force
```

### 5.4 Expected Output

```
📖 Loading config: video-config.json
   🎯 Generating scenes: 006, 007
   Output Dir: /path/to/project/video
   Storyboard Dir: /path/to/project/storyboard/frames

🏎️  Starting generation queue (Concurrency: 2)...

🎬 [Scene 006] Preparing (google/veo-3.1-fast)...
   🖼️  [Scene 006] First frame: scene_006.png
   🔗 [Scene 006] Last frame: scene_007.png (interpolation)
   🚀 Submitting to Replicate...
   ✅ [Scene 006] Generated successfully!

🎉 All video tasks completed!
```

---

## 6. Post-Processing

### 6.1 Merge Videos

Using FFmpeg to merge multiple scenes:

```bash
# Create file list
cat > filelist.txt << EOF
file 'scene_001.mp4'
file 'scene_002.mp4'
file 'scene_003.mp4'
EOF

# Merge videos
ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.mp4
```

### 6.2 Add Background Music

```bash
ffmpeg -i output.mp4 -i bgm.mp3 \
  -c:v copy -c:a aac \
  -map 0:v:0 -map 1:a:0 \
  -shortest \
  final_video.mp4
```

### 6.3 Format Conversion

```bash
# Convert to WebM
ffmpeg -i final_video.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 output.webm

# Convert to MOV
ffmpeg -i final_video.mp4 -c:v prores_ks -profile:v 3 output.mov
```

---

## 7. Advanced Usage

### 7.1 R2V Mode (Reference-to-Video)

Veo 3.1 Standard supports multi-image reference mode:

```json
{
  "id": "003",
  "prompt": "Scene description...",
  "reference_images": [
    "reference_style_01.png",
    "reference_style_02.png"
  ]
}
```

**R2V Mode Features**:
- Up to 2 reference images
- First image auto-set as first frame
- Last image auto-set as last frame
- Forced 8-second duration, 16:9 ratio

### 7.2 Explicit Last Frame

Create `scene_XXX_end.png` to specify a custom ending:

```
storyboard/frames/
├── scene_006.png      # Scene 6 first frame
├── scene_006_end.png  # Scene 6 explicit last frame
└── scene_007.png      # Scene 7 first frame
```

---

## 8. FAQ

### Q1: Transitions not smooth?

**Solutions**:
1. Ensure consistent style across adjacent keyframes
2. Add transition words to prompts
3. Use intermediate frames

### Q2: Video not using last frame?

**Cause**: Using `--only` with a single scene.

**Solution**: Include adjacent scene `--only=006,007`

### Q3: API call failed?

| Error | Solution |
|-------|----------|
| `Invalid API token` | Check config.json |
| `Model not found` | Use correct model name |
| `Rate limit exceeded` | Wait and retry |

---

## 9. Appendix

### 9.1 Veo 3.1 Model Comparison

| Model | ID | Speed | R2V Support |
|-------|-----|-------|-------------|
| Veo 3.1 Fast | `google/veo-3.1-fast` | Fast | ❌ |
| Veo 3.1 Standard | `google/veo-3.1` | Slower | ✅ |
| Hailuo 2.3 Fast | `minimax/hailuo-2.3-fast` | Fast | ❌ |

### 9.2 Parameter Reference

| Parameter | Options | Default |
|-----------|---------|---------|
| `duration` | 4, 6, 8 | 8 |
| `resolution` | 720p, 1080p | 1080p |
| `aspect_ratio` | 16:9, 9:16 | 16:9 |
| `generate_audio` | true, false | true |

---

## References

- [Replicate Veo 3.1 Documentation](https://replicate.com/google/veo-3.1-fast)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
