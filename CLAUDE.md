# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains Lightning Talk (LT) presentation materials in Japanese about Jetpack Compose and mobile app architecture topics. The content is technical in nature, targeting Android developers.

## Content Structure

The repository is organized by presentation topic:

- **jetpack_compose_side_effect/**: Presentation on side effects in Jetpack Compose
  - `talk.md`: Full talk script
  - `slide.md`: Marp-formatted slides
  - `slide.pdf`: Compiled PDF slides

- **intro_state_driven_design/**: Presentation on testable app architecture with state-driven design
  - `index.md`: Overview/outline
  - `talk_[1-6].md`: Talk script split into sections
  - `slide.md`: Marp-formatted slides

## Slide Generation

Slides use [Marp](https://marp.app/) for markdown-to-presentation conversion:

```bash
# Generate PDF slides from markdown (requires Marp CLI)
marp slide.md -o slide.pdf
```

The slides use the following Marp configuration:
- Theme: default
- Paginate: true
- Background: white (#fff)
- Custom styling for code blocks and headers

## Content Guidelines

When working with presentation content in this repository:

1. **Language**: All content is in Japanese. Maintain the language consistency.

2. **Technical Focus**: Content covers:
   - Jetpack Compose side effects and Effect APIs
   - State management and testability
   - Declarative UI patterns
   - Architecture patterns (MVI, state-driven design)
   - Domain-driven design concepts

3. **Slide Structure**: Marp slides follow a consistent pattern:
   - YAML frontmatter for configuration
   - `---` separates slides
   - Code examples use Kotlin syntax highlighting
   - Tables for API comparisons

4. **Talk Scripts**: The `talk.md` files contain:
   - Full narrative text
   - Section markers (`---`)
   - Code examples with explanations
   - Structured progression from concepts to implementation

## Common Tasks

### Adding New Presentation
Create a new directory with:
- `slide.md`: Marp-formatted slides
- `talk.md`: Full talk script
- `index.md`: (optional) Overview outline

### Editing Existing Content
- Talk scripts provide detailed context for slides
- Keep slide content concise while talk scripts can be detailed
- Maintain consistency between talk script and slides

### Reviewing Content
- Check that code examples are syntactically valid Kotlin
- Verify technical accuracy of Compose APIs mentioned
- Ensure Japanese text is natural and technical terms are used correctly
