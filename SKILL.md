---
name: dine-kv
description: Iterative creative exploration for generating brand-compliant marketing and operational images using AI. Use this skill whenever the user wants to create KV designs, marketing visuals, operational graphics, event posters, or any brand-compliant image assets. Also trigger when the user provides brand rules together with image generation needs, mentions "creative exploration", asks to iterate on generated visuals, or wants to brainstorm visual directions for a campaign. Works with any brand that has a rules document.
---

# Creative Exploration

Generate brand-compliant marketing and operational images through iterative creative exploration.

## Conversation Style

Be concise, friendly, and professional. Don't over-explain. Guide the user forward with short, clear questions at natural decision points: "Which directions do you want to take further?" / "Want to try with reference images this time?"

## Setup

On the first use in a session:

1. **Workspace path** — Ask: "Is the current directory your project workspace? If not, please tell me the path."
2. **Brand rules** — Look for a brand rules markdown file (e.g. `*-Brand-Rules.md`). Read it thoroughly — colors, typography, visual language, constraints, and any AI prompt guidelines.
3. **Brand assets** — Scan the `assets/` directory. Read each image to understand what's available (patterns, icons, logos, illustrations). Build a mental inventory — you'll use these automatically later.
4. **Brand colors** — Extract the primary brand colors from the rules for use in the HTML preview.
5. **Task requirements** — Ask the user for the brief. Also let them know: "You can also share a task requirements doc (Feishu/Lark URL works), reference images, or any style notes."

If the user provides a Feishu/Lark wiki URL, extract `document_id` from `larksuite.com/wiki/{document_id}` and read with `mcp__claude_ai_Feishu__read_document_content`.

## Workflow

### Round 1: Divergent Exploration

The first round is about **breadth and inspiration**.

1. **Brainstorm 7+ creative directions** as text. Each needs:
   - A short bilingual name (e.g. "Tessellation Wave (镶嵌波)")
   - A 1-2 sentence concept description
   - How it connects to the brief's keywords

   Push beyond the obvious. Consider metaphors, abstract interpretations, cultural references, unexpected visual approaches.

2. **Vary reference inputs across directions** — maximize divergence by using different reference combos. Adapt the allocation based on what's available:

   **With user refs + brand assets (typical case, 7 directions):**
   - ~3: user ref + different brand asset each (Pattern 01, 03, Icons, etc.) → edit mode
   - ~2: user ref only OR brand asset only → edit mode
   - ~2: no references at all → text-to-image mode

   **With user refs but no brand assets:**
   - ~4: user ref → edit mode
   - ~3: no references → text-to-image mode

   **With brand assets but no user refs:**
   - ~4: different brand asset each → edit mode
   - ~3: no references → text-to-image mode

   **Multiple user refs (2+):** Reduce brand asset usage — the user's refs already provide variety. Mix different user refs across directions. Use ~1-2 pure text-to-image for baseline.

   **No refs and no assets:** All text-to-image mode.

3. **Generate quick drafts** using `nano-banana-2`. Use edit mode for directions with references, `generate.sh` for pure text-to-image. One image per direction. Run all in parallel.

4. **MUST: Present in HTML preview** — after generating images, ALWAYS create a self-contained HTML preview file and open it in the browser. This is not optional. Use the inline approach described in the HTML Preview section below.

5. **Archive** the round.

6. **Summarize** what reference combos were used for each direction, then ask: "Which directions to explore further? And for next round, do you want more/fewer reference-guided vs pure-prompt directions?"

### Subsequent Rounds: Converge or Diverge

Based on user feedback:

- **Diverge further**: New directions, variations, or mashups of what worked
- **Converge**: Focus on selected directions, refine prompts. Suggest upgrading model for quality — "These directions look solid. Want to try GPT Image or Nano Banana Pro for higher quality?"

## Model Registry

When the user mentions a model by casual name, map it to the correct fal.ai model ID:

