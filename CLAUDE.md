# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hugo-based blog website for DevOps Toolkit Live (devopstoolkit.live), featuring articles about AI, DevOps, Kubernetes, Infrastructure-as-Code, and Platform Engineering. The site uses the hugo-theme-relearn theme with a custom "dot" variant.

## Development Environment

The project uses Devbox for environment management. Dependencies are managed in `devbox.json`:
- Hugo v0.125.4 for static site generation
- Dart Sass v1.75.0 for CSS compilation  
- Git v2.47.2 for version control

## Common Commands

### Development Server
```bash
devbox shell  # Enter development environment
hugo server   # Start local development server (typically on localhost:1313)
```

### Build
```bash
hugo          # Build static site to public/ directory
```

### Environment Setup
```bash
devbox install  # Install all dependencies defined in devbox.json
```

## Content Architecture

Content is organized in `/content/` directory by topic:
- `/ai/` - Articles about AI tools, coding assistants, and automation
- `/app-management/` - Application deployment and management
- `/ci-cd/` - Continuous Integration/Continuous Deployment 
- `/cloud/` - Cloud platforms and services
- `/containers/` - Docker and containerization
- `/crossplane/` - Crossplane infrastructure management
- `/development/` - Development environments and tools
- `/infrastructure-as-code/` - Terraform, Ansible, infrastructure automation
- `/internal-developer-platforms/` - Platform engineering and developer portals
- `/kubernetes/` - Kubernetes deployment and management
- `/observability/` - Monitoring and logging
- `/security/` - DevSecOps and security practices
- `/terminal/` - CLI tools and terminal productivity

Each article is in its own directory with `_index.md` and associated images.

## Hugo Configuration

- Site config: `hugo.toml` 
- Theme: hugo-theme-relearn with 'dot' variant
- Custom partials: `layouts/partials/` for branding
- Static assets: `static/` for logos and site assets

## Content Creation

New content follows Hugo's archetype pattern (`archetypes/default.md`). Articles should include:
- Frontmatter with title and metadata
- Thumbnail images in article directory
- Properly formatted markdown content

## Theme Customization

The site uses a custom theme variant ('dot') defined in hugo.toml. Custom styling is in `assets/css/theme-dot.css` and theme files are in `themes/hugo-theme-relearn/`.