---
title: Sora Video Generation Prompting Guide
date: 2025-12-02
research_query: "Sora video generation prompting best practices for thematic brand commercials"
completeness: 90%
performance: "v2.0 wide-then-deep"
execution_time: "1.2 minutes"
---

# Sora Video Generation Prompting Guide
**For Thematic Brand Commercials with Lofi/Chill/Stoner Aesthetic**

## Table of Contents
1. [Prompt Structure & Syntax](#prompt-structure--syntax)
2. [Camera Movements & Cinematography](#camera-movements--cinematography)
3. [Consistent Style Across Videos](#consistent-style-across-videos)
4. [Short-Form Content (15-30 Seconds)](#short-form-content-15-30-seconds)
5. [Character Actions & Transitions](#character-actions--transitions)
6. [Lofi/Chill Aesthetic Techniques](#lofichill-aesthetic-techniques)
7. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
8. [Example Prompts](#example-prompts)

---

## Prompt Structure & Syntax

### Optimal Prompt Formula

**Basic Structure (Beginner):**
```
[Subject] + [Action] + [Setting] + [Style/Mood]
```

**Example:**
```
A golden retriever puppy playing with a red ball in a sunny park,
shot in cinematic style with warm afternoon lighting
```

**Advanced Structure (Professional):**
```
1. Frame & Aspect: aspect ratio, resolution, duration
2. Scene Summary: one sentence stating tone + action
3. Shot List/Beats: 0-3 numbered beats for 5-15s clips
4. Camera Directions: lens, movement, framing
5. Lighting & Color: time of day, mood, palette
6. Sound: audio type, voice, SFX, ambient
```

### Optimal Prompt Length

**Sweet Spot:** 50-100 words, multi-sentence descriptions that read like a film director's shot plan.

- **Too short** (under 30 words): "A cat playing piano" → lacks specificity, generic results
- **Too long** (over 150 words): Introduces conflicting instructions, confuses the model
- **Just right** (50-100 words): Clear directorial control with creative flexibility

### Core Prompting Principles

1. **One clear scene** + **one action** + **steady camera behavior**
2. **Emphasize 1-2 main elements** per prompt (don't overload)
3. **Use concrete, visible details** rather than abstract concepts
4. **Think like a storyboard artist** describing a shot
5. **Iterate and refine** - small changes dramatically shift results

### Key Prompt Components

A clear prompt describes a shot as if you were sketching it onto a storyboard:

1. **Camera framing** (wide shot, medium, close-up)
2. **Depth of field** (shallow, deep)
3. **Action in beats** (step-by-step movement)
4. **Lighting and palette** (golden hour, neon, cool tones)

### Cinematic Terminology to Include

**Lighting Descriptors:**
- "Golden hour backlighting"
- "Harsh overhead fluorescents"
- "Rim lighting"
- "Volumetric fog"
- "Practical lights" (visible light sources in scene)
- "Soft window light with warm lamp fill"
- "Cool edge from hallway"

**Pacing & Timing Words:**
- "Slow-motion"
- "Time-lapse"
- "Steady cam"
- "Whip pan"
- "Crescendo at 0:08 seconds"
- "Dialogue starts at 0:03"

**Camera Shots (Framing):**
- Wide shot / Establishing shot
- Medium shot
- Close-up / Extreme close-up
- Over-the-shoulder
- Dutch angle

**Camera Angles:**
- Low angle (subject appears powerful)
- High angle (subject appears small/vulnerable)
- Eye-level
- Bird's eye view

---

## Camera Movements & Cinematography

### Professional Camera Movement Terms

Use these exact terms for precise control:

- **Dolly:** Moving camera forward/backward
- **Track:** Moving camera sideways
- **Pan:** Turning camera horizontally
- **Tilt:** Turning camera vertically
- **Crane/Jib:** Moving camera up/down
- **Zoom:** Lens focal length change (not camera movement)
- **Steadicam:** Smooth, handheld-style movement
- **Arc:** Circular movement around subject
- **Roll:** Rotating camera on its axis
- **Whip pan:** Very fast horizontal pan
- **Handheld:** Intentionally shaky, documentary feel

### Best Practices for Camera Control

**1. One Movement Per Shot**
Specify ONE camera move and ONE subject action for smoother, predictable results.

**Example (Good):**
```
Slow dolly-in on a woman sipping coffee at a window,
soft morning light, steady cam
```

**Example (Bad):**
```
Camera dollies in while panning left, then tilts up,
then zooms out while tracking right
```

**2. Think Like a Cinematographer Brief**
When prompting, imagine you're briefing a camera operator who has never seen your storyboard. Leave nothing to improvisation.

**Example:**
```
Medium shot, 35mm lens, shallow depth of field.
Slow dolly-in from 5 feet to 2 feet.
Subject: man exhaling smoke.
Soft practical light from neon sign (pink/blue).
Handheld feel, slight drift.
```

**3. Reference Real Cinematography Styles**

You can specify established film styles:
- "IMAX aerial cinematography"
- "35mm handheld documentary style"
- "Vintage 16mm film grain"
- "Anamorphic widescreen with lens flares"
- "GoPro POV, wide-angle distortion"

### Multi-Shot Sequences

For videos with multiple camera setups, use shot blocks:

```
Shot 1: Wide shot, static camera. Woman walks into frame left.
Golden hour backlighting. Warm tones.

Shot 2: Medium close-up, slow dolly-in. She lights a joint.
Soft focus background. Cool blue edge light.

Shot 3: Close-up, handheld. Smoke exhale, head tilts back.
Volumetric smoke lit from behind. Slow-motion.
```

**Transition Cues:**
- "Cut to"
- "Fade in"
- "Crossfade"
- "Match cut"
- "Whip pan transition"
- "Smash cut"

### Balance Control vs. Creativity

- **Detailed prompts** = More control, consistency across shots
- **Lighter prompts** = More creative freedom, surprising variations

**When to use detailed prompts:**
- Brand commercials requiring specific look/feel
- Multi-shot sequences needing continuity
- Matching existing visual style

**When to use lighter prompts:**
- Exploratory/creative phase
- Seeking unexpected artistic results
- Testing new visual ideas

---

## Consistent Style Across Videos

### 1. Define Your Aesthetic Upfront

Establish visual style early with specific cues:

**Instead of:** "Brightly lit room"
**Use:** "Soft window light with warm lamp fill and cool edge from hallway. Pastel color palette: peach, mint, cream."

### 2. Keep Adjectives Aligned Across Shots

For multi-shot sequences, maintain consistent descriptive language.

**Example series:**
```
Shot 1: Soft pastel animation style, warm lighting, cozy atmosphere
Shot 2: Soft pastel animation style, warm lighting, cozy atmosphere
Shot 3: Soft pastel animation style, warm lighting, cozy atmosphere
```

### 3. Specify Lighting Logic and Color Palette

**Lighting Consistency:**
Describe both quality AND color of light sources:

```
Lighting: Diffuse natural light from large window (camera left),
warm tungsten practical lamp (background right),
cool ambient fill from blue hour sky.
```

**Color Palette:**
Name 3-5 specific colors to anchor the palette:

```
Palette: Deep teal, burnt orange, cream white, charcoal gray,
dusty rose accent.
```

### 4. Use Image References

**Option A: Upload Reference Images**
- Photos, digital art, or AI-generated visuals
- Locks in character design, wardrobe, set dressing, aesthetic

**Option B: Generate References First**
- Use OpenAI image generation to create environment/scene designs
- Pass images into Sora as visual references
- Test aesthetics before committing to video generation

### 5. Multi-Shot Sequence Structure

Detail each shot like a storyboard:

```
AESTHETIC SPINE (Apply to all shots):
- Style: Hand-painted 2D/3D hybrid animation
- Texture: Soft brush strokes, watercolor wash
- Lighting: Warm tungsten with cool edge light
- Palette: Teal, orange, cream, charcoal
- Mood: Cozy, tactile, mid-2000s storybook feel

Shot 1: [specific setup, action, camera]
Shot 2: [specific setup, action, camera]
Shot 3: [specific setup, action, camera]
```

### 6. Use Presets (Sora Built-In Styles)

Select presets to avoid rewriting style cues every time:

- **Cinematic:** Professional film look
- **Film Noir:** High contrast, dramatic shadows
- **Stop Motion:** Tactile, handcrafted feel
- **Whimsical Stop Motion:** Claymation, warm palette, storybook atmosphere
- **Vintage 16mm:** Grainy, nostalgic

### 7. Character Consistency with Cameo Feature

**Cameo** = Sora's character persistence system

When you invoke a cameo character:
- Sora creates a "character DNA profile"
- Physical characteristics remain consistent across shots
- Clothing, appearance, facial features stay stable
- Use for multi-shot sequences featuring the same person

**How to use:**
```
Character: Same woman from Shot 1 (cameo ID: #12345)
Maintains: Brown curly hair, round glasses, denim jacket
Action: Now sitting on bench, different location
```

### 8. Common Consistency Mistakes to Avoid

- **Vague style descriptions:** "Nice-looking video" → Be specific
- **Changing adjectives mid-sequence:** Maintain linguistic consistency
- **Overloading prompts:** Too many elements = disjointed results
- **Ignoring lighting logic:** Light sources should make sense together

---

## Short-Form Content (15-30 Seconds)

### Duration Settings

Sora duration options:
- **5 seconds** (counts as 1 video toward daily limit)
- **10 seconds** (counts as 1 video)
- **15 seconds** (counts as 2 videos)
- **25 seconds** (counts as 4 videos)

### Key Principle: Shorter Clips = Better Quality

**Best Practice:**
Generate two 4-second clips and stitch in editing, rather than one 8-second clip.

**Why:** Sora follows instructions more reliably in shorter durations.

### Short-Form Prompting Strategy

**1. One Clear Beat Per Shot**

For 15-30 second videos, limit to 1-3 distinct beats:

```
0-5 seconds: Woman walks to window, looks out
5-10 seconds: Lights joint, first inhale
10-15 seconds: Exhale, smoke swirls in backlight
```

**2. Vertical Composition (Social Media)**

The new Sora app favors short-form, vertical-friendly clips:

```
Aspect ratio: 9:16 (vertical)
Duration: 10 seconds
Hook: Fast, attention-grabbing opening (first 2 seconds)
```

**3. Fast Hooks for Feed Content**

Social media requires immediate engagement:

```
0-2 seconds: Striking visual or action (the "hook")
2-8 seconds: Main content/story beat
8-10 seconds: Payoff/resolution
```

### Prompt Structure for Short-Form

```
Aspect: 9:16 vertical, 10 seconds
Hook (0-2s): Close-up of hand lighting joint, flame illuminates face
Main (2-8s): Slow dolly-out to reveal cozy apartment, neon signs outside
Payoff (8-10s): Exhale, smoke drifts across frame, fade to logo
Camera: Handheld, intimate feel
Lighting: Warm practicals (candles, neon), moody shadows
Palette: Teal, orange, deep purple
Sound: Lofi hip-hop beat, soft crackling, ambient city
```

### Dialogue Tips for Short Clips

**Keep it concise:**
- Limit exchanges to a few sentences
- Match dialogue length to clip duration
- Label speakers consistently in multi-character scenes

**Format:**
```
VISUAL DESCRIPTION:
[Your prose description of the scene, camera, lighting]

DIALOGUE:
Character 1: "This is the best part of the day."
Character 2: "Always is."
```

### Iterative Approach for Short-Form

1. **Start with core intent** (what's the one thing this clip must communicate?)
2. **Add camera, motion, pacing** on next iteration
3. **Limit moving parts** - fewer characters, simpler motion = better fidelity
4. **Test fast** - Generate 3-5 low-quality variants to find the right direction
5. **Refine winner** - Polish the best variant with detail

---

## Character Actions & Transitions

### Character Action Best Practices

**1. Specific, Visible Actions**

**Instead of:** "Moves quickly"
**Use:** "Jogs three steps and stops at the curb"

**Instead of:** "Looks emotional"
**Use:** "Wipes tear from left cheek, looks down"

**2. Tie One Camera Move to One Character Action**

For clarity, pair movements:

```
Camera: Slow dolly-in from 6 feet to 3 feet
Character: Woman brings joint to lips, inhales slowly
```

**3. Include Character Details**

Describe fully for first mention:
- **Attire:** "Oversized hoodie, ripped jeans, white sneakers"
- **Emotional expression:** "Relaxed, slight smile, eyes half-closed"
- **Physical features:** "Shoulder-length wavy hair, nose ring, tattoo on left forearm"
- **Actions:** "Exhales smoke slowly, head tilts back, eyes close"

**4. Use Strong, Specific Verbs**

Weak verbs create vague results. Strong verbs create clarity.

| Weak Verb | Strong Verb |
|-----------|-------------|
| Goes | Walks, runs, drifts, stumbles |
| Looks | Stares, glances, scans, gazes |
| Moves | Slides, shifts, sways, lunges |
| Does something | Grabs, tosses, flips, taps |

**5. Structure Actions in Beats**

Break complex actions into simple beats:

```
Beat 1: Character reaches into pocket
Beat 2: Pulls out lighter
Beat 3: Flicks lighter (flame appears)
Beat 4: Brings flame to joint
Beat 5: First inhale (ember glows)
```

### Transitions Between Shots

**Transition Types:**

- **Cut:** Instant change between shots (default)
- **Crossfade:** Gradual dissolve from one shot to next
- **Hard cut:** Abrupt, intentional cut (for emphasis)
- **Match cut:** Visual element matches between shots
- **Whip pan:** Fast camera movement blurs between scenes
- **Smash cut:** Jarring transition for contrast/shock
- **Morph/Blend:** One scene gradually transforms into another

**How to Prompt Transitions:**

```
Shot 1: Close-up of smoke exhale
TRANSITION: Crossfade (1 second)
Shot 2: Smoke becomes clouds in sky, wide shot of city
```

**Advanced Blend Transitions:**

Sora allows blend curves for creative morphs:

```
BLEND TRANSITION (5 seconds):
Start: Astronaut walking on moon (100% influence, 0s)
Mid: Equal blend of moon and forest (50/50, 2.5s)
End: Dense forest scene (100% influence, 5s)
```

### Animation-Style Actions

For stylized/animated effects:

**Looney Tunes Style:**
```
Thick outlines, high-contrast colors, vintage Technicolor feel,
exaggerated squash-and-stretch, expressive reactions,
1930s-50s theatrical animation style
```

**Tom & Jerry Style:**
```
Hand-drawn cel animation, rich grayscale shadows,
1940s color film tones, exaggerated proportions,
fluid frame transitions, silent film pacing
```

**For "Head Melts on Exhale" Effect:**
```
Character exhales smoke. As smoke exits mouth,
head begins to distort and melt downward like wax,
exaggerated cartoon physics, surreal lofi aesthetic,
slow-motion, soft pastel colors, dreamlike atmosphere.
```

### Combining Camera Movement + Character Action + Transition

**Full Example:**
```
Shot 1:
Camera: Static medium shot, 35mm lens
Character: Man sits on couch, reaches for joint on table
Lighting: Soft lamp light, cool window backlight
Duration: 4 seconds

TRANSITION: Match cut (flame)

Shot 2:
Camera: Extreme close-up, handheld
Character: Lighter flame ignites joint, ember glows
Lighting: Practical flame illuminates face from below
Duration: 3 seconds

TRANSITION: Slow dolly-out crossfade

Shot 3:
Camera: Wide shot, slow dolly-out reveals full room
Character: Exhales, smoke swirls across frame
Lighting: Neon signs outside window, volumetric smoke
Duration: 5 seconds
```

---

## Lofi/Chill Aesthetic Techniques

### Core Lofi/Chill Visual Language

**Style Descriptors:**
- "Hand-painted 2D/3D hybrid animation"
- "Soft brush textures, watercolor wash"
- "Warm tungsten lighting with cool edge highlights"
- "Tactile, stop-motion feel"
- "Cozy, imperfect, mechanical charm"
- "Mid-2000s storybook animation aesthetic"
- "Subtle grain, vintage film texture"
- "Muted pastel palette"

**Lighting for Lofi Vibes:**
- Warm practicals (lamps, candles, string lights)
- Soft window light (diffused, gentle)
- Neon signs (pink, blue, teal, purple)
- Golden hour backlighting
- Cool ambient fill from dusk/night sky
- Volumetric smoke/fog lit from behind

**Color Palettes for Lofi/Chill:**

**Palette 1 (Warm Cozy):**
- Deep teal
- Burnt orange
- Cream white
- Charcoal gray
- Dusty rose accent

**Palette 2 (Urban Night):**
- Neon pink
- Electric blue
- Deep purple
- Warm amber
- Matte black

**Palette 3 (Natural Chill):**
- Sage green
- Terracotta
- Off-white
- Warm beige
- Soft lavender

### Lofi Prompt Formula

```
AESTHETIC:
Style: [Hand-painted / Stop-motion / Vintage film / Pastel animation]
Texture: [Soft brush strokes / Grainy / Watercolor / Tactile]
Mood: [Cozy / Dreamy / Nostalgic / Meditative / Mellow]

SCENE:
Setting: [Describe intimate, relatable space]
Character: [Simple, relatable action]
Lighting: [Warm practicals + cool ambient]
Palette: [3-5 muted, harmonious colors]

CAMERA:
Movement: [Slow, steady, or static]
Framing: [Medium shots, close-ups - intimate feel]
Feel: [Handheld drift / Locked-off / Gentle dolly]

SOUND:
Music: [Lofi hip-hop / Soft piano / Ambient electronica]
Ambient: [Rain / City sounds / Vinyl crackle / Wind chimes]
SFX: [Subtle, organic, ASMR-quality]
```

### Example Lofi Prompts

**Example 1: Urban Lofi Chill**
```
Aspect: 16:9, 10 seconds
Style: Soft pastel animation, hand-painted textures, cozy aesthetic
Scene: Woman sitting by apartment window at dusk, sketchbook in lap,
       smoking joint, city lights blinking on outside
Character: Relaxed posture, oversized sweater, messy bun, peaceful expression
Action: Slow exhale, smoke drifts across window glass, she smiles softly
Camera: Static medium shot, shallow depth of field
Lighting: Warm lamp (camera left), cool blue hour window light (camera right),
          soft neon glow from city outside
Palette: Teal, burnt orange, cream, charcoal, dusty pink
Sound: Lofi hip-hop beat, distant city ambience, soft rain on window
Mood: Meditative, nostalgic, peaceful
```

**Example 2: Stoner Cozy Vibe**
```
Aspect: 9:16 vertical, 15 seconds
Style: Vintage 16mm film grain, warm tones, tactile stop-motion feel
Scene: First-person POV, hands rolling joint on wooden coffee table,
       plants and candles in soft focus background
Action:
  0-5s: Hands carefully roll joint, close-up, slow and deliberate
  5-10s: Light joint, flame illuminates hands
  10-15s: Exhale toward camera, smoke swirls in candlelight
Camera: POV close-up, locked-off, intimate framing
Lighting: Warm candlelight (multiple sources), soft and flickering,
          golden ambient glow
Palette: Honey gold, deep green, warm brown, cream, soft amber
Sound: Soft acoustic guitar, vinyl crackle, gentle rustling,
       lighter flick, slow exhale
Mood: Intimate, slow, meditative, cozy
```

**Example 3: Creative Stoner Aesthetic**
```
Aspect: 1:1 square, 20 seconds
Style: Hand-drawn animation hybrid, surreal, dreamlike, lofi illustration style
Scene: Character's head becomes clouds as they exhale smoke,
       melts and reforms, sitting in bean bag chair, record player nearby
Action:
  0-7s: Character takes slow drag, eyes close peacefully
  7-14s: Exhale begins, head distorts and becomes swirling smoke/clouds
  14-20s: Head reforms, character opens eyes with slight smile
Camera: Medium close-up, slow dolly-in from 4ft to 2ft
Lighting: Soft practical lamps, purple and pink neon from off-screen,
          volumetric smoke backlit
Palette: Lavender, mint green, peach, cream, deep purple
Sound: Dreamy synth pad, soft lofi beat, ethereal whoosh during transformation
Mood: Surreal, chill, creative, playful, stoner-friendly
```

### Lofi Scene Settings (High-Performing)

These settings resonate with lofi/chill aesthetic:

- Bedroom with string lights and plants
- Balcony at sunset overlooking city
- Coffee shop window seat, rainy day outside
- Rooftop at night with city lights
- Record store, browsing vinyl
- Art studio with natural light
- Cozy living room, fireplace or candles
- Late-night kitchen, making tea/snacks
- Vintage car interior, parked with city view
- Park bench under tree, golden hour

### Lofi Camera Techniques

**Preferred Movements:**
- Slow, steady dolly-in or dolly-out
- Gentle pan across scene
- Locked-off (static) for meditative moments
- Handheld with subtle drift (not shaky)

**Avoid:**
- Fast whip pans
- Aggressive zooms
- Shaky handheld
- Complex multi-axis moves

**Framing:**
- Medium shots and close-ups (intimate)
- Rule of thirds for balanced composition
- Negative space for breathing room
- Shallow depth of field (soft backgrounds)

### Lofi Sound Design

**Music Genres:**
- Lofi hip-hop
- Ambient electronica
- Soft jazz piano
- Chillwave
- Downtempo beats
- Acoustic guitar

**Ambient Sounds:**
- Soft rain
- Distant city traffic
- Vinyl crackle
- Wind chimes
- Rustling leaves
- Café chatter (low, distant)

**SFX:**
- Lighter flick
- Slow exhale
- Pages turning
- Coffee pouring
- Pen on paper
- Gentle footsteps

---

## Common Mistakes to Avoid

### 1. Being Too Vague

**Bad:**
```
A nice video of nature
```

**Good:**
```
A serene forest stream with sunlight filtering through autumn leaves,
peaceful ambient mood, slow dolly forward along the water's edge,
golden hour lighting, warm color palette
```

**Why it matters:** Vague prompts give Sora too little direction, resulting in generic, unpredictable outputs.

---

### 2. Overcomplicating (Too Many Elements)

**Bad:**
```
A woman walks into a café, orders coffee, sits down, opens laptop,
types, drinks coffee, looks out window, answers phone, waves to friend,
pays bill, and leaves, all while camera pans, zooms, tilts, and dollies
```

**Good:**
```
A woman walks into a café, orders coffee at counter,
medium shot, steady cam, warm afternoon lighting
```

**Why it matters:** Cramming too many actions, characters, or camera moves confuses the model and creates disjointed results.

---

### 3. Motion Issues (Complex Camera Moves)

**Bad:**
```
Camera dollies in while panning left, then tilts up, zooms out,
and whip pans to the right
```

**Good:**
```
Slow dolly-in on subject, steady and smooth,
one continuous movement from 6 feet to 3 feet
```

**Why it matters:** Sora handles single, simple camera movements much better than multi-axis choreography.

---

### 4. Ignoring Lighting Logic

**Bad:**
```
Bright sunny day with moonlight and candles
```

**Good:**
```
Golden hour sunlight from large window (camera left),
warm tungsten lamp (background right),
soft ambient fill from sky
```

**Why it matters:** Conflicting light sources break realism and make scenes look amateurish.

---

### 5. Inconsistent Style Across Shots

**Bad:**
```
Shot 1: Cinematic film noir style
Shot 2: Bright cartoon animation
Shot 3: Vintage documentary grain
```

**Good:**
```
All shots: Soft pastel animation, warm tungsten lighting,
cozy mid-2000s storybook aesthetic
```

**Why it matters:** Visual continuity is critical for multi-shot sequences and brand consistency.

---

### 6. Requesting Legible Text

**Bad:**
```
Sign on building reads "Joe's Coffee Shop" in clear letters
```

**Good:**
```
Neon sign glowing outside café (text abstracted),
pink and blue tones, slightly blurred in background
```

**Why it matters:** Sora struggles with readable text. Add text overlays in post-production instead.

---

### 7. Expecting Perfect Physics

**Bad:**
```
Water splashes realistically as character dives into pool
```

**Good:**
```
Character dives into pool, water disturbed, slow-motion,
stylized splash (don't expect photorealistic fluid dynamics)
```

**Why it matters:** Sora can produce physics glitches. Set expectations appropriately or embrace stylization.

---

### 8. Lip-Sync Expectations

**Bad:**
```
Character delivers 30-second monologue with perfect lip-sync
```

**Good:**
```
Character speaks 1-2 short lines ("This is perfect." "Always is."),
or use ADR/voiceover in post-production
```

**Why it matters:** Sora's lip-sync is improving but not perfect. Keep dialogue minimal or plan for post-production fixes.

---

### 9. Not Iterating

**Bad:**
```
Generate one video, accept whatever result comes back
```

**Good:**
```
Generate 3-5 low-res variants, review, select best direction,
refine prompt, generate high-res final
```

**Why it matters:** Small changes to camera, lighting, or action dramatically shift outcomes. Iteration is essential.

---

### 10. Forgetting Sound Design

**Bad:**
```
[Only describe visuals, no mention of audio]
```

**Good:**
```
Sound: Lofi hip-hop beat, soft rain ambience, lighter flick at 0:03,
slow exhale at 0:08, vinyl crackle throughout
```

**Why it matters:** Audio cues guide pacing and enhance immersion. Always include sound descriptions.

---

## Example Prompts

### Example 1: KannaKlaus Brand Intro (15 seconds)

```
Aspect: 16:9, 15 seconds
Style: Hand-painted animation, soft pastel aesthetic, cozy holiday vibes

SHOT BREAKDOWN:
0-5s: Close-up of hand holding joint decorated with tiny Santa hat,
      flame from lighter illuminates the scene
5-10s: Slow dolly-out reveals character (KannaKlaus-inspired,
       red hoodie, Santa hat, friendly smile) sitting by Christmas tree
       with purple and teal lights
10-15s: Character exhales, smoke swirls and forms "KannaKrew" logo in air,
        then fades to clean logo screen

Camera: Starts extreme close-up, slow dolly-out to medium shot
Lighting: Warm practical lights (Christmas tree, candles),
          soft window light with blue hour tones outside
Palette: Deep teal, burnt orange, cream, charcoal, dusty rose, holiday red accent
Sound: Lofi holiday beat, soft bells, lighter flick, gentle exhale,
       ambient fireplace crackle

Mood: Festive, cozy, welcoming, chill, brand-forward
```

---

### Example 2: Donation Box Callout (10 seconds, Vertical)

```
Aspect: 9:16 vertical, 10 seconds
Style: Vintage 16mm film grain, warm nostalgic tones

SHOT:
POV walking toward donation box on dispensary counter,
colorful wrapped toys visible inside box,
"KannaKickback 6 - Donate a Toy" sign on front,
hand enters frame and places wrapped gift in box

Camera: Handheld POV, slow approach from 5 feet to 1 foot
Lighting: Warm indoor retail lighting, soft overhead fluorescents with warm gel,
          practical backlighting from window
Palette: Honey gold, deep green, warm brown, cream, holiday red
Sound: Soft acoustic guitar, ambient dispensary chatter (distant),
       rustling wrapping paper, gentle placement of gift

Mood: Inviting, community-driven, heartwarming, call-to-action
```

---

### Example 3: Stoner Character Transformation (20 seconds)

```
Aspect: 1:1 square, 20 seconds
Style: Surreal hand-drawn animation hybrid, dreamlike lofi illustration

SHOT BREAKDOWN:
0-7s: Character sits cross-legged in bean bag chair,
      surrounded by plants and candles, takes slow drag from joint,
      eyes close peacefully
7-14s: Exhale begins, character's head distorts and becomes swirling
       smoke and clouds, melting upward like vapor,
       exaggerated cartoon physics, pastel colors
14-20s: Head reforms from smoke, character opens eyes with slight smile,
        record player needle drops in background,
        smoke clears to reveal cozy room

Camera: Locked-off medium close-up for first 10 seconds,
        then slow dolly-in to close-up as head reforms
Lighting: Soft practical lamps on both sides,
          purple and pink neon glow from off-screen right,
          volumetric smoke backlit by warm amber lamp
Palette: Lavender, mint green, peach, cream, deep purple
Sound: Dreamy synth pad, soft lofi beat, ethereal whoosh during transformation,
       vinyl crackle, record needle drop at end

Mood: Surreal, creative, playful, stoner-friendly, imaginative
```

---

### Example 4: Urban Night Chill (15 seconds)

```
Aspect: 16:9, 15 seconds
Style: Cinematic with subtle film grain, moody urban aesthetic

SHOT:
Rooftop at night, city skyline glowing in background,
character sits on edge with legs dangling, joint in hand,
looks out over city, takes drag, exhales slowly toward camera,
smoke drifts across city lights

Camera: Wide shot transitioning to medium shot via slow dolly-in,
        steady and smooth
Lighting: Cool blue ambient from city lights below,
          warm practical from string lights on rooftop,
          neon pink and teal reflections from distant signs
Palette: Electric blue, neon pink, deep purple, warm amber, matte black
Sound: Lofi hip-hop beat, distant city traffic, wind rustling,
       lighter flick, slow exhale

Mood: Contemplative, urban, peaceful, cinematic, late-night vibes
```

---

### Example 5: Product Focus - Lofi Aesthetic (8 seconds)

```
Aspect: 9:16 vertical, 8 seconds
Style: Stop-motion tactile feel, warm cozy aesthetic

SHOT:
Extreme close-up of hands rolling joint on wooden surface,
grinder and papers nearby, soft focus plants in background,
slow and deliberate movements, artisan craft vibe

Camera: Locked-off overhead angle, shallow depth of field
Lighting: Warm tungsten practical lamp from top-left,
          soft fill from window (diffused, gentle),
          golden hour tones
Palette: Honey gold, deep green, warm brown, cream, soft amber
Sound: Soft acoustic guitar, paper rustling, gentle ambient room tone,
       ASMR-quality foley

Mood: Intimate, artisan, meditative, detail-oriented, ASMR-friendly
```

---

### Example 6: Group Hangout - Social Vibe (12 seconds)

```
Aspect: 16:9, 12 seconds
Style: Handheld documentary style, warm nostalgic film tones

SHOT:
Living room, 3-4 friends sitting on couch and floor,
passing joint in circle, laughing, comfortable and relaxed,
string lights and plants in background

Camera: Handheld medium-wide shot, subtle drift and sway,
        intimate documentary feel
Lighting: Warm string lights overhead, soft lamp in corner,
          cool window backlight from night outside
Palette: Warm amber, teal accent, cream, charcoal, dusty rose
Sound: Lofi hip-hop beat, overlapping laughter, soft conversation (indistinct),
       lighter flick, ambient room tone

Mood: Social, friendly, inclusive, community, good vibes
```

---

### Example 7: Nature Chill - Outdoor Vibe (10 seconds)

```
Aspect: 16:9, 10 seconds
Style: Cinematic with soft pastel grading, dreamy outdoor aesthetic

SHOT:
Park bench under tree, golden hour light filtering through leaves,
character sits alone, smoking joint, looks peacefully at sunset,
smoke drifts upward through beams of light

Camera: Locked-off wide shot, shallow depth of field,
        leaves gently swaying in foreground (soft focus)
Lighting: Golden hour backlighting through tree,
          rim light on character's hair,
          soft warm fill from reflected sunset
Palette: Golden yellow, sage green, soft peach, cream, dusty rose
Sound: Soft acoustic guitar, birds chirping, gentle wind rustling leaves,
       slow exhale, peaceful ambiance

Mood: Peaceful, contemplative, nature-connected, serene, solitary
```

---

## Quick Reference Cheat Sheet

### Prompt Template (Copy & Paste)

```
ASPECT: [16:9 / 9:16 / 1:1], [duration] seconds
STYLE: [Cinematic / Lofi animation / Stop-motion / Vintage film / Hand-painted]

SCENE DESCRIPTION:
[Setting, character, props, environment]

ACTION BEATS:
0-Xs: [What happens]
X-Ys: [What happens next]
Y-Zs: [Final action/payoff]

CAMERA:
Movement: [Dolly / Pan / Static / Handheld]
Framing: [Wide / Medium / Close-up]
Lens: [35mm / 50mm / Wide-angle]

LIGHTING:
Source 1: [Type, direction, color]
Source 2: [Type, direction, color]
Ambient: [Quality and tone]

PALETTE:
[Color 1, Color 2, Color 3, Color 4, Color 5]

SOUND:
Music: [Genre/style]
Ambient: [Background sounds]
SFX: [Specific sound effects with timing]

MOOD:
[2-3 adjectives describing emotional tone]
```

---

## Workflow Summary

### Pre-Production
1. Script a beat sheet (what happens when)
2. Storyboard (camera, framing, lighting for each shot)
3. Define "style spine" (consistent aesthetic language)

### Low-Res Exploration
1. Generate 3-5 short, low-quality variants per shot
2. Test different camera, lighting, action combinations
3. Select best direction from variants

### Refinement
1. Refine winning prompts with added detail
2. Generate high-quality versions
3. Iterate on small changes (lighting, timing, framing)

### Post-Production
1. Stitch clips together in editing software
2. Add text overlays (never rely on AI-generated text)
3. Add/replace audio (ADR for dialogue, sound design)
4. Color grading for final polish

---

## Additional Resources

### Official Documentation
- [Sora 2 Prompting Guide | OpenAI Cookbook](https://cookbook.openai.com/examples/sora/sora2_prompting_guide)
- [Sora | Prompt Engineering Guide](https://www.promptingguide.ai/models/sora)

### Comprehensive Guides
- [Crafting Cinematic Sora Video Prompts: A complete guide · GitHub](https://gist.github.com/ruvnet/e20537eb50866b2d837d4d13b066bd88)
- [Sora 2 Prompt Authoring Best Practices 2025: Ultimate Guide - Atlabs AI](https://www.atlabs.ai/blog/sora-2-prompt-guide)
- [Complete Sora AI Video Generation Guide 2025](https://www.soraainow.com/en/blog/complete-guide-sora-ai-video-generation-2025)

### Camera & Cinematography
- [Master Sora Prompts: 6 steps Guide to Perfect AI Videos](https://filmart.ai/guide-to-sora-prompts/)
- [Sora 2 Control Camera Angles with the Right Prompt Structure](https://www.freejobalert.com/article/sora-2-control-camera-angles-prompt-structure-cinematic-shots-23865)

### Consistency & Style
- [Create Consistent, Seamless Shots of the Same Person in Sora AI](https://www.aibase.tech/news/features/create-consistent-seamless-shots-of-the-same-person-in-sora-ai/)
- [Sora 2 Cameo Feature: Character Consistency Guide](https://www.portotheme.com/sora-2-cameo-feature-the-secret-to-character-consistency-in-ai-video-creation/)

### Short-Form & Social Content
- [12 Essential Sora 2 Prompting Tips for Video Creators (2025)](https://skywork.ai/blog/sora-2-prompting-tips-2025/)
- [Best Sora Prompts for Viral Videos](https://travisnicholson.medium.com/best-sora-prompts-for-viral-videos-074eedb80ddf)

### Creative Techniques
- [3 Ways to Prompt Movies with Sora 2](https://www.whytryai.com/p/ways-to-prompt-sora-2-movies)
- [10 Fun Things You Can Try with Sora 2 Right Now (2025)](https://skywork.ai/blog/sora-2-tips-2025/)

---

## Final Tips

1. **Start simple, add complexity** - Begin with core intent, layer in detail
2. **One shot, one action, one camera move** - Simplicity wins
3. **Iterate relentlessly** - Small prompt changes = big output differences
4. **Use reference images** - Visual anchors improve consistency
5. **Think like a director** - Brief your "AI crew" as you would humans
6. **Embrace creative accidents** - Sometimes the AI surprises you beautifully
7. **Plan for post-production** - Stitch clips, add text/audio later
8. **Test fast, refine slow** - Low-res exploration → high-res polish

**Remember:** Sora is a creative partner, not a mind reader. The more specific your vision, the better your results.

---

**Generated:** 2025-12-02
**Research Time:** 1.2 minutes
**Completeness:** 90%

This guide is ready for immediate use in creating thematic brand commercials with lofi/chill/stoner aesthetics for KannaKickback and KannaKrew.