| User says | Model ID | Size values | Notes |
|---|---|---|---|
| "nano banana", "nb2", "fast" | `fal-ai/nano-banana-2` | `landscape_4_3`, `square`, `portrait_4_3` | Default for Round 1 (fast, cheap) |
| "nano banana pro", "nb pro", "pro" | `fal-ai/nano-banana-pro` | `landscape_4_3`, `square`, `portrait_4_3` | Higher fidelity |
| "gpt", "gpt image", "openai" | `fal-ai/gpt-image-1.5` | `1536x1024`, `1024x1024`, `1024x1536` | Best detail/composition |
| "edit", "with reference" | `fal-ai/nano-banana-2/edit` | `aspect_ratio`: "16:9", "4:3", "1:1" | Style/element reference mode |

If a new model version exists (e.g. user says "latest gpt"), use `search-models.sh` to look it up:
```bash
bash ~/.claude/skills/fal-generate/scripts/search-models.sh --category "text-to-image"
```

## Reference Image Handling

Users may provide one or more reference images. Each image can serve a different purpose — don't assume they're all style references.

### Determine reference intent

If the user doesn't specify how to use their reference images, make a smart judgment based on the image content:

- **Brand pattern / texture / abstract graphic** → Style reference (color palette, visual language)
- **Photo / scene / composition** → Composition or mood reference
- **Logo / mascot / specific object** → Element to embed in the composition
- **Another design / poster** → Layout and style reference

If ambiguous, ask briefly: "Image A looks like a style reference, image B is a mascot — should I use A for style and embed B into the composition?"

### Prompt patterns for each reference type

**Style reference** — extract visual language, don't reproduce:
```
Use the attached image ONLY as style reference for [color palette / geometric language / texture style]. Do NOT reproduce it. Create a new original composition: [prompt]
```

**Element / object to embed** — incorporate the specific object:
```
The attached image contains a [mascot / logo / product]. Incorporate this [element] prominently into the composition: [prompt]
```

**Composition / layout reference** — follow structure:
```
Use the attached image as a layout reference. Follow its general composition and spatial arrangement, but create original visuals in the brand style: [prompt]
```

**Mixed references** — when multiple images serve different roles, describe each:
```
Image 1 is a style reference for color palette and triangular geometric language — extract the style but do NOT copy it. Image 2 contains a mascot character — embed it into the scene. Create a new composition: [prompt]
```

### Technical rules for edit mode

- **Sync only**: Use `fal.run`, never `queue.fal.run` — queue returns HTTP 405 for edit endpoints
- **Set aspect_ratio explicitly**: Default is 1:1, always specify (e.g. "16:9", "4:3")
- Upload local files first with `upload.sh`, cache CDN URLs for the session

```bash
# Upload
bash ~/.claude/skills/fal-generate/scripts/upload.sh --file "/path/to/image.png"

# Generate with references
curl -s -X POST "https://fal.run/fal-ai/nano-banana-2/edit" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[reference instructions + actual prompt]",
    "image_urls": ["https://...ref1.png", "https://...ref2.png"],
    "aspect_ratio": "16:9",
    "resolution": "1K",
    "num_images": 1
  }'
```

## Text-to-Image (no references)

```bash
bash ~/.claude/skills/fal-generate/scripts/generate.sh \
  --prompt "..." \
  --model "fal-ai/nano-banana-2" \
  --size landscape
```

## FAL_KEY Setup

`$FAL_KEY` should be set in the user's shell profile. If missing, guide:
```
You need a fal.ai API key:
1. Sign up at https://fal.ai
2. Add to ~/.zshrc: export FAL_KEY="your_key_here"
3. Run: source ~/.zshrc
```

## HTML Preview

**IMPORTANT**: Always generate the HTML preview after each round of image generation. This is a core deliverable, not an optional step.

Write a single self-contained HTML file with DATA inlined — no external data.js, no server.

There are two modes depending on the environment:

