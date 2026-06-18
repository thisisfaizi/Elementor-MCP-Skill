# Elementor MCP Skill for Claude

A Claude Code Skill that enables Claude to rebuild websites, mockups, screenshots, and design briefs as real Elementor pages on WordPress using the Elementor MCP server.

Built for the [Elementor MCP](https://github.com/msrbuilds/elementor-mcp) project by Mian Shahzad Raza, this skill teaches Claude how to create pixel-perfect, SEO-friendly, and fully responsive Elementor pages using native Elementor widgets rather than raw HTML.

## What This Skill Does

This skill transforms Claude into an Elementor specialist capable of:

* Converting HTML mockups into Elementor pages
* Recreating website screenshots in Elementor
* Building landing pages from design briefs
* Editing existing Elementor pages
* Creating Theme Builder templates
* Creating Elementor popups
* Applying responsive layouts
* Configuring Elementor Global Styles
* Building SEO-friendly page structures

The skill enforces Elementor best practices and prevents common mistakes such as using HTML widgets for page construction.

## Core Principles

### Native Elementor Widgets Only

This skill strictly instructs Claude to use:

* Containers / Flexbox Containers
* Heading widgets
* Text Editor widgets
* Buttons
* Images
* Icons
* Accordions
* Tabs
* Testimonials
* Star Ratings
* Videos

Instead of:

* HTML widgets
* Shortcode widgets for layout
* Custom HTML injection
* Custom JavaScript for page structure

This ensures pages remain:

* Editable in Elementor
* Mobile responsive
* SEO friendly
* Future proof
* Easy for clients to maintain

### Pixel-Perfect Design Recreation

The skill teaches Claude to:

* Extract color systems
* Analyze typography scales
* Match spacing systems
* Recreate responsive behavior
* Maintain visual hierarchy
* Preserve layout structure

Rather than approximating designs.

### SEO-First Approach

The skill automatically promotes:

* Proper heading hierarchy
* Single H1 usage
* Image alt text
* Semantic content structure
* Meta title configuration
* Meta description configuration
* Crawlable text content

## Supported Workflows

### HTML to Elementor

Convert existing HTML/CSS pages into native Elementor layouts.

### Screenshot to Elementor

Analyze screenshots and recreate them as editable Elementor pages.

### Design Brief to Elementor

Generate complete Elementor pages from written requirements.

### Existing Page Editing

Modify, restructure, or improve existing Elementor pages.

### Theme Builder Templates

Create:

* Headers
* Footers
* Single Templates
* Archive Templates

When Elementor Pro is available.

### Popup Creation

Build Elementor Popups with proper triggers and targeting rules.

## Elementor Version Awareness

Before building, the skill instructs Claude to detect:

* Elementor version
* Available widget families
* Elementor Pro status
* Available tools

The user is then asked to choose between:

* Atomic Widgets (Elementor 4.x)
* Classic Widgets (Elementor 3.x style)

The skill never mixes widget systems in the same build.

## Design System Extraction

Before creating any page, Claude is instructed to identify:

* Color palette
* Typography system
* Spacing scale
* Border radius scale
* Shadow system
* Responsive breakpoints
* Section structure

These values are then applied to Elementor Global Styles whenever possible.

## Responsive Design Standards

The skill enforces:

* Desktop layouts
* Tablet layouts
* Mobile layouts
* Device-specific typography
* Device-specific spacing
* Responsive container behavior
* Proper grid stacking

To ensure high-quality responsive output.

## Tool Efficiency

To reduce MCP tool usage, the skill encourages:

* Batch operations
* Reusable templates
* Structured page builds
* Minimal redundant tool calls

This is especially useful on servers with tool limits.

## Installation

Clone or copy this skill into your Claude Skills directory:

```bash
~/.claude/skills/elementor-mcp/
```

Place the skill instructions inside:

```bash
skill.md
```

Then restart Claude Code or reload skills.

## Requirements

* Claude Code
* Elementor MCP Server
* WordPress Website
* Elementor Plugin
* Elementor Pro (optional)

## Recommended Use Cases

* Agency website builds
* Landing page creation
* Website migrations
* HTML-to-Elementor conversions
* Figma-to-Elementor workflows
* Design recreation projects
* White-label WordPress development

## Credits

### Elementor MCP

Created by Mian Shahzad Raza

GitHub Repository:

https://github.com/msrbuilds/elementor-mcp

### Skill Author

Created as a Claude Code Skill to provide structured guidance for using Elementor MCP effectively and producing professional Elementor websites using native Elementor architecture.
