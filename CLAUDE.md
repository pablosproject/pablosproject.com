# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio/resume website for Paolo Tagliani at https://pablosproject.com/. Built with Hugo static site generator using a custom fork of the hugo-researcher theme.

## Development Commands

```bash
# Serve locally with draft content
hugo server -D

# Build for production
hugo -D

# Create a new post
hugo new posts/POST.md
```

**Requirement**: Hugo Extended version (for SCSS/Sass support)

## Architecture

- **Static Site Generator**: Hugo
- **Theme**: Custom fork of hugo-researcher (git submodule at `/themes/researcher/`)
- **Content**: Markdown files in `/content/`
- **Configuration**: TOML (`config.toml`)
- **Styling**: SCSS in `/themes/researcher/assets/sass/`

### Content Structure

| File | Purpose |
|------|---------|
| `content/_index.md` | Homepage ("About Me") |
| `content/works.md` | Portfolio of projects |
| `content/contact.md` | Contact information |
| `content/about.md` | Redirect to homepage |

### Key Configuration (config.toml)

- Base URL: `https://pablosproject.com`
- Font: Inconsolata
- Page width: 750px
- Raw HTML enabled in Markdown (`unsafe: true`)
- Disabled: Taxonomies, RSS, Sitemap, RobotsTXT

## Theme Customization

The theme is a private fork managed as a git submodule. To modify styling:

1. Edit SCSS in `/themes/researcher/assets/sass/researcher.scss`
2. Theme layout templates in `/themes/researcher/layouts/`
3. Style variables configured in `config.toml` under `[params.style]`

## Static Assets

Place images, favicons, and other static files in `/static/`. They will be served from the site root.