| Environment | Image strategy | How to open |
|---|---|---|
| **Normal (CLI / terminal)** | Relative file paths | `open preview.html` |
| **Claude cowork (sidebar sandbox)** | Base64 data URI inline | Write the HTML file, it renders in the sidebar preview |

**How to detect**: If the user is working in Claude cowork / the conversation has a sidebar preview panel, use the cowork mode. Otherwise use normal mode.

### Normal mode (CLI)

1. Download all generated images (and input reference images) to the round's archive directory with `curl -o`
2. Write `preview.html` in the same directory, using **relative paths** for all images
3. Open it: `open "{path}/preview.html"`

#### Image file naming

```
round1/
  d1-direction-name.png      # generated image for direction 1
  d2-direction-name.png      # generated image for direction 2
  ref-user-style.jpg          # input reference image
  ref-pattern-01.png          # input reference image
  preview.html                # self-contained preview
```

#### DATA example (relative paths)

```javascript
directions: [{
  image: "d1-direction-name.png",
  inputs: [
    { url: "ref-user-style.jpg", label: "User ref: style" },
    { url: "ref-pattern-01.png", label: "Pattern 01" }
  ],
  // ...
}]
```

### Cowork mode (sidebar sandbox)

The sidebar sandbox cannot access local files. Embed all images as base64 data URIs.

1. Download images to the archive directory (same as normal mode)
2. Convert each image to base64 data URI: `base64 -i image.png | tr -d '\n'`
3. Use `data:image/png;base64,...` URLs in the DATA object
4. Write `preview.html` — it will render directly in the sidebar

#### DATA example (base64)

```javascript
directions: [{
  image: "data:image/png;base64,iVBORw0KGgo...",
  inputs: [
    { url: "data:image/jpeg;base64,/9j/4AAQ...", label: "User ref: style" },
    { url: "data:image/png;base64,iVBORw0K...", label: "Pattern 01" }
  ],
  // ...
}]
```

Tip: run all `base64` conversions in parallel for speed.

### How to write the file

Take the template from `references/preview-template.html` and inline the DATA object:

```html
<!-- copy the <head> and <style> from preview-template.html, then: -->
<body>
<div id="app"></div>
<script>
const DATA = {
  title: "Brand - Task Title",
  subtitle: "Round N | context",
  accent: "#935BFF",
  colors: ["#935BFF", "#C4B5FF", "#D8D0FF"],
  directions: [
    {
      name: "1. Direction Name (中文名)",
      meta: "edit mode | User ref + Pattern 01",
      desc: "One sentence concept description.",
      image: "d1-direction-name.png",  // or data:image/png;base64,... in cowork mode
      model: "nano-banana-2/edit",
      inputs: [
        { url: "ref-user-style.jpg", label: "User ref: style" },
        { url: "ref-pattern-01.png", label: "Pattern 01" }
      ],
      prompt: "The full prompt used..."
    }
  ]
};
</script>
<script>
/* rendering code from preview-template.html */
const d=DATA,a=d.accent||'#666';
// ... (copy the rest of the rendering script from the template)
</script>
</body>
```

For directions with no reference images, set `inputs: []` and `model: "nano-banana-2 (text-to-image)"`.

See `references/preview-template.html` for the full template and rendering script.

## Archive

The round directory IS the archive — images are already downloaded there with the preview HTML:

```
{workspace}/{brand or project name}/{yyyy-mm-dd}-{task title}/round{N}/
```

## Prompt Writing Patterns

Adapt to the brand's visual language (read the brand rules first):

- **Context first**: "Abstract [concept] for a [context], [background description]"
- **Brand colors by hex**: Models respond well to specific hex codes
- **Constraints at the end**: "No text, no logos, no people. [mood adjectives]."
- **Layout**: "Left side with generous whitespace for text placement"
- **Bilingual names**: Help multilingual teams — "Gateway (门户)", "Constellation (星座)"

Common pitfalls:
- Not using available brand assets as references — always check the assets directory first
- Forgetting to specify background lightness — many models default to dark
- Over-describing — shorter, focused prompts often work better
