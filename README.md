# Obsidian Visual Skills Pack

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status: Experimental](https://img.shields.io/badge/Status-Experimental-orange.svg)](#status)

Claude Code skills that generate **Excalidraw**, **Mermaid**, and **Obsidian Canvas** diagrams from text.

> Forked from [axton-obsidian-visual-skills](https://github.com/axtonliu/axton-obsidian-visual-skills) by [Axton Liu](https://github.com/axtonliu) and localized to English. The original repo has a video walkthrough and demo images.

## Status

Experimental prototype. It works well for common cases, but output quality varies with model version and input structure. Bug reports with a reproducible case (input + output file + steps) are welcome; feature requests may not be acted on.

## Skills

### Excalidraw Diagram Generator

Hand-drawn style diagrams (Excalifont) with bound elements and three output modes:

| Mode | Output | Use |
|------|--------|-----|
| **Obsidian** (default) | `.md` | Open in Obsidian |
| **Standard** | `.excalidraw` | Open/share on excalidraw.com |
| **Animated** | `.excalidraw` | Animate via [excalidraw-animate](https://dai-shi.github.io/excalidraw-animate/) |

Diagram types: flowchart, mind map, hierarchy, relationship, comparison, timeline, matrix, freeform.
Triggers: `Excalidraw`, `diagram`, `flowchart`, `mind map`; `standard excalidraw`; `animate`.

### Mermaid Visualizer

Mermaid diagrams with built-in syntax-error prevention (list conflicts, subgraph naming, special characters) and configurable layouts.

Diagram types: process flow, circular flow, comparison, mindmap, sequence, state.
Triggers: `Mermaid`, `visualize`, `flowchart`, `sequence diagram`.

### Obsidian Canvas Creator

Valid JSON Canvas (`.canvas`) files with smart node sizing, labeled edges, color-coded nodes, and overlap-avoiding spacing.

Layouts: MindMap (radial hierarchy) or Freeform (custom positioning).
Triggers: `Canvas`, `mind map`, `visual diagram`.

## Installation

Requires the [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code). The Excalidraw skill's Obsidian mode also needs the [Excalidraw plugin](https://github.com/zsviczian/obsidian-excalidraw-plugin).

**Plugin marketplace (recommended):**

```
/plugin marketplace add samcolmanetti/visual-skills
/plugin install obsidian-visual-skills
```

Restart Claude Code to load the skills.

**Manual:**

```bash
git clone https://github.com/samcolmanetti/visual-skills.git
cp -r visual-skills/{excalidraw-diagram,mermaid-visualizer,obsidian-canvas-creator} ~/.claude/skills/
```

## Usage

Claude Code uses these skills automatically when you ask for a visualization:

```
"Create an Excalidraw flowchart showing the CI/CD pipeline"
"Visualize this process as a Mermaid sequence diagram"
"Turn this article into an Obsidian Canvas"
```

## File Structure

```
visual-skills/
├── excalidraw-diagram/     # SKILL.md, references/ (JSON schema), assets/ (example)
├── mermaid-visualizer/     # SKILL.md, references/ (syntax rules)
├── obsidian-canvas-creator/ # SKILL.md, references/, assets/ (templates)
├── README.md
└── LICENSE
```

## Acknowledgments

- [Axton Liu](https://github.com/axtonliu): original author of [axton-obsidian-visual-skills](https://github.com/axtonliu/axton-obsidian-visual-skills)
- [Excalidraw](https://excalidraw.com/), [Mermaid](https://mermaid.js.org/), [JSON Canvas](https://jsoncanvas.org/), [Obsidian](https://obsidian.md/)

## License

MIT License, see [LICENSE](LICENSE). Maintained by [Sam Colmanetti](https://github.com/samcolmanetti).
