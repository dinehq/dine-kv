# Creative Explore

A Claude Code skill for iterative creative exploration — generating brand-compliant marketing and operational images using AI image models.

## What It Does

1. **Reads your brand** — brand rules (.md), assets (patterns, icons), and task requirements
2. **Brainstorms 7+ creative directions** — bilingual names, varied reference combos, divergent concepts
3. **Generates images via fal.ai** — text-to-image and style-reference (edit) modes, all in parallel
4. **Presents an HTML preview** — data-driven template with images, prompts, and reference thumbnails
5. **Iterates** — user picks favorites, Claude refines prompts or explores new directions
6. **Archives** — each round saved to workspace with downloaded images

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) or [Claude Code via Codex](https://docs.anthropic.com/en/docs/claude-code)
- [fal-generate](https://github.com/imneway/fal-generate) skill (same install method below)
- fal.ai API key in your shell profile:
  ```bash
  # Add to ~/.zshrc or ~/.bashrc
  export FAL_KEY="your_key_here"
  ```

## Install

Clone into your Claude skills directory:

```bash
# Default skills path (works with both Claude Code and Codex)
mkdir -p ~/.claude/skills
git clone https://github.com/imneway/creative-explore.git ~/.claude/skills/creative-explore
git clone https://github.com/imneway/fal-generate.git ~/.claude/skills/fal-generate
```

Or, if your project uses a custom skills path, clone into that directory instead.

After installing, restart Claude Code / Codex for the skill to take effect.

## Usage

The skill triggers automatically when you:
- Ask to create KV designs, marketing visuals, event posters, or operational graphics
- Provide brand rules together with image generation needs
- Mention "creative exploration" or ask to brainstorm visual directions

### Quick Start

```
> I need a conference stage backdrop for our Web3 event.
  Here are our brand rules: [path or Feishu URL]
  Reference image attached — use it for style only.
  Keywords: Asia, Connection
```

Claude will:
1. Read your brand rules and scan available assets
2. Brainstorm 7+ directions with varied reference combos
3. Generate draft images in parallel (fast model)
4. Open an HTML preview at `localhost:8899`
5. Ask which directions to take further

### Workspace Structure

```
workspace/
├── Brand-Rules.md
├── assets/           # Brand patterns, icons, logos
│   ├── Pattern-01.png
│   ├── Pattern-03.png
│   └── Icons.png
└── 2025-03-12-event-backdrop/
    ├── round1/       # Archived rounds
    ├── round2/
    └── ...
```

## Supported Models

| Casual Name | Model | Mode |
|---|---|---|
| nano banana / nb2 | `fal-ai/nano-banana-2` | Text-to-image (default, fast) |
| nano banana pro | `fal-ai/nano-banana-pro` | Text-to-image (higher fidelity) |
| gpt / gpt image | `fal-ai/gpt-image-1.5` | Text-to-image (best detail) |
| edit / with reference | `fal-ai/nano-banana-2/edit` | Style reference / edit mode |

## How Reference Images Work

The skill automatically detects how to use reference images:

- **Brand pattern / texture** → Style reference (extract visual language, don't reproduce)
- **Photo / scene** → Composition or mood reference
- **Logo / mascot** → Element to embed in the composition
- **Another design / poster** → Layout and style reference

In Round 1, references are distributed across directions to maximize divergence — some directions use reference images, others are pure text-to-image for comparison.

## License

MIT
