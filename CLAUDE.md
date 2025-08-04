# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal portfolio/resume website built with Hugo static site generator, using the `hugo-goa` theme. The site showcases David Weterings' professional profile as a freelance senior software consultant.

## Architecture

- **Generator**: Hugo (v0.148.2+extended)
- **Theme**: hugo-goa (included as git submodule)
- **Configuration**: config.toml (Hugo configuration file)
- **Content**: Markdown files in `content/` directory
- **Static Assets**: Images and other assets in `static/img/`
- **Generated Site**: Output goes to `public/` directory

## Key Commands

### Development
```bash
# Start development server with live reload
hugo server

# Start development server including draft content
hugo server -D

# Start development server and bind to all interfaces
hugo server --bind 0.0.0.0
```

### Building
```bash
# Build the site for production
hugo

# Build including draft content
hugo -D

# Clean generated files
rm -rf public/
```

### Content Management
```bash
# Create new content
hugo new content-name.md

# Create new post
hugo new posts/post-name.md
```

### Theme Management
```bash
# Update theme submodule
git submodule update --remote themes/hugo-goa
```

## Project Structure

- `config.toml` - Hugo site configuration, including theme settings, social links, and site metadata
- `content/` - Markdown content files (resume.md, code_of_conduct.md)
- `themes/hugo-goa/` - Git submodule containing the Hugo Goa theme
- `static/img/` - Static images (headshot.jpg)
- `public/` - Generated static site (git-ignored)
- `resources/` - Hugo's processed assets cache

## Configuration Notes

The site is configured for:
- Base URL: https://davidweterings.nl
- Author: David Weterings
- Theme: hugo-goa with custom styling
- Social links: GitHub (davidweterings), LinkedIn (davidweterings), Email (david@davidweterings.nl)

## Content Guidelines

- Use front matter with `title` and `draft: false` for published content
- The theme supports markdown content with highlighting via HighlightJS
- Main navigation is configured in config.toml under `[[menu.main]]`
- Author image should be placed in `static/img/headshot.jpg`

## Deployment

The site builds to the `public/` directory and can be deployed to any static hosting service. The current setup appears to target production deployment at https://davidweterings.nl.